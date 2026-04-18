### uw-task — 分布式任务框架

**Maven 坐标**: `com.umtone:uw-task`

支持定时任务和队列任务的分布式任务框架，依赖 RabbitMQ + Redis。

**配置前缀**: `uw.task`

```yaml
uw:
  task:
    task-project: com.demo.task
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

| 任务类型 | 基类 | 说明 |
|---|---|---|
| 定时任务 | `TaskCroner` | Cron 表达式定时执行，支持服务端动态配置 |
| 队列任务 | `TaskRunner<P, R>` | 流量控制、错误重试、动态配置 |
| RPC 调用 | - | 同步/异步远程调用，自动本地/远程判定 |

#### TaskCroner 定时任务

**TaskCroner API**：

```java
public abstract class TaskCroner {
    // 运行任务，返回执行结果字符串
    public abstract String runTask(TaskCronerLog taskCronerLog) throws Exception;
    
    // 初始化配置信息
    public abstract TaskCronerConfig initConfig();
    
    // 初始化联系人信息
    public abstract TaskContact initContact();
}

// TaskCronerConfig 配置类
public class TaskCronerConfig implements Serializable {
    // 运行模式常量
    public static final int RUN_TYPE_ANYWAY = 0;           // 直接运行模式
    public static final int RUN_TYPE_SINGLETON = 1;        // 全局单例模式
    
    // 日志类型常量
    public static final int TASK_LOG_TYPE_NONE = -1;       // 不记录日志
    public static final int TASK_LOG_TYPE_RECORD = 0;      // 记录基本日志
    public static final int TASK_LOG_TYPE_RECORD_TASK_PARAM = 1;   // 记录含请求参数
    public static final int TASK_LOG_TYPE_RECORD_RESULT_DATA = 2;  // 记录含返回参数
    public static final int TASK_LOG_TYPE_RECORD_ALL = 3;          // 记录全部日志
    
    private String taskName;       // 任务名称（必填）
    private String taskCron;       // cron表达式，默认"*/5 * * * * ?"
    private int runType;           // 运行类型：0随意执行，1全局唯一执行
    private String runTarget;      // 运行目标，默认"default"
    private int logLevel;          // 日志级别
    private int logLimitSize;      // 日志字符串字段大小限制，0表示无限制
    private int alertFailRate;     // 失败率告警阈值
    private int alertFailPartnerRate;  // 接口失败率告警阈值
    private int alertFailProgramRate;  // 程序失败率告警阈值
    private int alertFailDataRate;     // 数据失败率告警阈值
    private int alertWaitTimeout;      // 等待超时时长告警
    private int alertRunTimeout;       // 运行超时时长告警
    
    // Builder模式
    public static Builder builder(String taskName);
    public static Builder builder(String taskName, String taskCron);
}

// TaskCronerLog 日志对象
public class TaskCronerLog extends LogBaseVo {
    private long id;               // 日志ID
    private long taskId;           // 任务配置ID
    private String taskClass;      // 执行类名
    private String taskParam;      // 执行参数
    private int runType;           // 运行类型
    private String runTarget;      // 运行目标
    private String taskCron;       // cron表达式
    private Date scheduleDate;     // 计划执行时间
    private Date runDate;          // 开始运行时间
    private Date finishDate;       // 运行结束时间
    private Date nextDate;         // 下次执行时间
    private String resultData;     // 执行结果
    private int state;             // 执行状态
}
```

**定时任务示例**：
```java
@Component
public class OrderTimeoutCheckTask extends TaskCroner {
    
    @Autowired
    private OrderService orderService;
    
    @Override
    public String runTask(TaskCronerLog taskCronerLog) throws Exception {
        // 获取任务参数（如果有）
        String taskParam = taskCronerLog.getTaskParam();
        
        // 执行业务逻辑
        int timeoutCount = orderService.checkTimeoutOrders();
        
        // 返回执行结果
        return "检查完成，处理超时订单: " + timeoutCount + " 条";
    }
    
