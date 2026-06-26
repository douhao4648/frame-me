# 架构设计

## 模块/组件依赖图

```
┌─────────────────────────────────────────────┐
│              fm-demo (聚合 POM)               │
│  ┌─────────────┐    ┌─────────────────────┐  │
│  │ fm-demo-api │    │   fm-demo-service   │  │
│  │  (契约/DTO) │◄───│  (启动 + 实现)     │  │
│  └──────┬──────┘    └──────────┬──────────┘  │
│         │                      │              │
│         │         ┌────────────┴──────────┐  │
│         │         │   frame-me-booter     │  │
│         │         │  (传递各 starter 能力) │  │
│         │         └──────────┬────────────┘  │
│         │                    │               │
│         │    ┌───────────────┼──────────┐    │
│         │    │               │          │    │
│         ▼    ▼               ▼          ▼    │
│  frame-me-api    frame-me-starter-*    ...   │
│  (Result/VO)                                 │
└─────────────────────────────────────────────┘
```

更准确的依赖关系：

- `fm-demo` 聚合 `fm-demo-api` 与 `fm-demo-service`。
- `fm-demo-service` 依赖 `fm-demo-api` 和 `frame-me-booter`。
- `frame-me-booter` 进一步集成 `frame-me-starter-auth`、`frame-me-starter-cloud`、`frame-me-starter-dynamic-ds` 等 starter。

## 分层原则

项目采用经典四层分层：

```
Controller 层（接口适配）
    ├── DemoController          — 实现本地 IDemoApi，参数校验 + 委托 Service
    └── TesterDemoController    — 实现 ITesterDemoApi，委托 TesterDemoClient 做远程透传

Service 层（业务逻辑）
    ├── IDemoService            — 业务接口定义
    └── DemoServiceImpl         — 业务实现（分页查询、乐观锁校验、实体转换）

Mapper 层（数据访问）
    └── DemoMapper              — 继承 BaseMapper，零 XML

Entity 层（领域模型）
    └── DemoEntity              — 继承 BaseVersionEntity，对应表 demo_user
```

转换层：`DemoConvert`（MapStruct）负责 Entity / DTO / VO 之间的类型转换，由 Service 层调用。

## 初始化 / 自动装配机制

