# 编码约定

## 适用范围

本页约定适用于 `Frame_Me` 总工程下所有子工程。若子工程 `docs/conventions.md` 另有说明，优先以子工程为准。

## 响应规范

- 接口契约统一返回 `frame-me-api` 的 `com.frame.me.api.result.IResult<T>`，默认实现为 `frame-me-starter-base` 的 `com.frame.me.base.result.Result<T>`（含静态工厂方法）。
- 分页统一用 `PageQuery`（入参）+ `PageData<T>`（出参）。
- 需对接老接口规范的工程，引入 `frame-me-adapter-starter`，由 `Result2ResponseAdvice` 自动将 `IResult<T>` 转为外部 `Response<T>`。
- 细节见 [frame-me-parent/docs/conventions.md](../frame-me-parent/docs/conventions.md)。

## 异常体系

- 统一以 `frame-me-starter-base` 的 `BusinessException` / `InternalException` / `RetryException` 为基础，由 `GlobalExceptionHandler` 全局捕获并转为统一 `Result`。
- 状态码统一走 `com.frame.me.base.result.ResultCode` 枚举。
- 细节见 [frame-me-parent/docs/conventions.md](../frame-me-parent/docs/conventions.md)。

## 编码风格

- 各子工程优先遵循自身技术栈的主流风格。
- 跨工程公共模块命名、包结构应保持一致。

## 数据访问约定

- 默认使用 MyBatis-Plus（`frame-me-starter-mybatis-plus`），实体继承 `BaseEntity`（雪花 ID、`createTime`/`updateTime`/`deleted` 逻辑删除）或 `BaseVersionEntity`（额外 `version` 乐观锁）；亦可选用 MyBatis-Flex（`frame-me-starter-mybatis-flex`），二者二选一。
- Mapper 必须标注 `@Mapper` 并继承对应的 `BaseMapper<T>`（MyBatis-Plus 或 MyBatis-Flex）。
- **表名到实体名映射**：去掉第一个下划线前缀，如 `spo_fms_device` → `FmsDevice`。
- 多数据源经 `frame-me-starter-dynamic-ds`（baomidou dynamic-datasource），用 `@DS("...")` 切换；默认自动创建 `master` 数据源。
- 细节见 [frame-me-parent/docs/conventions.md](../frame-me-parent/docs/conventions.md) 与 [frame-me-parent/docs/modules.md](../frame-me-parent/docs/modules.md)。

## 基础设施能力

由 `frame-me-parent` 提供，业务工程引入 `frame-me-booter` 默认获得多数据源、Redis、两级缓存、审计日志、配置加密能力，鉴权/云组件为占位，接口文档、老规范适配、SSE/WebSocket 推送按需引入。能力清单与默认开关见 [architecture.md#脚手架横切能力](./architecture.md)，配置细节见 [frame-me-parent/docs/modules.md](../frame-me-parent/docs/modules.md)。

- Redis：统一走 `RedisUtils`（`me.redis.*`），分布式锁默认 `SET NX PX` 简单实现，引入 Redisson 后自动升级为可重入 + 看门狗续期。
- 两级缓存：JetCache `@Cached(cacheType = BOTH)`，默认关闭（`me.cache.enabled=true`），缓存对象需 `Serializable`。
- 审计日志：方法标注 `@AuditLog` 即记录动作/参数/返回/异常/耗时（`me.audit.*`，默认开启打印本地日志），配 `me.audit.target-service` 后经事件桥接发往审计服务。
- 配置加密：敏感值写成 `ME(密文)`，启动时由 Jasypt 解密（`me.encrypt.*`），主密码经环境变量 `ME_ENCRYPT_PASSWORD` 注入、不入库。
- SSE / WebSocket：`frame-me-starter-sse-mvc`（`me.sse.*`）、`frame-me-starter-ws-mvc`（`me.ws.mvc.*`）按需引入，支持按事件类型广播与按接收者定向推送，订阅 `MeApplicationEvent` 自动转发。
- 接口文档：`me.swagger.enabled=true` 启用，按 `groups` 分组匹配路径。

## 子工程独立约定

- `frame-me-parent`：见 [frame-me-parent/docs/conventions.md](../frame-me-parent/docs/conventions.md)
- `fm-demo`：见 [fm-demo/docs/conventions.md](../fm-demo/docs/conventions.md)