    @Override
    public TaskCronerConfig initConfig() {
        TaskCronerConfig config = new TaskCronerConfig();
        config.setTaskName("订单超时检查");
        config.setTaskDesc("每5分钟检查一次超时未支付订单");
        config.setTaskCron("0 */5 * * * ?");  // 每5分钟执行一次
        config.setRunType(TaskCronerConfig.RUN_TYPE_SINGLETON);  // 全局单例执行
        config.setLogLevel(TaskCronerConfig.TASK_LOG_TYPE_RECORD_ALL);
        config.setAlertRunTimeout(300);  // 运行超过300秒告警
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

#### TaskRunner 队列任务

**TaskRunner API**：

```java
public abstract class TaskRunner<TP, RD> {
    // 执行任务，TP为参数类型，RD为返回类型
    public abstract RD runTask(TaskData<TP, RD> taskData) throws Exception;
    
    // 初始化配置信息
    public abstract TaskRunnerConfig initConfig();
    
    // 初始化联系人信息
    public abstract TaskContact initContact();
}

// TaskData 任务数据类
public class TaskData<TP, RD> implements Serializable {
    // 任务状态常量
    public static final int STATE_UNKNOWN = 0;       // 未设置
    public static final int STATE_SUCCESS = 1;       // 成功
    public static final int STATE_FAIL_PROGRAM = 2;  // 程序错误
    public static final int STATE_FAIL_CONFIG = 3;   // 配置错误
    public static final int STATE_FAIL_PARTNER = 4;  // 第三方接口错误
    public static final int STATE_FAIL_DATA = 5;     // 数据错误
    
    // 运行模式常量
    public static final int RUN_TYPE_LOCAL = 1;      // 本地运行
    public static final int RUN_TYPE_GLOBAL = 3;     // 全局运行
    public static final int RUN_TYPE_GLOBAL_RPC = 5; // 全局RPC返回结果
    public static final int RUN_TYPE_AUTO_RPC = 6;   // 自动选择本地或远程
    
    private long id;                // 任务ID（自动生成）
    private String taskClass;       // 执行类名（必填）
    private String taskTag;         // 任务标签
    private TP taskParam;           // 任务参数
    private RD resultData;          // 结果数据
    private int state;              // 任务状态
    private int runType;            // 运行模式
    private long taskDelay;         // 延迟时间（毫秒）
    private String runTarget;       // 运行目标
    private long refId;             // 关联ID
    private long refSubId;          // 关联子ID
    private String refTag;          // 关联TAG
    private String rateLimitTag;    // 流量限制TAG
    private Date queueDate;         // 进入队列时间
    private Date consumeDate;       // 开始消费时间
    private Date runDate;           // 开始运行时间
    private Date finishDate;        // 运行结束时间
    private int ranTimes;           // 已执行次数
    private String errorInfo;       // 错误信息
    
    // Builder模式
    public static <TP, RD> Builder<TP, RD> builder();
    public static <TP, RD> Builder<TP, RD> builder(String taskClass);
    public static <TP, RD> Builder<TP, RD> builder(Class<? extends TaskRunner<TP, RD>> taskClass);
    public static <TP, RD> Builder<TP, RD> builder(String taskClass, TP taskParam);
    public static <TP, RD> Builder<TP, RD> builder(Class<? extends TaskRunner<TP, RD>> taskClass, TP taskParam);
    
    // 复制当前实例
    public TaskData<TP, RD> copy();
}

// TaskRunnerConfig 配置类
public class TaskRunnerConfig implements Serializable {
    // 队列类型常量
    public static final int TYPE_QUEUE_PROJECT = 0;          // 项目队列
    public static final int TYPE_QUEUE_PROJECT_PRIORITY = 1; // 项目优先级队列
    public static final int TYPE_QUEUE_GROUP = 2;            // 任务组队列
    public static final int TYPE_QUEUE_GROUP_PRIORITY = 3;   // 任务组优先级队列
    public static final int TYPE_QUEUE_TASK = 5;             // 任务队列
    
    // 延迟类型常量
    public static final int TYPE_DELAY_ON = 1;   // 延迟任务
    public static final int TYPE_DELAY_OFF = 0;  // 非延迟任务
    
    // 限速类型常量
    public static final int RATE_LIMIT_NONE = 0;                  // 不限速
    public static final int RATE_LIMIT_LOCAL = 1;                 // 本地进程限速
    public static final int RATE_LIMIT_LOCAL_TASK = 2;            // 本地TASK限速
    public static final int RATE_LIMIT_LOCAL_TASK_TAG = 3;        // 本地TASK+TAG限速
    public static final int RATE_LIMIT_GLOBAL_HOST = 4;           // 全局主机HOST限速
    public static final int RATE_LIMIT_GLOBAL_TAG = 5;            // 全局TAG限速
    public static final int RATE_LIMIT_GLOBAL_TASK = 6;           // 全局TASK限速
    public static final int RATE_LIMIT_GLOBAL_TAG_HOST = 7;       // 全局TAG+HOST限速
    public static final int RATE_LIMIT_GLOBAL_TASK_HOST = 8;      // 全局TASK+IP限速
    public static final int RATE_LIMIT_GLOBAL_TASK_TAG = 9;       // 全局TASK+TAG限速
    public static final int RATE_LIMIT_GLOBAL_TASK_TAG_HOST = 10; // 全局TASK+TAG+IP限速
    
    // 日志类型常量（同TaskCronerConfig）
    
    private String taskName;              // 任务名称
    private String taskDesc;              // 任务描述
    private String taskClass;             // 执行类名
    private String taskTag;               // 任务标签
    private int queueType;                // 队列类型
    private int delayType;                // 延迟类型
    private int consumerNum;              // 消费者数量，默认1
    private int prefetchNum;              // 预取任务数，默认1
    private int rateLimitType;            // 限速类型
    private int rateLimitValue;           // 流量限定数值，默认10次
    private int rateLimitTime;            // 流量限定时间（秒），默认1秒
    private int rateLimitWait;            // 流量限制等待秒数，默认30秒
    private int retryTimesByOverrated;    // 超流量限制重试次数，默认0
    private int retryTimesByPartner;      // 对方接口错误重试次数，默认0
    private int logLevel;                 // 日志级别
    private int logLimitSize;             // 日志大小限制
    
    // Builder模式
    public static Builder builder();
    public static Builder builder(String taskName);
}
```

**队列任务示例**：
```java
// 定义任务参数
@Data
public class OrderNotifyParam implements Serializable {
    private Long orderId;
    private Long userId;
    private String notifyType;
}

// 定义返回结果
@Data
public class NotifyResult implements Serializable {
    private boolean success;
    private String message;
}

@Component
public class OrderNotifyTask extends TaskRunner<OrderNotifyParam, NotifyResult> {
    
    @Autowired
    private NotificationService notificationService;
    
    @Override
    public NotifyResult runTask(TaskData<OrderNotifyParam, NotifyResult> taskData) throws Exception {
        OrderNotifyParam param = taskData.getTaskParam();
        
        // 执行业务逻辑
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
        config.setConsumerNum(5);          // 5个消费者
        config.setPrefetchNum(1);          // 每次预取1条
        config.setRateLimitType(TaskRunnerConfig.RATE_LIMIT_GLOBAL_TASK);  // 全局任务限速
        config.setRateLimitValue(100);     // 限速100次
        config.setRateLimitTime(60);       // 每60秒
        config.setRetryTimesByPartner(3);  // 第三方错误重试3次
        config.setLogLevel(TaskRunnerConfig.TASK_LOG_TYPE_RECORD_ALL);
        return config;
    }
    
    @Override
    public TaskContact initContact() {
        return TaskContact.builder("通知服务负责人")
            .email("notify@example.com")
            .build();
    }
}
```

#### TaskFactory 任务执行

**TaskFactory API**：

```java
public class TaskFactory {
    // 获取全局唯一实例
    public static TaskFactory getInstance();
    
    // 发送到延迟队列
    public void sendToQueue(final TaskData<?, ?> taskData);
    
    // 本地运行队列（优先本地执行，本地线程池满时转队列）
    public void runQueue(final TaskData<?, ?> taskData);
    
    // 同步执行任务（阻塞等待结果）
    public <TP, RD> TaskData<TP, RD> runTask(final TaskData<TP, RD> taskData);
    
    // 本地同步执行任务
    public <TP, RD> TaskData<TP, RD> runTaskLocal(final TaskData<TP, RD> taskData);
    
    // 异步执行任务（返回Future）
    public <TP, RD> Future<TaskData<TP, RD>> runTaskAsync(final TaskData<TP, RD> taskData);
    
    // 获取队列信息 [消息数量, 消费者数量]
    public int[] getQueueInfo(String queueName);
    
    // 清除队列，返回被清除的消息数
    public int purgeQueue(String queueName);
}
```

**任务调用示例**：
```java
@Service
public class OrderService {
    
    @Autowired
    private TaskFactory taskFactory;
    
    // 异步发送队列任务
    public void sendOrderNotify(Long orderId, Long userId) {
        OrderNotifyParam param = new OrderNotifyParam();
        param.setOrderId(orderId);
        param.setUserId(userId);
        param.setNotifyType("ORDER_CREATED");
        
        TaskData<OrderNotifyParam, NotifyResult> taskData = TaskData
            .builder(OrderNotifyTask.class, param)
            .refId(orderId)           // 设置关联ID
            .refTag("ORDER_NOTIFY")   // 设置关联TAG
            .taskDelay(5000)          // 延迟5秒执行
            .build();
        
        // 发送到队列
        taskFactory.sendToQueue(taskData);
    }
    
    // 同步执行任务并获取结果
    public NotifyResult syncNotify(Long orderId, Long userId) {
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
        } else {
            throw new RuntimeException("通知失败: " + result.getErrorInfo());
        }
    }
    
    // 异步执行任务
    public void asyncNotify(Long orderId, Long userId) throws Exception {
        TaskData<OrderNotifyParam, NotifyResult> taskData = TaskData
            .builder(OrderNotifyTask.class, new OrderNotifyParam(orderId, userId, "ORDER_ASYNC"))
            .build();
        
        Future<TaskData<OrderNotifyParam, NotifyResult>> future = taskFactory.runTaskAsync(taskData);
        
        // 稍后获取结果
        TaskData<OrderNotifyParam, NotifyResult> result = future.get(30, TimeUnit.SECONDS);
    }
}
```

#### 任务异常处理

```java
// 第三方接口异常（会触发重试）
public class TaskPartnerException extends RuntimeException {
    public TaskPartnerException(String msg);
    public TaskPartnerException(String msg, Throwable cause);
}

// 数据异常（不会触发重试）
public class TaskDataException extends RuntimeException {
    public TaskDataException(String msg);
    public TaskDataException(String msg, Throwable cause);
}
```

**异常使用示例**：
```java
@Override
public NotifyResult runTask(TaskData<OrderNotifyParam, NotifyResult> taskData) throws Exception {
    OrderNotifyParam param = taskData.getTaskParam();
    
    try {
        // 调用第三方接口
        Response response = thirdPartyApi.send(param);
        
        if (response.getCode() == 429) {
            // 限流错误，抛出自定义异常触发重试
            throw new TaskPartnerException("第三方接口限流");
        }
        
        if (response.getCode() == 400) {
            // 参数错误，数据异常不触发重试
            throw new TaskDataException("参数错误: " + response.getMessage());
        }
        
        return new NotifyResult(true, "成功");
        
    } catch (IOException e) {
        // 网络异常，触发重试
        throw new TaskPartnerException("网络异常", e);
    }
}
```

#### TaskContact 联系人配置

```java
public class TaskContact implements Serializable {
    private String taskClass;     // 任务类名
    private String contactName;   // 联系人姓名
    private String mobile;        // 联系电话
    private String email;         // 联系邮箱
    private String wechat;        // 微信
    private String im;            // 备用IM
    private String notifyUrl;     // 通知链接（钉钉/微信等）
    private String remark;        // 备注
    
    // Builder模式
    public static Builder builder();
    public static Builder builder(String contactName);
}
```

