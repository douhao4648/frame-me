# 构建与运行

## 环境要求

- JDK 25+
- Maven 3.9+
- （可选）本地已安装 `frame-me-parent` 及其模块到本地 Maven 仓库

## 常用命令

### 编译/安装依赖

```bash
mvn clean compile
```

### 运行测试

```bash
mvn test
```

### 运行单个测试

```bash
mvn -pl fm-demo-service test -Dtest=ClassName#methodName
```

### 启动应用

```bash
mvn -pl fm-demo-service spring-boot:run
```

带 Swagger 文档启动：

```bash
mvn -pl fm-demo-service spring-boot:run -Pswagger
```

带 SQL 打印启动：

```bash
mvn -pl fm-demo-service spring-boot:run -Pp6spy
```

## 新增子模块/组件

在 `fm-demo` 聚合工程下新增子模块的步骤：

1. **创建子模块目录**：在 `fm-demo/` 下新建子目录（如 `fm-demo-xxx`）。
2. **编写子模块 POM**：在 `fm-demo-xxx/pom.xml` 中定义 `<parent>` 为 `com.fm.demo:fm-demo:1.0.0-SNAPSHOT`，并声明所需依赖。
3. **注册到聚合 POM**：在根 `pom.xml` 的 `<modules>` 节点中新增 `<module>fm-demo-xxx</module>`。
4. **补充 dependencyManagement**（如需要版本锁定）：在根 `pom.xml` 的 `<dependencyManagement>` 中增加该模块的依赖声明，供其他模块引用。
5. **添加模块间依赖**：若 `fm-demo-service` 需要引用新模块，在 `fm-demo-service/pom.xml` 的 `<dependencies>` 中增加对应 `<dependency>`。

## Lint / 代码检查

当前项目**未配置** SpotBugs、Checkstyle、PMD 等代码检查插件。如需引入，建议在根 `pom.xml` 的 `<build><plugins>` 中统一配置。
