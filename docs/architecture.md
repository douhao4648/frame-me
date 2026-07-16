# 架构设计

## 总工程结构

```
Frame_Me/
├── CLAUDE.md              # 总工程入口
├── docs/                  # 总工程知识库
├── frame-me-parent/       # 脚手架父工程
│   ├── CLAUDE.md
│   └── docs/
└── fm-demo/               # 演示聚合服务
    ├── CLAUDE.md
    └── docs/
```

## 分层原则

- 每个子工程独立演进，独立构建、测试、部署。
- 子工程之间通过接口契约、事件或 RPC 协作，不直接依赖彼此内部实现。
- 总工程提供跨工程约定索引，不承载业务代码。

## 跨服务调用约定

- 服务间调用统一用 **Spring HTTP Interface**：在服务方的 `xx-api` 模块用 `@HttpExchange` 声明接口契约（如 `frame-me-tester-api` 的 `IDemoApi`）。
- 消费方在 `infrastructure/client/**` 下定义客户端接口，并通过配置类（`@ImportHttpServices`）装配为可注入的 HTTP 客户端。范例见 fm-demo：`com.fm.demo.infrastructure.client.tester.TesterDemoClient` + `infrastructure.config.DemoConfiguration` 调用 `frame-me-tester`。
- 契约共享：消费方直接依赖服务方的 `xx-api` 复用其 DTO/Query/VO，不重复定义。
- 异常透传：远端统一 `Result`/`Response` 结构原样回传，由消费方按统一响应规范处理。
- 跨服务事件：进程内用 Spring `ApplicationEvent`（`MeApplicationEvent`）解耦，跨服务通过可插拔 transport（Redis Pub/Sub，MQ 可扩展）桥接，统一开关 `me.event-bridge.*`。审计日志、SSE/WS 推送均基于此机制。详见 [frame-me-parent/docs/guides/event-bridge.md](../frame-me-parent/docs/guides/event-bridge.md)。
- 鉴权：`frame-me-starter-auth` 已实现认证抽象层（`AuthContext`、`@LoginUser`/`@Anonymous`、`IAuthService`/`IAuthUserResolver` SPI、默认请求头兜底解析器，已纳入 booter）；具体实现按需引入 `frame-me-starter-auth-jwt`（JWT 登录/刷新）与 `frame-me-starter-auth-rbac`（RBAC 授权 + 数据权限）。
- 服务发现、网关、链路追踪等云组件能力预留在 `frame-me-starter-cloud`（当前为占位），后续接入 Nacos/Gateway/Sentinel 时在此补充。

## 脚手架横切能力

`frame-me-parent` 通过 `frame-me-booter` 为业务 `xx-service` 默认提供以下横切能力，引入 `frame-me-booter` 即可一键获得（详见 [frame-me-parent/docs/modules.md](../frame-me-parent/docs/modules.md)）：

| 能力 | starter | 默认 | 说明 |
|---|---|---|---|
| Web 基础设施 | `frame-me-starter-base` | 开启（传递引入） | 统一响应/异常、事件桥接、HTTP Interface 客户端、池化 RestClient、异步/调度线程池 |
| 认证抽象 | `frame-me-starter-auth` | 开启 | `AuthContext`、`@LoginUser`/`@Anonymous`、认证 SPI，默认请求头兜底实现 |
| Redis / 分布式锁 | `frame-me-starter-multi-redis` | 开启 | `RedisUtils` + 可选 Redisson 高阶能力 |
| 两级缓存 | `frame-me-starter-l1l2-cache` | 关闭 | JetCache：Caffeine(L1) + Redis(L2)，`me.cache.enabled` 开启 |
| 审计/行为日志 | `frame-me-starter-op-audit` | 开启 | `@AuditLog` 注解记录操作，可经事件桥接发往审计服务 |
| 配置密钥加密 | `frame-me-starter-sensi-encrypt` | 开启 | Jasypt 解密配置中的 `ME(密文)`，主密码由环境变量注入 |
| SSE 推送 | `frame-me-starter-sse-mvc` | 开启 | `me.sse.enabled`，按事件类型广播 / 按接收者定向推送 |
| 消息通知 | `frame-me-starter-msg-notify` | 开启 | `INotifySender` 统一发送，邮件 / Webhook / 短信多通道 |
| 云组件 | `frame-me-starter-cloud` | 占位 | 预留 Nacos/Gateway/Sentinel |

按需引入（不纳入 `frame-me-booter`）：

- 数据访问 `frame-me-starter-mybatis-plus` 或 `frame-me-starter-mybatis-flex`（二选一）。
- 多数据源 `frame-me-starter-dynamic-ds`（baomidou dynamic-datasource，`@DS` 切换）。
- RBAC 授权 `frame-me-starter-auth-rbac`（`@RequireAuth` SpEL、数据权限、可选 Redis 权限后端）。
- JWT 认证 `frame-me-starter-auth-jwt`（接管 auth 抽象层，登录/登出/刷新接口）。
- 接口文档 `frame-me-starter-doc-openapi`（SpringDoc OpenAPI，`me.swagger.enabled` 开启）。
- 老接口规范适配 `frame-me-adapter-starter`（`IResult` → `Response` 转换，可被业务自定义适配层替换）。
- WebSocket `frame-me-starter-ws-mvc`（`me.ws.mvc.enabled`，原生 WebSocket 全双工，广播 / 定向推送）。
