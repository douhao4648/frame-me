# 测试与运行

## 测试策略

- 各子工程独立负责单元测试与集成测试。
- 总工程层面可补充跨服务集成测试或端到端测试（如有）。

## 子工程测试入口

- `frame-me-parent`：见 [frame-me-parent/docs/testing.md](../frame-me-parent/docs/testing.md)
- `fm-demo`：见 [fm-demo/docs/testing.md](../fm-demo/docs/testing.md)

## 批量测试

```bash
# 示例：批量运行各子工程测试（根据实际子工程数量调整）
cd frame-me-parent && mvn test && cd ..
cd fm-demo && mvn test && cd ..
```
