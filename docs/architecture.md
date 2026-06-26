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
- 服务发现、鉴权、链路追踪等云组件能力预留在 `frame-me-starter-cloud`（当前为占位），后续接入 Nacos/Gateway/Sentinel 时在此补充。
