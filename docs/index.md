# 总工程文档导航

本文档用于帮助 AI 助手（以及开发者）快速理解 `Frame_Me` 总工程的结构、约定与关键实现。

## 项目速览

`Frame_Me` 是多个微服务/应用工程的总目录，采用「总工程 + 子工程」组织方式：

- 每个子工程在独立子目录中维护，拥有独立的构建、测试、部署生命周期。
- 总工程通过 `docs/projects.md` 统一索引所有子工程，并提供跨工程的约定与架构。
- 当前已集成的子工程：
  - `frame-me-parent`：Spring Boot 4.0.7 + Java 25 的多模块 Maven 脚手架。
  - `fm-demo`：基于 `frame-me-parent` 的演示聚合服务（`fm-demo-api` + `fm-demo-service`）。

## 阅读前置

- 总工程本身不直接编译运行，各子工程独立构建。
- 进入具体业务问题前，先确认问题属于哪个子工程，再阅读该子工程的 `CLAUDE.md` 与 `docs/index.md`。
- 跨子工程的规范（如统一异常、接口响应、数据访问）以总工程 `docs/conventions.md` 为准；若子工程另有约定，优先子工程。

## 文档地图

| 文档 | 回答的问题 |
|---|---|
| [projects.md](./projects.md) | 有哪些子工程？如何进入某个子工程的知识库？ |
| [build.md](./build.md) | 总工程层面的构建、聚合打包、CI/CD 约定是什么？ |
| [architecture.md](./architecture.md) | 总工程如何分层？服务边界与跨服务调用如何约定？ |
| [conventions.md](./conventions.md) | 跨子工程的响应/异常/状态码、编码风格、数据访问约定是什么？ |
| [modules.md](./modules.md) | 总工程内各顶层目录/服务的职责、依赖、关键类是什么？ |
| [testing.md](./testing.md) | 总工程层面的测试策略、集成测试如何运行？ |
| [reference.md](./reference.md) | 关键文件路径、子工程入口、已知扩展点在哪里？ |

## 关键约定一句话

总工程只负责索引与跨工程约定，不替代子工程独立知识库；处理具体问题前，先定位到正确的子工程。
