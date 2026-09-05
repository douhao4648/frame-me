# frame-me-gateway 部署到 ACK daily 环境

本文档记录 `frame-me-gateway` 通过云效 Flow 流水线部署到阿里云 ACK 集群 daily 环境的完整步骤。换环境/换电脑后按本文档即可恢复全部上下文。

> ⛔ **云效安全约束**：凡会影响线上发布或应用的云效操作（创建/修改/编辑/删除/运行/部署/回滚），AI 助手一律不得直接执行，必须人工审核确认。详见 `docs/deployment-yunxiao.md`。

## 前置条件

**敏感配置统一存放**：`.secrets/gateway-deploy.env`（不入库，真值人工填入）。所有部署步骤执行前先 `source .secrets/gateway-deploy.env` 加载环境变量。模板见 `.secrets/gateway-deploy.env`（占位符，需人工填入真值）。

> 🔐 隐私信息存储规则见根 `CLAUDE.md`——密钥/凭证/云上资源 ID 一律存 `.secrets/*.env`，禁止写入 git 可追踪文件。

| 项 | 要求 | 说明 |
|---|---|---|
| ACK 集群 | 已创建，kubectl 可连通 | 部署目标集群 |
| MSE Nacos | 已开通，拿到地址/AK/SK | 注册中心+配置中心，gateway 强依赖 |
| ACR 镜像仓库 | 已创建仓库 | 或复用现有，按组织约定 |
| 云效组织 | 已加入，PAT 已设置 | `source .secrets/yunxiao-identity.env` |
| GitHub 服务连接 | 已在云效控制台建立 | OAuth 授权 `$GITHUB_REPO` 仓库一次性 |
| JDK 25 | 本地已安装 JDK 25 | 仅本地验证用，路径按本机实际调整 |

## 产物清单

| 产物 | 路径 |
|---|---|
| Dockerfile | `frame-me-parent/frame-me-launcher/frame-me-gateway/Dockerfile` |
| K8s Deployment+Service | `frame-me-parent/frame-me-launcher/frame-me-gateway/deployment.yaml` |
| .dockerignore | `frame-me-parent/frame-me-launcher/frame-me-gateway/.dockerignore` |

> K8s Secret **不入库、不提供 yaml 文件**——真值只存 `.secrets/gateway-deploy.env`，通过环境变量方式创建（见步骤 1）。避免真值被误提交到 git。

## 步骤 1：ACK 确认 namespace + Secret

`deployment.yaml` 统一引用单个 Secret `me-credentials`（含 `NACOS_HOST`/`NACOS_AK`/`NACOS_SK`/`OFFLINE_TOKEN`/`ME_ENCRYPT_PASSWORD` 全部 key）。**Secret 已在 ACK 集群配好**，部署前只需确认存在：

```bash
# 加载部署参数
source .secrets/gateway-deploy.env

# 确认 namespace 存在
kubectl get namespace "app-$ENV"

# 确认 me-credentials Secret 存在且包含全部 key
kubectl get secret me-credentials -n "app-$ENV" -o jsonpath='{.data}' | grep -o '"NACOS_HOST\|NACOS_AK\|NACOS_SK\|OFFLINE_TOKEN\|ME_ENCRYPT_PASSWORD'
# 期望：5 个 key 全部出现
```

> Secret 真值由人工在 ACK 集群配置，不通过 AI 注入、不入库。若后续需重建，参考下方"重建 Secret 命令"。

<details>
<summary>重建 Secret 命令（仅参考，真值不入库）</summary>

```bash
kubectl create secret generic me-credentials \
  --from-literal=NACOS_HOST="<NACOS地址>" \
  --from-literal=NACOS_AK="<AK>" \
  --from-literal=NACOS_SK="<SK>" \
  --from-literal=OFFLINE_TOKEN="<下线token>" \
  --from-literal=ME_ENCRYPT_PASSWORD="<加密密码>" \
  --namespace="app-$ENV" --dry-run=client -o yaml | kubectl apply -f -
```

</details>

流水线部署时需要传值的变量（定义在 `.secrets/gateway-deploy.env`）：

| 环境变量 | 用途 |
|---|---|
| `SLS_PROJECT` | 阿里云 SLS 日志项目（`deployment.yaml` `aliyun_logs_*_project` 占位符引用，流水线 IMAGES 注入） |

## 步骤 2：ACR 创建镜像仓库

