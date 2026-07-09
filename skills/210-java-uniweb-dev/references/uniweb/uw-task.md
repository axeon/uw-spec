# uw-task — 分布式任务框架

**Maven 坐标**: `com.umtone:uw-task`

支持定时任务和队列任务的分布式任务框架，依赖 RabbitMQ + Redis。配合服务端 uw-task-center 实现动态配置、状态上报与告警分发。

**配置前缀**: `uw.task`

```yaml
uw:
  task:
    # 是否启用任务注册（任务执行主机 true；仅作调用方的服务 false）
    enable-registry: true
    # 任务项目，必须是包名前缀，只扫描该包下的 TaskCroner/TaskRunner
    task-project: com.demo.task
    # 运行目标，识别任务执行集群，默认 default
    run-target: default
    # croner 调度线程数（建议 = croner 任务数 * 70%）
    croner-thread-num: 3
    rabbitmq:
      host: 127.0.0.1
      port: 5672
      username: guest
      password: guest
      publisher-confirms: true
      virtual-host: /
    redis:
      database: 0
      host: 127.0.0.1
      port: 6379
      password: password
      lettuce:
        pool:
          max-active: 20
          max-idle: 8
          max-wait: -1ms
          min-idle: 0
      timeout: 30s
```

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|--------|
| 定时任务 | 继承 `TaskCroner`，加 `@Component` | runTask 需幂等：配置覆盖/降级切换用 cancel(false) 让上次跑完，可能与本次重叠 |
| 队列任务 | 继承 `TaskRunner<P,R>`，加 `@Component` | 同上 |
| 发送到队列 | `taskFactory.sendToQueue(taskData)` | 完全异步，无返回值 |
| 本地优先执行（高频短任务） | `taskFactory.runQueue(taskData)` | 本机有 runner 则本地跑，否则降级入队，不走同步 RPC |
| 同步执行任务 | `taskFactory.runTask(taskData)` | 阻塞等待结果；taskData 不可复用 |
| 构建任务数据 | `TaskData.builder(TaskClass.class, param).refId(id).build()` | Builder 模式 |
| 延迟任务（到期执行） | 继承 `TaskDelayer<P,R>`，加 `@Component` | 第三种任务类型，基于 Redis zset poll，无队头阻塞 |
| 投递延迟任务 | `taskFactory.delayTask(taskData)` | 异步无返回值，`taskDelay` 设延迟毫秒 |
| 队列任务延时 | `TaskData.builder(...).taskDelay(5000)` + `delayType=ON` | 走 MQ TTL+死信，长延时阻塞短延时，建议改用 TaskDelayer |
| 触发重试 | 抛 `TaskPartnerException` | 第三方接口错误，按 retryTimesByPartner 重试 |
| 超限流重试 | （框架自动，配置 retryTimesByOverrated） | STATE_FAIL_CONFIG 时重试 |
| 不重试 | 抛 `TaskDataException` | 数据错误不重试 |
| center 不可用降级 | 三套任务自动用 initConfig 本地默认配置继续跑 | 拉取失败+首次触发（非失败计数），center 恢复后配置覆盖；详见客户端 README |

> **TaskFactory 获取**：Helper/Service 中 `@Autowired private TaskFactory taskFactory;` 注入；或 `TaskFactory.getInstance()` 静态获取。

## TaskCroner 定时任务

> **包路径**：`uw.task.TaskCroner`

构造：继承 `TaskCroner` 抽象类 + `@Component` — 实现 `runTask`/`initConfig`/`initContact` 三个抽象方法。

**TaskCronerConfig 构造**：`new TaskCronerConfig()` setter 链式配置

| 属性 | 类型 | 说明 |
|------|------|------|
| taskName | String | 任务名称（必填） |
| taskCron | String | cron 表达式，默认 `*/5 * * * * ?` |
| runType | int | 运行类型（见下表） |
| taskParam | String | 多实例区分参数（同一 taskClass 不同 taskParam = 不同实例） |
| runTarget | String | 运行目标，默认 "default" |
| logLevel | int | 日志级别（见下表） |
| logLimitSize | int | 日志大小限制，0=无限制 |
| alertRunTimeout | int | 运行超时告警（毫秒） |
| alertFailRate | int | 失败率告警阈值（百分比） |

**TaskCronerConfig.runType 常量**：

