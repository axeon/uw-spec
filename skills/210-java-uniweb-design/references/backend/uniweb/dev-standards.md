## 微服务开发规范

### 项目初始化

所有微服务项目继承 uw-base Parent POM，**不指定子模块版本号**，由 Parent POM 统一管理（版本号请查看 `uw-base/pom.xml` 获取最新值）：

```xml
<parent>
    <groupId>com.umtone</groupId>
    <artifactId>uw-base</artifactId>
    <version>${uw-base.version}</version>
    <relativePath/>
</parent>
```

### 标准依赖组合

```xml
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Spring Cloud Bootstrap -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bootstrap</artifactId>
    </dependency>
    <!-- Nacos 服务注册 -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!-- Nacos 配置中心 -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
    <!-- 负载均衡 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>
    <!-- UW基础组件 -->
    <dependency>
        <groupId>com.umtone</groupId>
        <artifactId>uw-common</artifactId>
    </dependency>
    <dependency>
        <groupId>com.umtone</groupId>
        <artifactId>uw-common-app</artifactId>
    </dependency>
    <dependency>
        <groupId>com.umtone</groupId>
        <artifactId>uw-dao</artifactId>
    </dependency>
    <dependency>
        <groupId>com.umtone</groupId>
        <artifactId>uw-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>com.umtone</groupId>
        <artifactId>uw-auth-service</artifactId>
    </dependency>
    <dependency>
        <groupId>com.umtone</groupId>
        <artifactId>uw-log-es</artifactId>
    </dependency>
    <dependency>
        <groupId>com.umtone</groupId>
        <artifactId>uw-logback-es</artifactId>
    </dependency>
    <!-- API文档 -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    </dependency>
</dependencies>
```

### 配置文件规范

- `bootstrap.yml` — Nacos 配置、应用名称
- `bootstrap-debug.yml` — 本地开发配置（spring profiles: debug）
- `logback-spring.xml` — 日志配置（含 uw-logback-es appender）

**bootstrap.yml 标准模板**：
```yaml
server:
  port: 100XX

spring:
  application:
    name: uw-xxx-center
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_SERVER:127.0.0.1:8848}
        namespace: ${NACOS_NAMESPACE:public}
      config:
        server-addr: ${NACOS_SERVER:127.0.0.1:8848}
        namespace: ${NACOS_NAMESPACE:public}
        file-extension: yml
```

### Controller 路径规范

Controller 第一级路径必须为用户类型名：

```java
@RestController
@RequestMapping("/root/user")       // ROOT 用户访问
@RequestMapping("/ops/template")    // OPS 用户访问
@RequestMapping("/admin/user")      // ADMIN 用户访问
@RequestMapping("/saas/config")     // SAAS 用户访问
@RequestMapping("/mch/order")       // MCH 用户访问
@RequestMapping("/guest/order")     // GUESTS 用户访问
@RequestMapping("/rpc/data")        // RPC 内部调用
```

### 代码生成与开发流程

**代码生成机制**：

entity、dto、controller 基础代码通过 uw-code-center 工具自动生成。开发人员必须对自动生成的代码进行裁剪：

- **DTO 裁剪**：精确控制搜索条件字段和排序字段，确保接口参数符合业务需求
- **Controller 裁剪**：实现权限控制逻辑，移除不需要的功能接口，优化接口设计

**开发重点与代码组织**：

- 开发核心为 **service 层** 和 **controller 层**实现
- 针对非 CRUD 的复杂业务逻辑，必须封装在 **service 层**或 **Helper 工具类**中，供 controller 层调用
- 如业务需要使用 VO 对象，统一在 **vo 包**下创建并管理

### 权限声明规范

所有接口必须使用 `@MscPermDeclare` 注解声明权限：

```java
@RestController
@RequestMapping("/admin/user")
@MscPermDeclare(name = "用户管理", user = UserType.ADMIN, auth = AuthType.PERM)
public class UserController {

    @GetMapping("/list")
    @MscPermDeclare(name = "用户列表", log = ActionLog.REQUEST)
    public ResponseData<DataList<User>> list(AuthQueryParam param) {
        // ...
    }

    @PostMapping("/save")
    @MscPermDeclare(name = "保存用户", log = ActionLog.ALL)
    public ResponseData<User> save(@RequestBody User user) {
        // ...
    }
}
```

**$PackageInfo$ 类**：

每个 controller 包中必须包含 `$PackageInfo$` 类，这是 uw-auth 权限目录自动构建的必要组件。该类用于声明包级别的权限元数据，必须确保该类存在且配置正确，否则将导致权限目录无法自动构建。

```java
@MscPermDeclare(name = "用户管理模块", user = UserType.ADMIN)
package uw.xxx.center.controller.admin;

import uw.auth.service.anno.MscPermDeclare;
import uw.auth.service.vo.UserType;
```

### 响应格式规范

所有 Controller 方法返回值必须使用 `ResponseData` 包装：

```java
// 单个对象
public ResponseData<User> getUser() { ... }

// 分页列表
public ResponseData<DataList<User>> listUsers() { ... }

// 无数据返回
public ResponseData saveUser() { ... }
```

### 实体类规范

实体类继承 `DataEntity`，使用 `@TableMeta` 和 `@ColumnMeta` 注解：

```java
@TableMeta(tableName = "my_table")
public class MyEntity extends DataEntity {
    @ColumnMeta(columnName = "id", primaryKey = true)
    private long id;

    @ColumnMeta(columnName = "name")
    private String name;

    // getters/setters
}
```

### Docker 部署规范

每个微服务都有 Dockerfile，使用 BellSoft Liberica JRE 25 CDS 优化镜像：

```dockerfile
FROM bellsoft/liberica-runtime-container:jre-cds-25
COPY target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### CI/CD 规范

使用 Gitea Actions，每个项目 `.gitea/workflows/build.yml` 配置自动构建和部署流水线。

### 数据访问规范

- 优先使用 `DaoManager`（ResponseData 链式lambda风格），避免使用 `DaoFactory`（抛异常风格）
- 查询参数使用 QueryParam 系列类（`IdQueryParam`、`AuthQueryParam`、`AuthIdStateQueryParam` 等）
- 批量操作使用 `BatchUpdateManager`

### 缓存使用规范

- 高频读取场景使用 `FusionCache`（本地 + Redis 融合缓存）
- Kryo 序列化**必须使用具体实现类**，不能使用接口类型（如 List/Map/Set），必须传 ArrayList/LinkedHashMap/HashSet 等
- Caffeine 设定过期时间后性能劣化 200 倍，建议仅设 Redis 过期时间，不设 Caffeine 本地过期

### 认证授权规范

- 内部 RPC 调用使用 `@Qualifier("authRestTemplate")` 注入带鉴权的 RestTemplate
- Token 自动管理（自动 login/refresh/重试），无需手动设置 Authorization 请求头
- 收到 401 或 498 时自动刷新 Token 并重试一次

### 配置文件检查清单

确保配置文件包含必要的前缀和参数：
- `uw.dao.*` — uw-dao配置
- `uw.cache.*` — uw-cache缓存配置
- `uw.auth.service.*` — uw-auth 认证服务配置
- `uw.auth.client.*` — uw-auth RPC客户端配置
- `uw.log.es.*` — ES 日志地址（如需要）
- `uw.task.*` — uw-task 任务框架配置（如需要）

---

