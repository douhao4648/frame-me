# Frame_Me

`Frame_Me` 是本仓库的总工程目录，用于聚合多个独立子工程。每个子工程在独立子目录中维护，拥有各自的构建、测试与部署能力。

## 定位与 JDK 版本策略

本架构定位是**支撑未来 5-10 年中小公司企业级架构的基础框架**。当前为更好地兼容云产品（如阿里云），Java 运行时环境暂定 **JDK 21**；架构本身已按 JDK 25 LTS 就绪设计，后续可**无感升级至 JDK 25 LTS**。

同时，本框架也是 **frame-me 系列「超级个体」定位探索与实施**的重要一环。后续除服务端脚手架和示例工程外，还将陆续推出：

- 前端脚手架
- 微信小程序
- iOS / Android 移动端
- 桌面端应用
- 企业级 AI Agent 等探索工程及应用

感谢关注！

## 子工程

| 子工程 | 说明 |
|---|---|
| [`frame-me-parent`](./frame-me-parent) | Spring Boot 4.0.7 + Java 21 多模块 Maven 脚手架（starter 体系：base/auth/redis/cache/审计/加密/云组件等） |
| [`fm-demo`](./fm-demo) | 基于 `frame-me-parent` 的演示聚合服务（`fm-demo-api` + `fm-demo-service`） |

## 文档导航

本仓库通过 `docs/` 知识库维护架构、约定、构建说明等文档：

- [`docs/index.md`](./docs/index.md) — 总工程文档导航
- [`docs/projects.md`](./docs/projects.md) — 子工程索引
- [`docs/build.md`](./docs/build.md) — 构建与 CI/CD 约定
- [`docs/architecture.md`](./docs/architecture.md) — 分层与服务边界
- [`docs/conventions.md`](./docs/conventions.md) — 跨工程编码与接口约定
- [`docs/modules.md`](./docs/modules.md) — 顶层目录/服务职责
- [`docs/testing.md`](./docs/testing.md) — 测试策略
- [`docs/reference.md`](./docs/reference.md) — 关键文件路径与扩展点

各子工程的具体信息，请进入对应子目录查看其 `CLAUDE.md` 与 `docs/` 知识库。