| 常量 | 值 | 说明 |
|------|-----|------|
| RUN_TYPE_ANYWAY | 0 | 到处运行（所有匹配主机都执行） |
| RUN_TYPE_SINGLETON | 1 | 全局单例执行（仅 Leader 主机执行） |

**TaskCronerConfig.logLevel 常量（TASK_LOG_TYPE）**：

| 常量 | 值 | 说明 |
|------|-----|------|
| TASK_LOG_TYPE_NONE | -1 | 不记录日志 |
| TASK_LOG_TYPE_RECORD | 0 | 基础信息 |
| TASK_LOG_TYPE_RECORD_TASK_PARAM | 1 | 含参数 |
| TASK_LOG_TYPE_RECORD_RESULT_DATA | 2 | 含返回 |
| TASK_LOG_TYPE_RECORD_ALL | 3 | 全部（参数+返回） |

**TaskCronerLog 字段**：id / taskId / taskClass / taskParam / runType / runTarget / taskCron / scheduleDate / runDate / finishDate / nextDate / resultData / state

**示例**：
```java
// TaskCroner 是 Spring Bean（由框架管理生命周期），必须加 @Component。
// 这与 Helper（纯静态工具类，禁止 @Component）不同。
@Component
public class OrderTimeoutCheckTask extends TaskCroner {

    @Override
    public String runTask(TaskCronerLog taskCronerLog) throws Exception {
        String taskParam = taskCronerLog.getTaskParam();  // 多实例参数（可空）
        int timeoutCount = orderService.checkTimeoutOrders();
        return "检查完成，处理超时订单: " + timeoutCount + " 条";
    }

    @Override
    public TaskCronerConfig initConfig() {
        TaskCronerConfig config = new TaskCronerConfig();
        config.setTaskName("订单超时检查");
        config.setTaskDesc("每5分钟检查一次超时未支付订单");
        config.setTaskCron("0 */5 * * * ?");
        config.setRunType(TaskCronerConfig.RUN_TYPE_SINGLETON);  // 全局单例
        config.setLogLevel(TaskCronerConfig.TASK_LOG_TYPE_RECORD_ALL);
        config.setAlertRunTimeout(300);
        return config;
    }

    @Override
    public TaskContact initContact() {
        return TaskContact.builder("运维负责人")
            .email("ops@example.com")
            .mobile("13800138000")
            .build();
    }
}
```

## TaskRunner 队列任务

> **包路径**：`uw.task.TaskRunner`

构造：继承 `TaskRunner<TP, RD>` 抽象类 + `@Component` — 泛型 TP=参数类型，RD=返回类型。**TaskRunner 是单例，多消费者线程并发调用同一实例的 runTask，勿使用非线程安全的实例变量**。

**TaskRunnerConfig 构造**：`new TaskRunnerConfig()` setter 链式配置

| 属性 | 类型 | 说明 |
|------|------|------|
| taskName | String | 任务名称（必填） |
| queueType | int | 队列类型（见下表） |
| delayType | int | 延迟类型：TYPE_DELAY_OFF=0 / TYPE_DELAY_ON=1 |
| consumerNum | int | 消费者并发数 |
| prefetchNum | int | 预取数量 |
| rateLimitType | int | 限流类型（见下表） |
| rateLimitValue | int | 限流值（窗口内次数） |
| rateLimitTime | int | 限流时间窗口（秒） |
| rateLimitWait | int | 触发限速时最长等待秒数 |
| retryTimesByPartner | int | 合作方异常重试次数（N → 总计执行 N+1 次） |
| retryTimesByOverrated | int | 超限流异常重试次数（N → 总计执行 N+1 次） |
| runType | int | 运行模式（见 TaskData 运行模式常量） |
| logLevel | int | 日志级别（见 TASK_LOG_TYPE 常量） |
| logLimitSize | int | 日志大小限制 |
| alertRunTimeout | int | 超时告警（毫秒） |
| alertFailRate | int | 失败率告警阈值 |

> 注意：**没有 `retryTimesByProgram`**。程序异常（STATE_FAIL_PROGRAM）不重试。

**TaskRunnerConfig.queueType 常量（TYPE_QUEUE）**：