在阿里云 ACR 企业版实例下创建仓库。涉及的敏感变量（`.secrets/gateway-deploy.env`）：`ACR_INSTANCE`、`ACR_NAMESPACE`、`ACR_REPO`、`ACR_REGISTRY`。

仓库名用 `$ACR_REPO`（建议独立仓库便于管理，也可复用现有仓库按组织约定）。

创建后在云效流水线参数里引用：
- `DOCKER_INSTANCE` → `$ACR_INSTANCE`
- `DOCKER_NAMESPACE` → `$ACR_NAMESPACE`
- `DOCKER_REPO` → `$ACR_REPO`

## 步骤 3：云效流水线 SOP

参考组织内一条现有 Java 流水线（模板 ID 从 `$YUNXIAO_TEMPLATE_PIPELINE_ID` 加载）创建。复用其 ACR/K8s 服务连接（ID 从 `.secrets` 加载）。

### 只读：拉取模板 YAML

```bash
source .secrets/yunxiao-identity.env
source .secrets/gateway-deploy.env

aliyun devops flow-get-pipeline \
  --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" \
  --pipeline-id "$YUNXIAO_TEMPLATE_PIPELINE_ID"
```

### 需改的参数（对照模板，逐处替换）

| 参数 | 改为（环境变量） |
|---|---|
| 代码源 | GitHub `$GITHUB_REPO` + 服务连接 `$YUNXIAO_GITHUB_SERVICE_CONNECTION_ID` + 分支 `$GITHUB_BRANCH` |
| Maven 构建命令 | `mvn -B clean package -pl frame-me-launcher/frame-me-gateway -am -Dmaven.test.skip=true`（在 `frame-me-parent/` 目录执行） |
| JDK 版本 | jdk25（若云效构建环境无 JDK25，需自定义构建镜像或用 `maven:3.9-eclipse-temurin-25`） |
| ACR 服务连接 ID | `$YUNXIAO_ACR_SERVICE_CONNECTION_ID` |
| ACR 实例 / 命名空间 / 仓库 | `$ACR_INSTANCE` / `$ACR_NAMESPACE` / `$ACR_REPO` |
| K8s 服务连接 ID | `$YUNXIAO_K8S_SERVICE_CONNECTION_ID` |
| deployment.yaml 路径 | `frame-me-parent/frame-me-launcher/frame-me-gateway/deployment.yaml` |
| deployment.yaml 变量 | 云效"部署"阶段 `IMAGES` JSON 传入 `IMAGE` / `PROJECT_NAME` / `ENV` / `REPLICAS_NUM`，替换 yaml 里的 `${VAR}` 占位符 |
| Dockerfile build args | **必填**：云效流水线"构建"阶段的 `ARGS` 传入 `PROJECT_NAME` / `PROJECT_VERSION` / `ENV` / `BUILD_DATE` / `GIT_COMMIT`，注入镜像 LABEL（OCI 标准）和容器 ENV。`ENV` 决定 `--spring.profiles.active`（如 `daily`）。不传则构建失败（Dockerfile 内有必填校验） |
| 构建节点组 | `$YUNXIAO_BUILD_NODE_GROUP` |

> 模板的 ACR/K8s 服务连接、构建节点组等组织级配置原样复用，只需把模板里的具体 ID 换成上面的环境变量值。

### 云效"部署"阶段 IMAGES JSON 配置

`IMAGES` 字段的值（JSON），变量从"设置环境变量"阶段注入：

```json
{
  "IMAGE": "${上游构建步骤.DOCKER_OUTPUT_VPC}",
  "PROJECT_NAME": "${USER_PROJECT_NAME}",
  "ENV": "${USER_ENV}",
  "REPLICAS_NUM": "${USER_REPLICAS_NUM}"
}
```

替换 deployment.yaml 中的对应 `${VAR}` 占位符。

### 创建流水线（人工执行）

```bash
# 0. 加载敏感配置（云效身份 + 服务连接 ID）
source .secrets/yunxiao-identity.env
source .secrets/gateway-deploy.env

# 1. 列出流水线确认模板
aliyun devops flow-list-pipelines --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" --page 1 --per-page 10

# 2. 拉模板完整 YAML（上面已做）
# 3. 在 YAML 里改上述 4 处 + 服务连接 ID 用环境变量值替换，人工审核后创建：
aliyun devops flow-create-pipeline --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" --content <修改后的YAML>

# 4. 验证创建成功
aliyun devops flow-get-pipeline --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" --pipeline-id <新ID>
```

