# 编码约定

## 响应规范

统一响应体由 `frame-me-parent` 定义，fm-demo 直接复用：

- `Result<T>` / `IResult<T>`：成功/失败的统一响应包装。
- `PageData<T>`：分页查询的统一响应结构（含 records、total、page、size）。

具体字段定义与使用方式见 `../../frame-me-parent/docs/conventions.md`。

## 异常体系

异常体系由 `frame-me-parent` 定义，fm-demo 直接复用：

- `BusinessException`：业务异常基类，支持 `ResultCode` + 占位符消息（如 `new BusinessException(ResultCode.NOT_FOUND, "数据 {} 不存在", id)`）。
- `GlobalExceptionHandler`：统一捕获并转换为 `Result` 响应。

具体继承关系与使用场景见 `../../frame-me-parent/docs/conventions.md`。

## 状态码

HTTP 状态码与业务状态码由 `ResultCode` 枚举定义（父工程），常用值：

| 状态码 | 含义 |
|---|---|
| 200 | 成功 |
| 400 | 参数校验失败 / 请求错误 |
| 404 | 资源不存在 |
| 500 | 系统内部错误 |

完整状态码列表见 `../../frame-me-parent/docs/conventions.md`。

## 编码风格

### 命名规范

- 接口名以 `I` 开头（如 `IDemoApi`、`IDemoService`）。
- 实现类以 `Impl` 结尾（如 `DemoServiceImpl`）。
- Controller 不额外加后缀，直接命名（如 `DemoController`）。
- 转换器以 `Convert` 结尾（如 `DemoConvert`）。
- 客户端以 `Client` 结尾（如 `TesterDemoClient`）。

### Lombok 使用

- 实体/DTO/VO 使用 `@Data` 生成 getter/setter/toString/equals/hashCode。
- 继承父类的实体使用 `@EqualsAndHashCode(callSuper = true)`。
- Service/Controller 使用 `@RequiredArgsConstructor` 进行构造器注入。

### MapStruct 使用

- 转换器接口标注 `@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)`。
- 由 Spring 自动装配为 Bean，Service 层通过构造器注入使用。
- 示例：`DemoConvert` 提供 `toVo(DemoEntity)`、`toVoList(List<DemoEntity>)`、`toEntity(DemoDTO)` 三个方法。

## 数据访问约定

### MyBatis-Plus 约定

- Mapper 继承 `BaseMapper<T>`，不编写 XML（当前 `DemoMapper` 无自定义 SQL）。
- 实体继承 `BaseVersionEntity`（含 `id`、`createTime`、`updateTime`、`deleted`、`version`）。
- 表名通过 `@TableName("demo_user")` 显式指定。

### 逻辑删除

- 字段名：`deleted`
- 删除值：`1`
- 未删除值：`0`
- 配置位置：`application.yml` → `mybatis-plus.global-config.db-config.logic-delete-*`

### 主键策略

- 策略：`assign_id`（雪花 ID）
- 配置位置：`application.yml` → `mybatis-plus.global-config.db-config.id-type: assign_id`
- 代码中通过 `DemoMapper.insert(entity)` 后 `entity.getId()` 自动获取生成的主键。

### 乐观锁

- 字段名：`version`
- 实体继承 `BaseVersionEntity` 已包含该字段。
- `DemoServiceImpl.update` 中显式校验 `dto.getVersion() != null`，更新时 MyBatis-Plus 自动做乐观锁版本比对。

## 接口/文档约定

### REST 路径前缀

- 本地接口：`/api/demo`
- 跨服务透传接口：`/api/tester-demo`

### SpringDoc / Swagger

- 接口契约使用 `@Tag`、`@Operation`、`@Parameter`、`@Schema` 注解描述。
- 分组配置在 `application.yml` 中：`frame.me.swagger.groups` 定义 `demo-api` 分组，匹配 `/api/**`。
- 激活 Swagger 依赖：`mvn -Pswagger spring-boot:run`（`swagger` Maven profile 引入 `frame-me-starter-doc-openapi`）。
- 文档地址：启动后访问 `/swagger-ui.html` 或 `/v3/api-docs`。