| 常量 | 值 | 说明 |
|------|-----|------|
| TYPE_QUEUE_PROJECT | 0 | 项目级队列（同项目共用） |
| TYPE_QUEUE_PROJECT_PRIORITY | 1 | 项目级优先队列 |
| TYPE_QUEUE_GROUP | 2 | 任务组队列（按 taskClass 所在包） |
| TYPE_QUEUE_GROUP_PRIORITY | 3 | 任务组优先队列 |
| TYPE_QUEUE_TASK | 5 | 任务级队列（按 taskClass+taskTag 独立） |

**TaskRunnerConfig.rateLimitType 常量（RATE_LIMIT）**：

本地限速（进程内 Guava 令牌桶）/ 全局限速（Redis 固定窗口）：

| 常量 | 值 | 限流 key 维度 | 说明 |
|------|-----|--------------|------|
| RATE_LIMIT_NONE | 0 | — | 不限速 |
| RATE_LIMIT_LOCAL | 1 | 进程（共享） | 本地所有任务共用一个限速器（慎用，易卡死） |
| RATE_LIMIT_LOCAL_TASK | 2 | 进程 + taskClass | 本地按任务隔离 |
| RATE_LIMIT_LOCAL_TASK_TAG | 3 | 进程 + taskClass + tag | 本地按任务+TAG 隔离 |
| RATE_LIMIT_GLOBAL_HOST | 4 | host | **跨任务共享**：按主机 |
| RATE_LIMIT_GLOBAL_TAG | 5 | tag | **跨任务共享**：按 TAG（多任务对接同一接口共享配额） |
| RATE_LIMIT_GLOBAL_TASK | 6 | taskClass | 按任务隔离 |
| RATE_LIMIT_GLOBAL_TAG_HOST | 7 | tag + host | **跨任务共享** |
| RATE_LIMIT_GLOBAL_TASK_HOST | 8 | taskClass + host | 按任务+主机隔离 |
| RATE_LIMIT_GLOBAL_TASK_TAG | 9 | taskClass + tag | 按任务+TAG 隔离 |
| RATE_LIMIT_GLOBAL_TASK_TAG_HOST | 10 | taskClass + tag + host | 全维度隔离 |

> 选择建议：单实例用 LOCAL*（开销低）；多实例且对接同一第三方接口需共享 QPS 用 `GLOBAL_TAG/HOST/TAG_HOST`；每任务独立配额用 `GLOBAL_TASK*`。全局限速建议 rateLimitTime ≥ 5 秒。

**TaskData 构造**：`TaskData.builder(TaskClass.class, param).refId(id).taskDelay(5000).build()` — Builder 模式

也支持 `new TaskData<TP, RD>()` + setter

**TaskRunner 实现示例**：
```java
// TaskRunner 是 Spring Bean（由框架管理生命周期），必须加 @Component。
@Component
public class OrderNotifyTask extends TaskRunner<OrderNotifyParam, NotifyResult> {

    @Autowired
    private NotificationService notificationService;

    @Override
    public NotifyResult runTask(TaskData<OrderNotifyParam, NotifyResult> taskData) throws Exception {
        OrderNotifyParam param = taskData.getTaskParam();
        boolean success = notificationService.sendNotify(param.getUserId(), param.getNotifyType());
        NotifyResult result = new NotifyResult();
        result.setSuccess(success);
        result.setMessage(success ? "发送成功" : "发送失败");
        return result;
    }

    @Override
    public TaskRunnerConfig initConfig() {
        TaskRunnerConfig config = new TaskRunnerConfig();
        config.setTaskName("订单通知任务");
        config.setTaskDesc("发送订单状态变更通知给用户");
        config.setQueueType(TaskRunnerConfig.TYPE_QUEUE_PROJECT);
        config.setConsumerNum(5);
        config.setPrefetchNum(1);
        config.setRateLimitType(TaskRunnerConfig.RATE_LIMIT_GLOBAL_TASK);  // 全局按任务隔离
        config.setRateLimitValue(100);
        config.setRateLimitTime(60);
        config.setRetryTimesByPartner(3);  // 第三方错误：总计执行 4 次（1+3）
        return config;
    }

    @Override
    public TaskContact initContact() {
        return TaskContact.builder("通知服务负责人").email("notify@example.com").build();
    }
}
```