## 步骤 4：触发运行 + 查状态

```bash
# 触发运行（人工执行）
aliyun devops flow-create-pipeline-run \
  --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" \
  --pipeline-id <新ID>

# 查最新运行状态（status 非 FAIL 即成功）
aliyun devops flow-get-latest-pipeline-run \
  --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" \
  --pipeline-id <新ID>
```

## 验证清单

部署成功后验证：

```bash
# 先加载环境变量
source .secrets/gateway-deploy.env

# 1. Pod 状态
kubectl get pods -n "app-$ENV" -l app="$PROJECT_NAME"
# 期望：2 副本 Running

# 2. 日志（确认 Nacos 注册成功）
kubectl logs -n "app-$ENV" deploy/"$PROJECT_NAME" | tail -50

# 3. 健康端点
kubectl exec -n "app-$ENV" deploy/"$PROJECT_NAME" -- curl -s localhost:10031/actuator/health
# 期望：{"status":"UP"}

# 4. 路由测试（需下游服务已注册到同一 MSE Nacos）
kubectl exec -n "app-$ENV" deploy/"$PROJECT_NAME" -- curl -s localhost:10030/actuator/health
```

## 回滚 / 排障

### 回滚到上一版本

```bash
source .secrets/gateway-deploy.env

# 查看部署历史
kubectl rollout history deployment/"$PROJECT_NAME" -n "app-$ENV"

# 回滚到上一版
kubectl rollout undo deployment/"$PROJECT_NAME" -n "app-$ENV"

# 回滚到指定版本
kubectl rollout undo deployment/"$PROJECT_NAME" -n "app-$ENV" --to-revision=<N>
```

### 常见问题

| 问题 | 排查 |
|---|---|
| Pod CrashLoopBackOff | `kubectl logs` 看是否 Nacos 连不上（检查 Secret 值）、JDK 版本不对、profile 配置错误 |
| `AnonymousAccessGuard` 拒绝启动 | daily profile 下 `allow-anonymous` 应为 false（默认）；但若 Nacos 配置中心里 `app-shared.yml` 或 `frame-me-gateway-daily.yml` 误设了 `allow-anonymous: true`，会触发守卫拒绝启动。**deployment.yaml 已显式设 `ALLOW_ANONYMOUS=false` 环境变量覆盖**，环境变量优先级高于 Nacos 配置，确保 daily 环境强制鉴权 |
| Nacos 注册失败 | 检查 `mse-nacos-credentials` Secret 的 `nacos-host`/`nacos-ak`/`nacos-sk` 是否正确，网络是否通 |
| 路由 404 | 确认下游服务已注册到同一 MSE Nacos；`discovery.locator.enabled=true` 自动路由需下游服务名匹配 |
| 优雅下线不生效 | 确认 `OFFLINE_TOKEN` Secret 值与配置一致，`terminationGracePeriodSeconds` ≥ 60s |

## 本地验证（可选，部署前自测）

本地 Nacos 地址（docker-compose）：`localhost:8848`，如需连其他 Nacos，从 `.secrets/gateway-deploy.env` 加载 `MSE_NACOS_HOST`。

```bash
# 0. 如需连非本地 Nacos，加载敏感配置
# source .secrets/gateway-deploy.env

# 1. Maven 构建（JAVA_HOME 指向本机 JDK 25 安装路径）
cd frame-me-parent
export JAVA_HOME=<JDK_25_HOME>
./mvnw -pl frame-me-launcher/frame-me-gateway -am clean package -DskipTests

# 2. Docker 构建镜像
docker build -t frame-me-gateway:daily frame-me-launcher/frame-me-gateway/

# 3. 启动（连本地 docker-compose Nacos；如连 MSE 换成 $MSE_NACOS_HOST）
docker run -d --name gateway-test -p 10030:10030 -p 10031:10031 \
  -e NACOS_HOST=host.docker.internal:8848 \
  -e NACOS_AK=nacos -e NACOS_SK=nacos \
  -e ALLOW_ANONYMOUS=false \
  frame-me-gateway:daily

# 4. 健康检查（管理端口）
curl localhost:10031/actuator/health
# 期望：{"status":"UP"}

# 5. 清理
docker rm -f gateway-test
```
