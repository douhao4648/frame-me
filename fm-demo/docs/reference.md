# 关键文件索引

## 项目配置

| 文件 | 路径 |
|---|---|
| 聚合 POM | `pom.xml` |
| API 模块 POM | `fm-demo-api/pom.xml` |
| Service 模块 POM | `fm-demo-service/pom.xml` |
| Service 应用配置 | `fm-demo-service/src/main/resources/application.yml` |
| 数据库初始化脚本 | `fm-demo-service/src/main/resources/schema.sql` |

## 核心类/文件

| 类/文件 | 路径 | 作用 |
|---|---|---|
| `IDemoApi` | `fm-demo-api/src/main/java/com/fm/demo/api/IDemoApi.java` | 本地演示 CRUD 接口契约 |
| `ITesterDemoApi` | `fm-demo-api/src/main/java/com/fm/demo/api/ITesterDemoApi.java` | 跨服务透传接口契约 |
| `DemoDTO` | `fm-demo-api/src/main/java/com/fm/demo/api/dto/DemoDTO.java` | 演示数据请求 DTO |
| `DemoQuery` | `fm-demo-api/src/main/java/com/fm/demo/api/query/DemoQuery.java` | 演示数据分页查询参数 |
| `DemoVO` | `fm-demo-api/src/main/java/com/fm/demo/api/vo/DemoVO.java` | 演示数据返回 VO |
| `Application` | `fm-demo-service/src/main/java/com/fm/demo/Application.java` | Spring Boot 启动类 |
| `DemoController` | `fm-demo-service/src/main/java/com/fm/demo/controller/DemoController.java` | 本地接口实现 |
| `TesterDemoController` | `fm-demo-service/src/main/java/com/fm/demo/controller/TesterDemoController.java` | 跨服务透传接口实现 |
| `DemoEntity` | `fm-demo-service/src/main/java/com/fm/demo/entity/DemoEntity.java` | 演示实体（表 demo_user） |
| `DemoMapper` | `fm-demo-service/src/main/java/com/fm/demo/mapper/DemoMapper.java` | 数据访问 Mapper |
| `IDemoService` | `fm-demo-service/src/main/java/com/fm/demo/service/IDemoService.java` | 业务接口 |
| `DemoServiceImpl` | `fm-demo-service/src/main/java/com/fm/demo/service/impl/DemoServiceImpl.java` | 业务实现 |
| `DemoConvert` | `fm-demo-service/src/main/java/com/fm/demo/service/convert/DemoConvert.java` | MapStruct 转换器 |
| `TesterDemoClient` | `fm-demo-service/src/main/java/com/fm/demo/infrastructure/client/tester/TesterDemoClient.java` | 声明式 HTTP 客户端 |
| `DemoConfiguration` | `fm-demo-service/src/main/java/com/fm/demo/infrastructure/config/DemoConfiguration.java` | HTTP 客户端装配配置 |

## 已知扩展点

- **新增业务模块**：参照 `DemoController → IDemoService → DemoServiceImpl → DemoMapper → DemoEntity` 分层创建新包与类。
- **新增跨服务客户端**：在 `infrastructure/client/` 下新建接口继承目标 API，在 `DemoConfiguration` 的 `@ImportHttpServices` 中增加 `basePackages` 路径。
- **自定义 starter 接入**：在 `fm-demo-service/pom.xml` 引入目标 starter，配置项写入 `application.yml`。
- **动态数据源扩展**：`frame-me-booter` 已集成 `frame-me-starter-dynamic-ds`，在 `application.yml` 中按动态数据源格式配置即可。
- **SQL 打印**：通过 `p6spy` Maven profile 激活（`mvn -Pp6spy spring-boot:run`），由 `decorator.datasource.p6spy` 配置控制。
- **Swagger 分组**：在 `application.yml` 的 `frame.me.swagger.groups` 中新增分组定义。