**异常处理示例**：
```java
@Override
public NotifyResult runTask(TaskData<OrderNotifyParam, NotifyResult> taskData) throws Exception {
    OrderNotifyParam param = taskData.getTaskParam();
    try {
        Response response = thirdPartyApi.send(param);
        if (response.getCode() == 429) {
            throw new TaskPartnerException("第三方接口限流");  // 会重试（retryTimesByPartner）
        }
        if (response.getCode() == 400) {
            throw new TaskDataException("参数错误: " + response.getMessage());  // 不重试
        }
        return new NotifyResult(true, "成功");
    } catch (IOException e) {
        throw new TaskPartnerException("网络异常", e);  // 会重试
    }
}
```

**TaskData 关键字段**：

| 字段 | 类型 | 说明 |
|------|------|------|
| taskClass | String | 要执行的 TaskRunner 全限定名（必填，builder 自动填） |
| taskParam | TP | 任务参数 |
| resultData | RD | 结果数据 |
| state | int | 任务状态 |
| runType | int | 运行模式 |
| runTarget | String | 运行目标，默认 default |
| taskTag | String | 任务标签，多实例区分 |
| taskDelay | long | 延迟时间（毫秒） |
| refId / refSubId / refTag | long/String | 关联信息（第三方统计用） |
| ranTimes | int | 已执行次数 |
| errorInfo | String | 错误信息 |

**TaskData 状态常量**：STATE_UNKNOWN=0 / STATE_SUCCESS=1 / STATE_FAIL_PROGRAM=2 / STATE_FAIL_CONFIG=3 / STATE_FAIL_PARTNER=4 / STATE_FAIL_DATA=5

**TaskData 运行模式**：

| 常量 | 值 | 说明 |
|------|-----|------|
| RUN_TYPE_LOCAL | 1 | 本地执行，不受流控 |
| RUN_TYPE_GLOBAL | 3 | 全局执行（入队），受流控 |
| RUN_TYPE_GLOBAL_RPC | 5 | 全局同步 RPC，不受流控 |
| RUN_TYPE_AUTO_RPC | 6 | 自动选择本地或远程（默认） |

## TaskDelayer 延迟任务

> **包路径**：`uw.task.TaskDelayer`

构造：继承 `TaskDelayer<TP, RD>` 抽象类 + `@Component` — 泛型 TP=参数类型，RD=返回类型。三方法与 TaskRunner 对齐（`initConfig`/`initContact`/`run`），区别是 `run(TaskData)` 返回 **void**（结果经 taskData 回写）、载体是 **Redis zset 而非 RabbitMQ**。

**机制**：投递时写入 Redis zset（key=`uw-task-delayer:`+configKey，score=到期时间戳=`queueDate+taskDelay`，member=Kryo 序列化 TaskData）；各 uw-task 实例独立 poll 同一 zset，Lua 原子 `ZRANGEBYSCORE+ZREM` 取出到期任务、虚拟线程即取即执行（无内存队列、并发许可耗尽则任务留 zset）。多实例竞争消费、**无队头阻塞、重启不丢存量**。

**TaskDelayerConfig 构造**：`new TaskDelayerConfig()` setter 链式，或 `TaskDelayerConfig.builder()`

| 属性 | 类型 | 默认 | 说明 |
|------|------|------|------|
| taskName | String | — | 任务名称（必填） |
| consumerNum | int | 1 | 执行并发数（虚拟线程 + Semaphore） |
| pollInterval | long | 3 | zset poll 间隔（秒），命中连续 poll、空才 sleep |
| prefetchNum | int | 50 | 单次 poll 最大条数（实际取 `min(此值, 空闲并发许可)`） |
| rateLimitType | int | 0 | 限流类型，复用 `TaskRunnerConfig.RATE_LIMIT_*` 常量 |
| rateLimitValue / rateLimitTime / rateLimitWait | int | 10 / 1 / 30 | 限流值 / 窗口秒 / 触发限速等待秒 |
| retryTimesByPartner | int | 0 | 合作方异常重试次数（N → 总计执行 N+1 次） |
| retryTimesByOverrated | int | 3 | 连续限速超限放弃次数上限（防死循环），超限标 STATE_FAIL_CONFIG |
| retryTimesByProgram | int | 0 | 程序异常重试次数（默认 0 不重试） |
| alertFailRate / alertFailProgramRate / alertFailPartnerRate | int | — | 失败率报警阈值（%） |
| alertRunTimeout | int | — | 平均运行耗时报警（毫秒） |
| alertDelayOvertime | int | — | **Delayer 特有**：实际执行晚于 runAt 的平均超时报警（毫秒） |
| logLevel | int | 0 | 日志类型，复用 `TaskRunnerConfig.TASK_LOG_TYPE_*` |

