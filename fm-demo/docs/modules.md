# 模块速查

## 模块依赖速查表

| 模块/组件 | 直接依赖 | 职责 |
|---|---|---|
| fm-demo-api | 无 | 对外暴露的接口契约与 DTO |
| fm-demo-service | fm-demo-api、frame-me-booter | 启动服务，集成 Frame Me 各 starter 能力 |

## 详细说明

### fm-demo-api

职责：对外暴露 fm-demo 的 HTTP 接口契约与数据传输对象，供内部 service 实现及外部服务引用。

关键包结构：

```
com.fm.demo.api
├── IDemoApi.java          — 本地演示 CRUD 接口契约
├── ITesterDemoApi.java    — 跨服务调用演示接口契约（透传 frame-me-tester）
├── dto
│   └── DemoDTO.java       — 演示数据请求 DTO（含 CreateGroup/UpdateGroup 校验分组）
├── query
│   └── DemoQuery.java     — 演示数据分页查询参数（继承 PageQuery）
└── vo
    └── DemoVO.java        — 演示数据返回 VO
```

对外接口：
- `IDemoApi`：本地 CRUD，路径前缀 `/api/demo`，包含 list / page / getById / create / update / delete 6 个方法。
- `ITesterDemoApi`：跨服务透传，路径前缀 `/api/tester-demo`，方法签名与 frame-me-tester 的 `IDemoApi` 对齐，用于验证 Spring HTTP Interface 客户端链路。

核心依赖：`frame-me-api`（Result/IResult/PageData）、`frame-me-tester-api`（跨服务调用 DTO/VO/Query）、`spring-web`（`@HttpExchange` 等注解）。

### fm-demo-service

职责：Spring Boot 启动服务，实现 `fm-demo-api` 定义的接口，集成 `frame-me-booter` 获得通用能力（统一响应、异常处理、MyBatis-Plus、雪花 ID、分页等）。

启动类路径：`com.fm.demo.Application`

关键包结构：

```
com.fm.demo
├── Application.java
├── controller
│   ├── DemoController.java        — 实现 IDemoApi，委托 IDemoService
│   └── TesterDemoController.java  — 实现 ITesterDemoApi，委托 TesterDemoClient
├── entity
│   └── DemoEntity.java            — 对应表 demo_user，继承 BaseVersionEntity
├── mapper
│   └── DemoMapper.java            — 继承 BaseMapper<DemoEntity>
├── service
│   ├── IDemoService.java          — 业务接口
│   ├── impl
│   │   └── DemoServiceImpl.java   — 业务实现（含分页、乐观锁校验）
│   └── convert
│       └── DemoConvert.java       — MapStruct 转换器（entity <-> VO/DTO）
└── infrastructure
    ├── client
    │   └── tester
    │       └── TesterDemoClient.java  — 声明式 HTTP 客户端（extends frame-me-tester 的 IDemoApi）
    └── config
        └── DemoConfiguration.java     — @ImportHttpServices 装配 tester 客户端
```

核心配置项（`fm-demo-service/src/main/resources/application.yml`）：
- `server.port`：8080
- `management.server.port`：8081
- `spring.datasource`：H2 内存数据库（`jdbc:h2:mem:fm_demo`）
- `spring.sql.init.schema-locations`：`classpath:schema.sql`
- `mybatis-plus.global-config.db-config`：逻辑删除字段 `deleted`、主键策略 `assign_id`（雪花 ID）
- `spring.http.serviceclient.tester.base-url`：http://localhost:9090（frame-me-tester 地址）
- `spring.jackson.default-property-inclusion`：`non_null`，全局 JSON 序列化忽略 null 字段（跨服务透传时避免传递空字段）
- `frame.me.swagger`：SpringDoc 分组配置（`demo-api`，匹配 `/api/**`）
