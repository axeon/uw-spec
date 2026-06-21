# uw-mydb-client — 数据库客户端

**Maven 坐标**: `com.umtone:uw-mydb-client`
**配置前缀**: `uw.mydb`

MyDB 数据库运维中心客户端，通过 HTTP RPC 调用 mydb-center 动态分配运营商（SAAS）数据节点，实现分库分表路由。

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 分配SAAS节点(自动) | `MydbClientHelper.assignSaasNode(saasId)` | 默认配置组 "default" |
| 分配SAAS节点(指定节点) | `MydbClientHelper.assignSaasNode(saasId, preferNode)` | preferNode 可为 null |
| 分配SAAS节点(指定配置组) | `MydbClientHelper.assignSaasNode(configKey, saasId, preferNode)` | configKey 默认 "default" |

## MydbClientHelper 方法签名

> **包路径**：`uw.mydb.client.MydbClientHelper`

全部静态方法。共享 `RestClient` 由 `uw-auth-client` 提供（带鉴权拦截器）。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `assignSaasNode(Serializable saasId)` | `ResponseData<DataNode>` | 默认配置组，自动分配 |
| `assignSaasNode(Serializable saasId, String preferNode)` | `ResponseData<DataNode>` | 默认配置组，指定偏好节点 |
| `assignSaasNode(String configKey, Serializable saasId, String preferNode)` | `ResponseData<DataNode>` | 指定配置组与偏好节点 |

返回语义：`SUCCESS`=新建节点 / `WARN`=节点已存在（`data` 仍可用） / `ERROR`=分配失败。任何异常均被捕获并以 `ResponseData.errorMsg` 返回，**不会抛出**。

## DataNode

> **包路径**：`uw.mydb.client.vo.DataNode`

构造：`new DataNode(long clusterId, String database)` 或 `new DataNode("clusterId.database")`

| 字段 | 类型 | 说明 |
|------|------|------|
| clusterId | long | MySQL集群ID |
| database | String | 数据库名 |

`toString()` 返回 `"clusterId.database"`；字符串构造含格式校验，非法抛 `IllegalArgumentException`。

## Helper 使用示例

```java
public class DatabaseHelper {
    /**
     * 为新租户分配数据库节点
     */
    public DataNode initSaasDatabase(long saasId) {
        // 自动分配节点
        ResponseData<DataNode> response = MydbClientHelper.assignSaasNode(saasId);
        if (response.isSuccess()) {
            DataNode node = response.getData();
            log.info("创建新节点: 集群={}, 库名={}", node.getClusterId(), node.getDatabase());
            return node;
        } else if (response.isWarn()) {
            DataNode node = response.getData();
            log.warn("节点已存在: 集群={}, 库名={}", node.getClusterId(), node.getDatabase());
            return node;
        } else {
            log.error("节点分配失败: {}", response.getMsg());
            throw new RuntimeException("节点分配失败: " + response.getMsg());
        }
    }

    /**
     * 指定集群和预设节点名分配
     */
    public ResponseData<DataNode> assignToSpecificCluster(long saasId) {
        return MydbClientHelper.assignSaasNode(
            "cluster-a",      // 配置组
            saasId,           // 运营商ID
            "db_shard_01"     // 预设节点名
        );
    }
}
```

## 接入步骤

1. 引入依赖 `com.umtone:uw-mydb-client`（依赖链含 `uw-auth-client`）。
2. 按需配置 `uw.mydb.mydb-center-host`（默认 `http://uw-mydb-center`）。
3. 确保容器中存在 `authRestClient`（由 `uw-auth-client` 提供）。
4. 启动后自动装配，直接调用静态方法。