**投递**：`taskFactory.delayTask(taskData)` — 完全异步、无返回值。taskData 需设 `taskClass`（TaskDelayer 实现类）+ `taskDelay`（延迟毫秒）+ `taskParam`。

**示例**：
```java
@Component
public class OrderTimeoutTask extends TaskDelayer<OrderTimeoutParam, Void> {

    @Override
    public void run(TaskData<OrderTimeoutParam, Void> task) throws Exception {
        orderService.closeIfUnpaid(task.getTaskParam().getOrderId());
    }

    @Override
    public TaskDelayerConfig initConfig() {
        TaskDelayerConfig config = new TaskDelayerConfig();
        config.setTaskName("订单超时关闭");
        config.setConsumerNum(5);
        config.setPollInterval(3);
        config.setRetryTimesByPartner(3);       // 第三方失败：总计执行 4 次（1+3）
        config.setAlertDelayOvertime(5000);     // 延迟超时报警（毫秒）
        return config;
    }

    @Override
    public TaskContact initContact() {
        return TaskContact.builder("订单负责人").email("order@example.com").build();
    }
}

// 投递：30 分钟后执行
TaskData<OrderTimeoutParam, Void> data = TaskData
    .<OrderTimeoutParam, Void>builder(OrderTimeoutTask.class, new OrderTimeoutParam(orderId))
    .taskDelay(30 * 60 * 1000L)
    .refId(orderId)
    .build();
taskFactory.delayTask(data);
```

> 与 TaskRunner 的选择：需"X 时间后执行"用 **TaskDelayer**（Redis zset，无队头阻塞）；需异步队列消费用 **TaskRunner**（RabbitMQ）。TaskRunner 的 `taskDelay` + `delayType=TYPE_DELAY_ON` 走 MQ 死信延时，有长延时阻塞短延时问题，优先用 TaskDelayer。

## TaskFactory

> **包路径**：`uw.task.TaskFactory`

构造：`TaskFactory.getInstance()` 静态获取，或 `@Autowired` 注入。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `sendToQueue(TaskData)` | void | 发送到 MQ 队列（异步） |
| `delayTask(TaskData)` | void | 投递延迟任务到 Redis zset，到期本机 poll 执行（异步，`taskDelay` 设延迟毫秒） |
| `runQueue(TaskData)` | void | 本地线程池优先执行，满则降级入队；带 taskDelay 直接入队 |
| `runTask(TaskData)` | `TaskData<TP, RD>` | 同步执行（阻塞，按 runType 选本地/远程 RPC） |
| `runTaskLocal(TaskData)` | `TaskData<TP, RD>` | 强制本地同步执行（无 runner 抛异常） |
| `runTaskAsync(TaskData)` | `Future<TaskData<TP, RD>>` | 异步执行 |
| `getQueueInfo(queueName)` | int[] | 获取队列信息 [消息数, 消费者数] |
| `purgeQueue(queueName)` | int | 清空队列（返回清除条数） |

**run 系列方法对比与快速选型**（三个维度：执行位置 × 是否阻塞调用方 × 是否需要返回值）：

| 方法 | 执行位置 | 阻塞调用方 | 返回结果 | 本机无 runner 时 | 典型用途 |
|---|---|---|---|---|---|
| `sendToQueue` | 远程（入队） | 否 | 无 | 入队（正常） | 标准异步队列任务，不关心结果 |
| `runTask` | AUTO 自动选 | **是** | 有 | 回退远程 RPC | 需立即同步拿结果 |
| `runTaskLocal` | 仅本地 | **是** | 有 | **抛异常** | 强制本机执行，不允许远程 |
| `runTaskAsync` | AUTO 自动选 | 否（Future） | 有（get 时） | 回退远程 RPC | 异步拿结果，注意线程池 |
| `runQueue` | 本地优先，否则入队 | 否 | 无 | 入队 | 高频短任务优化，省 MQ |

**选型决策树**：

