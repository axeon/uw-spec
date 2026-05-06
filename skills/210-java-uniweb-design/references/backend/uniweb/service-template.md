# 新建微服务模板

## 标准项目骨架

```
demo/
├── .gitea/
│   └── workflows/
│       └── build.yml               # CI/CD 流水线
├── database/
│   └── ddl.sql                     # 数据库建表 DDL
├── src/
│   └── main/
│       ├── java/demo/
│       │   ├── MainApplication.java  # Spring Boot 入口
│       │   ├── conf/                         # 配置类
│       │   ├── controller/
│       │   │   ├── admin/                    # ADMIN 用户接口
│       │   │   ├── saas/                     # SAAS 用户接口
│       │   │   ├── mch/                      # MCH 用户接口
│       │   │   ├── guest/                    # GUESTS 用户接口
│       │   │   ├── ops/                      # OPS 用户接口
│       │   │   └── rpc/                      # RPC 内部调用接口
│       │   ├── entity/                       # 数据实体
│       │   ├── dto/                          # 查询参数/DTO
│       │   └── service/                      # 业务逻辑
│       │   └── vo/                           # 视图对象
│       └── resources/
│           ├── bootstrap.yml                 # Nacos + 应用配置
│           ├── bootstrap-debug.yml           # 本地开发配置
│           └── logback-spring.xml            # 日志配置
├── Dockerfile                               # Docker 构建文件
├── README.md                                # 项目说明
└── pom.xml                                  # Maven 项目描述
```

## 参考实现

demo使用的基本功能
- uw-dao 的多数据源配置和 CRUD 操作
- uw-auth-service 的权限声明和鉴权
- uw-cache 的缓存使用
- uw-task 的定时任务和队列任务
- uw-log-es 的日志记录
- uw-common-app 的启动入口、 QueryParam 和数据历史
