# docker

本地测试基础设施的 Docker 编排目录。所有服务通过根目录的 `docker-compose.yml` 统一管理。

## 启动 / 停止

```bash
cd docker
docker compose up -d        # 启动全部
docker compose down         # 停止并删除容器（数据随之清空）
docker compose logs -f nacos
```

## 服务清单

| 服务 | 端口 | 说明 |
|---|---|---|
| Nacos | 8080（控制台）、8848（OpenAPI）、9848（gRPC） | 注册中心 + 配置中心，standalone 模式，控制台 http://localhost:8080（已关鉴权，无需登录） |
| MySQL | 3306 | 8.4，root / root，数据挂 `mysql-data` 卷持久化 |
| Redis | 6379 | 8.x，无密码 |

## 应用接入（Spring Cloud Alibaba）

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
```

新增服务直接在 `docker-compose.yml` 里加 service，并在上方清单登记。