```
需要执行结果吗？
├─ 否 → 本机有 runner 且高频短耗时？
│        ├─ 是 → runQueue（本地优先，省 MQ）
│        └─ 否 → sendToQueue（标准入队）
│
└─ 是 → 调用方能接受阻塞吗？
         ├─ 能（且要立即拿结果）
         │    ├─ 必须本机跑  → runTaskLocal
         │    └─ 本地/远程都行 → runTask
         │
         └─ 不能（异步） → runTaskAsync（注意线程池上限）
```

> **重要**：`runTask`/`runTaskLocal`/`runTaskAsync`/`runQueue` 会向传入的 taskData 写入 id/queueDate/runType 等运行期字段，**禁止复用同一 taskData 对象**。远程模式（`runTask`/`runTaskAsync`）默认 sendAndReceive 超时 180 秒，避免在 Web 请求线程高频同步调用以免耗尽 Tomcat 线程。

## 重试与异常处理

| 异常类 | 状态 | 触发重试 | 使用场景 |
|--------|------|---------|---------|
| `TaskPartnerException(msg)` | STATE_FAIL_PARTNER | ✅ 按 retryTimesByPartner | 第三方接口限流、网络异常 |
| （框架自动） | STATE_FAIL_CONFIG | ✅ 按 retryTimesByOverrated | 超过流量限制 |
| `TaskDataException(msg)` | STATE_FAIL_DATA | ❌ 不重试 | 参数错误、数据格式错误 |
| 其他未捕获异常 | STATE_FAIL_PROGRAM | ❌ 不重试 | 程序 bug |

**重试语义**：`retryTimes = N` 时，总计执行 **N+1 次**（1 次初始 + N 次重试）。重试延时按执行轮次线性递增。

## 多实例配置

同一套任务需要多个并发运行实例时，通过服务端多份配置实现：

- **TaskCroner**：用 `taskParam` 区分多实例（同一 taskClass 不同 taskParam = 不同实例）。
- **TaskRunner**：用 `taskTag` 区分多实例，发送任务时指定 taskTag/runTarget。

多实例唯一性由服务端按三元组（taskClass + 区分维度 + runTarget）保证。无法精确匹配时框架宽松匹配最合适的配置。

## TaskContact

> **包路径**：`uw.task.TaskContact`

构造：`TaskContact.builder(contactName).email("...").mobile("...").build()` — Builder 模式；或 7 参构造器 `new TaskContact(contactName, mobile, email, wechat, im, notifyUrl, remark)`

| 字段 | 类型 | 说明 |
|------|------|------|
| contactName | String | 联系人姓名 |
| mobile | String | 联系电话 |
| email | String | 联系邮箱 |
| wechat | String | 微信 |
| im | String | IM |
| notifyUrl | String | 通知链接（钉钉/微信 webhook） |
| remark | String | 备注 |

## Helper 调用示例

```java
public class OrderHelper {
    @Autowired
    private TaskFactory taskFactory;  // 注入（也可 TaskFactory.getInstance()）

    public void sendOrderNotify(long orderId, long userId) {
        OrderNotifyParam param = new OrderNotifyParam();
        param.setOrderId(orderId);
        param.setUserId(userId);
        param.setNotifyType("ORDER_CREATED");

        TaskData<OrderNotifyParam, NotifyResult> taskData = TaskData
            .builder(OrderNotifyTask.class, param)
            .refId(orderId)
            .refTag("ORDER_NOTIFY")
            .taskDelay(5000)          // 延迟5秒（需 delayType=ON）
            .build();

        taskFactory.sendToQueue(taskData);  // 异步入队
    }

    public NotifyResult syncNotify(long orderId, long userId) {
        OrderNotifyParam param = new OrderNotifyParam();
        param.setOrderId(orderId);
        param.setUserId(userId);
        param.setNotifyType("ORDER_URGENT");

        TaskData<OrderNotifyParam, NotifyResult> taskData = TaskData
            .builder(OrderNotifyTask.class, param)
            .runType(TaskData.RUN_TYPE_AUTO_RPC)  // 自动选择本地或远程
            .build();

        TaskData<OrderNotifyParam, NotifyResult> result = taskFactory.runTask(taskData);
        if (result.getState() == TaskData.STATE_SUCCESS) {
            return result.getResultData();
        }
        throw new RuntimeException("通知失败: " + result.getErrorInfo());
    }
}
```
