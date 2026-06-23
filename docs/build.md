# 构建与运行

## 总工程构建原则

`Frame_Me` 总工程本身不直接编译运行，各子工程拥有独立的构建生命周期。

## 子工程构建入口

- `frame-me-parent`：见 [frame-me-parent/docs/build.md](../frame-me-parent/docs/build.md)
- `fm-demo`：见 [fm-demo/docs/build.md](../fm-demo/docs/build.md)

## 批量构建

```bash
# 示例：进入各子工程后分别构建（根据实际子工程数量调整）
cd frame-me-parent && mvn clean compile && cd ..
cd fm-demo && mvn clean compile && cd ..
```

## 新增子工程

1. 在总工程目录下新建子目录。
2. 进入子目录并运行 `/me-init <子工程名>` 初始化该子工程的知识库。
3. 更新 `docs/projects.md` 子工程清单。