`fm-demo-service` 引入 `frame-me-booter`（Maven 依赖），该模块通过 Spring Boot 的 `spring.factories` / `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 自动装配以下能力：

- 统一响应包装（`Result` / `IResult`）
- 统一异常处理（`GlobalExceptionHandler`）
- MyBatis-Plus 配置（分页插件、元对象处理器 `MetaObjectHandler`）
- 雪花 ID 生成器（`SnowflakeUtils`）
- Swagger / SpringDoc 分组配置

手动开启的配置：
- `DemoConfiguration` 使用 `@ImportHttpServices(group = "tester", basePackages = "com.fm.demo.infrastructure.client.tester")` 装配 `TesterDemoClient`。
- `application.yml` 中 `frame.me.mybatis.meta-object-handler.enabled: true` 开启自动填充 create_time / update_time。
- `spring.sql.init.mode: always` 确保 H2 内存库启动时执行 `schema.sql` 建表。

## 响应与异常流水线

### 正常请求

以 `GET /api/demo/{id}` 为例：

1. **请求入口**：`DemoController.getById(Long id)` 接收请求。
2. **参数校验**：`@Positive` 校验 id > 0，失败时由 `MethodArgumentNotValidException` 抛出。
3. **业务处理**：`DemoController` 委托 `IDemoService.getById(id)`。
4. **数据访问**：`DemoServiceImpl` 调用 `DemoMapper.selectById(id)` 查询 `DemoEntity`。
5. **实体转换**：`DemoServiceImpl` 通过 `DemoConvert.toVo(entity)` 转为 `DemoVO`。
6. **响应包装**：`DemoController` 使用 `Result.success(vo)` 包装为 `IResult<DemoVO>` 返回。
7. **HTTP 输出**：Spring 自动序列化为 JSON，结构如 `{"code":200,"data":{...},"message":"success"}`。

分页查询 `page` 额外经过 `PageUtils.toPage(query, "create_time:desc")` 构建分页对象，再经 `PageUtils.toPageData(...)` 包装为 `PageData<DemoVO>`。

### 异常请求

异常统一由 `frame-me-booter` 传递的 `GlobalExceptionHandler` 捕获并转换，fm-demo 本身不重复定义：

- **业务异常**：`BusinessException`（如数据不存在）→ 返回对应 `ResultCode` 的响应。
- **参数校验异常**：`MethodArgumentNotValidException`、`ConstraintViolationException` → 返回 `ResultCode.BAD_REQUEST`（400）。
- **其他异常**：统一返回 `ResultCode.INTERNAL_ERROR`（500）。

具体异常体系与处理器路径见 `../../frame-me-parent/docs/conventions.md`。

### 跨服务调用链路

`TesterDemoClient` 是 fm-demo 作为消费方调用 `frame-me-tester` 的演示链路：

1. **接口定义**：`TesterDemoClient` 继承 `com.frame.me.tester.api.IDemoApi`（来自 `frame-me-tester-api` 依赖）。
2. **装配配置**：`DemoConfiguration` 使用 `@ImportHttpServices(group = "tester", basePackages = "com.fm.demo.infrastructure.client.tester")`，Spring HTTP Interface 自动扫描并生成代理 Bean。
3. **调用入口**：`TesterDemoController` 实现 `ITesterDemoApi`，将请求委托给 `TesterDemoClient`。
4. **远程地址**：`application.yml` 中 `spring.http.serviceclient.tester.base-url: http://localhost:9090` 指定目标服务地址。
5. **超时配置**：`spring.http.clients.connect-timeout: 3s`、`read-timeout: 10s` 为全局默认值；`tester` 分组可单独覆盖 `read-timeout: 5s`。
6. **结果透传**：`TesterDemoController` 直接将 `TesterDemoClient` 返回的 `IResult` 透传给调用方，不做额外转换。

### 关键类路径

| 类/接口 | 路径 | 说明 |
|---|---|---|
| `Result` / `IResult` | `frame-me-api`（父工程） | 统一响应包装 |
| `PageData` | `frame-me-api`（父工程） | 分页响应结构 |
| `BusinessException` | `frame-me-base`（父工程） | 业务异常基类 |
| `GlobalExceptionHandler` | `frame-me-booter`（父工程） | 统一异常处理器 |
| `BaseVersionEntity` | `frame-me-base`（父工程） | 带乐观锁的实体基类 |
| `BaseMapper` | MyBatis-Plus | Mapper 基类 |
| `SnowflakeUtils` | `frame-me-base`（父工程） | 雪花 ID 生成器 |
| `PageUtils` | `frame-me-base`（父工程） | 分页对象转换工具 |
| `DemoConfiguration` | `com.fm.demo.infrastructure.config.DemoConfiguration` | HTTP 客户端装配配置 |
| `TesterDemoClient` | `com.fm.demo.infrastructure.client.tester.TesterDemoClient` | 声明式 HTTP 客户端 |

## 扩展提示

- **新增业务模块**：参照 `DemoController → IDemoService → DemoServiceImpl → DemoMapper → DemoEntity` 分层，新建对应包与类。
- **新增跨服务客户端**：新建接口继承目标服务的 API 接口，在 `DemoConfiguration` 的 `@ImportHttpServices` 中增加 `basePackages` 路径。
- **自定义 starter 接入**：在 `fm-demo-service/pom.xml` 引入目标 starter，配置项写入 `application.yml`，由 `frame-me-booter` 自动装配。
- **扩展数据源**：`frame-me-booter` 已集成 `frame-me-starter-dynamic-ds`，在 `application.yml` 中按动态数据源配置格式增加即可。
- **扩展统一异常**：继承 `BusinessException` 或自定义异常，由 `GlobalExceptionHandler`（父工程）统一处理，无需在 fm-demo 中新增处理器。
