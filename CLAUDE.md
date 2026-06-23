# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# 项目身份

`Frame_Me` 是多服务总工程目录，聚合了后端微服务脚手架、各业务演示/实现服务，以及未来可能加入的前端、Python、Node.js 等二方应用。每个子目录都是一个独立工程，拥有各自的 `CLAUDE.md` 与 `docs/` 知识库。

详细架构、约定、命令和类索引见 `docs/` 知识库。

# 知识库检索

仓库在 `docs/` 目录下维护了一套总工程知识库，`docs/index.md` 是索引入口；各子工程在子目录下维护独立知识库。在回答实现问题、新增模块、处理异常或配置数据源之前，**优先读取相关文档**，不要仅凭已有记忆推断。

| 文档 | 何时读取 |
|---|---|
| `docs/index.md` | 首次接触总工程或需要文档地图时 |
| `docs/projects.md` | 查看子工程清单、进入子工程知识库 |
| `docs/build.md` | 总工程层面的构建、聚合打包、CI/CD 约定 |
| `docs/architecture.md` | 总工程分层、服务边界、跨服务调用约定 |
| `docs/conventions.md` | 跨子工程的编码、接口、命名、数据访问约定 |
| `docs/modules.md` | 总工程内各顶层目录/服务职责 |
| `docs/testing.md` | 总工程测试策略、集成测试 |
| `docs/reference.md` | 关键文件路径、子工程入口、已知扩展点 |

检索顺序建议：
1. 先读 `docs/index.md` 定位主题；
2. 若问题属于某个子工程，按 `docs/projects.md` 进入对应子工程的 `CLAUDE.md` → `docs/index.md`；
3. 跨工程问题回到总工程 `docs/conventions.md` / `docs/architecture.md`。

# 子工程速查

| 子工程 | 路径 | 说明 |
|---|---|---|
| `frame-me-parent` | `./frame-me-parent` | Spring Boot 4.0.7 + Java 25 多模块 Maven 脚手架 |
| `fm-demo` | `./fm-demo` | 基于 `frame-me-parent` 的演示聚合服务 |

新增子工程后，应更新 `docs/projects.md` 并确保该子工程目录下存在独立的 `CLAUDE.md` + `docs/`。
