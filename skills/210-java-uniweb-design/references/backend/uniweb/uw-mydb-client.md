### uw-mydb-client — 数据库客户端

**Maven 坐标**: `com.umtone:uw-mydb-client`

MyDB 数据库运维中心客户端，用于动态分配 SAAS 数据节点，实现分库分表的数据库路由。

#### MydbClientHelper API

```java
public class MydbClientHelper {
    
    /**
     * 分配SAAS节点（使用默认配置）
     * 
     * @param saasId 运营商ID
     * @return ResponseData<DataNode> 
     *         - SUCCESS: 正常创建节点
     *         - WARN: 系统已存在节点
     *         - ERROR: 创建失败
     */
    public static ResponseData<DataNode> assignSaasNode(Serializable saasId);
    
    /**
     * 分配SAAS节点（指定预设节点）
     * 
     * @param saasId     运营商ID
     * @param preferNode 预设节点名
     * @return ResponseData<DataNode>
     */
    public static ResponseData<DataNode> assignSaasNode(Serializable saasId, String preferNode);
    
    /**
     * 分配SAAS节点（指定配置组）
     * 
     * @param configKey  配置组key（默认"default"）
     * @param saasId     运营商ID
     * @param preferNode 预设节点名
     * @return ResponseData<DataNode>
     */
    public static ResponseData<DataNode> assignSaasNode(String configKey, Serializable saasId, String preferNode);
}
```

#### DataNode 数据节点

```java
public class DataNode {
    private long clusterId;    // MySQL集群ID
    private String database;   // 数据库名
    
    public DataNode(long clusterId, String database);
    public DataNode(String dataNodeKey);  // 格式: "clusterId.database"
    
    public String toString();  // 返回: "clusterId.database"
}
```

#### 使用示例

```java
    /**
     * 为新租户分配数据库节点
     */
    public void initSaasDatabase(long saasId) {
        // 自动分配节点
        ResponseData<DataNode> response = MydbClientHelper.assignSaasNode(saasId);
        
        if (response.isSuccess()) {
            DataNode node = response.getData();
            log.info("创建新节点: 集群={}, 库名={}", node.getClusterId(), node.getDatabase());
        } else if (response.isWarn()) {
            DataNode node = response.getData();
            log.warn("节点已存在: 集群={}, 库名={}", node.getClusterId(), node.getDatabase());
        } else {
            log.error("节点分配失败: {}", response.getMsg());
        }
    }
    
    /**
     * 指定集群和预设节点名分配
     */
    public void assignToSpecificCluster(long saasId) {
        ResponseData<DataNode> response = MydbClientHelper.assignSaasNode(
            "cluster-a",      // 配置组
            saasId,           // 运营商ID
            "db_shard_01"     // 预设节点名
        );
    }
```

