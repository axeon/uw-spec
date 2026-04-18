## 开发新 SaaS 应用的步骤

### 创建项目结构

```
{project-name}/
├── pom.xml
├── {project-name}-app/
│   ├── pom.xml
│   └── src/
│       └── main/
│           ├── java/{project-name}/   
│           │   ├── aip/           # AIP Vendor
│           │   ├── ais/           # AIS Linker（可选）
│           │   ├── constant/      # 常量
│           │   ├── controller/    # 控制器
│           │   ├── entity/        # 实体
│           │   └── helper/        # 帮助类
│           └── resources/
└── database/
    └── ddl.sql
```

### 配置 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>saas</groupId>
    <artifactId>{project-name}</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>{project-name}-app</module>
    </modules>
</project>
```

### 应用模块 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.umtone</groupId>
        <artifactId>uw-base</artifactId>
        <version>2025.1101.0003</version>
    </parent>
    
    <groupId>saas</groupId>
    <artifactId>{project-name}-app</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <!-- SaaS 基础公共模块 -->
        <dependency>
            <groupId>saas</groupId>
            <artifactId>saas-base-common</artifactId>
            <version>2.2.210</version>
        </dependency>
        
        <!-- 财务客户端（如需支付功能） -->
        <dependency>
            <groupId>saas</groupId>
            <artifactId>saas-finance-client</artifactId>
            <version>2.0.118</version>
        </dependency>
    </dependencies>
</project>
```

### 实现 AIP Vendor

```java
@Service
public class YourAppVendor implements AipAppVendor {
    @Override
    public ResponseData initData(AipVendorExecuteParam runParam) {
        // 初始化数据表和默认数据
        return ResponseData.success();
    }

    @Override
    public ResponseData destroyData(AipVendorExecuteParam runParam) {
        // 清理数据
        return ResponseData.success();
    }

    @Override
    public String name() {
        return "你的应用名称";
    }

    @Override
    public String code() {
        return "";
    }
}
```

### 创建启动类

```java
@SpringBootApplication
@EnableFeignClients
public class YourAppApplication {
    public static void main(String[] args) {
        SpringApplication.run(YourAppApplication.class, args);
    }
}
```

### 配置 bootstrap.yml

```yaml
spring:
  application:
    name: saas-{your-app}-app
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_SERVER}
        namespace: ${NACOS_NAMESPACE}
      config:
        server-addr: ${NACOS_SERVER}
        namespace: ${NACOS_NAMESPACE}
        file-extension: yml
```


