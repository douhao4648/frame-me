# 测试与运行

## 测试现状

当前 fm-demo 项目中**无测试类**（`fm-demo-service/src/test` 目录下没有 `.java` 测试文件）。

`fm-demo-service/pom.xml` 中已声明测试依赖，但尚未编写实际测试：
- `spring-boot-starter-test`（含 JUnit 5、Mockito、AssertJ）
- `testcontainers-mysql`
- `testcontainers-junit-jupiter`

建议后续补充以下测试：
- **单元测试**：`DemoServiceImpl` 各方法（使用 Mockito 模拟 `DemoMapper`）。
- **集成测试**：`DemoController` 端到端测试（使用 `@SpringBootTest` + `MockMvc`）。
- **数据库测试**：若需真实 MySQL 环境，使用 Testcontainers 启动 MySQL 容器。

## 运行测试

```bash
mvn test
```

由于当前无测试类，执行后会显示 `Tests run: 0`。

运行单个测试（当有测试类后）：

```bash
mvn -pl fm-demo-service test -Dtest=ClassName#methodName
```

## 启动应用

```bash
mvn -pl fm-demo-service spring-boot:run
```

应用默认端口：8080，管理端口：8081。

Swagger UI：`http://localhost:8080/swagger-ui.html`

## 测试约定

- 测试类命名：`XxxTest`（如 `DemoServiceImplTest`）。
- 测试方法命名：`shouldXxxWhenYyy` 或 `testXxx`。
- 数据准备：H2 内存数据库启动时自动执行 `schema.sql` 建表，测试可直接操作数据库。
- 若使用 Testcontainers：在测试类上标注 `@Testcontainers` 和 `@Container` 启动 MySQL 实例。
