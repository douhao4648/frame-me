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
| `docs/deployment-yunxiao.md` | 部署、云效流水线、ACK 相关操作（**含最高优先级安全约束：云效一切写操作禁止直接执行，详见文档**） |

> ⛔ **云效安全约束（不可违背）**：凡会影响线上发布或应用的云效操作（创建、修改、编辑、删除、运行、部署、回滚等），一律不得直接执行，必须人工审核确认 —— 任何模式（含 auto mode）都不例外；只允许只读查询（list/get/search）。一次性授权不扩展到其他操作。细则见 `docs/deployment-yunxiao.md`。

> 🔐 **隐私信息存储规则（不可违背）**：一切密钥、凭证、云上资源标识（AK/SK、密码、token、服务连接 ID、实例地址等）**不得写入任何会被 git 追踪的文件**——包括代码、配置文件、部署 yaml、文档示例。统一存放 `.secrets/*.env`（已 gitignore），使用时 `source` 加载环境变量。云效 PAT 只走环境变量 `ALIBABA_CLOUD_YUNXIAO_ACCESS_TOKEN`，不写入任何文件。交付产物（Dockerfile、K8s yaml、文档）里只允许出现环境变量引用或 `<占位符>`，禁止出现具体值。

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
