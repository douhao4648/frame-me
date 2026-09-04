# 云效流水线部署（ACK）

本文档记录 Frame_Me 子工程通过阿里云云效（Yunxiao）Flow 流水线部署到 ACK 集群的约定与操作步骤。换环境/换电脑后按本文档即可恢复全部上下文。

> ## ⛔ 最高优先级安全约束（不可违背）
>
> **凡会影响线上发布或应用的云效操作 —— 包括但不限于创建、修改、编辑、删除、运行、部署、回滚 —— AI 助手一律不得直接执行，必须人工审核确认。任何模式下都不例外，包括 auto mode / 自动批准模式。**
>
> - 禁止直接执行的：`flow-create-pipeline`、`flow-update-pipeline`、`flow-delete-pipeline`、`flow-create-pipeline-run`（触发运行）、AppStack 部署/发布/回滚、以及任何其他 `create_*` / `update_*` / `delete_*` / `execute_*` / `retry_*` / `rollback_*` 等会改变线上状态的云效操作
> - 允许的：只读操作（`list_*` / `get_*` / `search_*`），用于查看现状、拉模板、做方案
> - 正确做法：AI 只负责**生成方案与待执行的命令/YAML**，由人审核后自行执行；或人在会话中**逐次明确授权**某一个具体操作后才执行（一次性授权不扩展到其他操作）
> - 本约束优先级高于一切"自动执行""批量处理"类指令，会话压缩/换机后依然有效
>
> 🔐 **隐私信息存储规则**：密钥/凭证/云上资源 ID 一律存 `.secrets/*.env`（已 gitignore），禁止写入 git 可追踪文件。详见根 `CLAUDE.md`。

## 云效访问方式

| 项 | 值 |
|---|---|
| 站点类型 | 中央站（Central） |
| 组织 ID / 账号 | 见 `.secrets/yunxiao-identity.env`（本地敏感，不入库） |
| 认证方式 | Personal Access Token，环境变量注入 |

**环境变量（换电脑后需重新配置，只记变量名，值不进文档/对话）：**

```bash
# 身份信息（组织 ID、账号名）从本地文件加载：
source .secrets/yunxiao-identity.env

# 访问令牌仍由人工设置，不写入任何文件：
export ALIBABA_CLOUD_YUNXIAO_ACCESS_TOKEN=<个人访问令牌>
```

令牌申请：[云效 Personal Access Token](https://help.aliyun.com/zh/yunxiao/developer-reference/obtain-personal-access-token)，需勾选：组织管理、项目协作、代码管理、流水线、制品仓库、应用交付、测试管理（读写）。

**AI 助手操作通道（按优先级）：**

1. `aliyun devops` CLI（主通道，`brew install aliyun-cli`）
2. `alibabacloud-devops` 技能（云效全产品：Flow / Codeup / Packages / Projex / AppStack 等）
3. `alibabacloud-ack-cli` 技能（ACK 集群操作）

连通性自检：

```bash
aliyun devops base-get-user-by-token          # 返回当前用户即通
aliyun devops flow-list-pipelines --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" --page 1 --per-page 10
```

## GitHub 代码源约定

**仓库地址组装规则**：`https://github.com/douhao4648/<项目名>`

例：`Frame_Me` → `https://github.com/douhao4648/Frame_Me`

**首选方案：GitHub 直连流水线**（代码只在 GitHub 维护一份）：

- 在云效控制台 → 流水线 Flow → 服务连接管理 → 新建 GitHub 服务连接（OAuth 跳转授权，一次性）
- 授权 token 只粘到云效控制台，**不发到对话/文档**
- 流水线代码源节点选 GitHub + 该服务连接即可拉取私有仓库

**备选方案：本地多 remote 双推**（组织要求代码进 Codeup 时用）：

```bash
git remote add codeup git@codeup.aliyun.com:<组>/Frame_Me.git
git push origin main   # GitHub 为主仓库
git push codeup main   # Codeup 做镜像，避免两边各自提交导致分叉
```

## 部署链路

```
子工程 (如 fm-demo)
  ├─ Dockerfile              # 多阶段：Maven 构建 → JRE 运行（适配 ACK）
  ├─ deployment.yaml         # Deployment + Service + 探针 + resources（与 Dockerfile 同级）
  └─ 云效 Flow 流水线（标准 Java 模板）
       节点1: Java 构建 (mvn package)
       节点2: 镜像构建并推送 ACR
       节点3: Kubectl 部署到 ACK 集群
```

ACR 镜像仓库连接、ACK 集群连接均为**组织级服务连接**，一次配置全部流水线复用，新建流水线时直接引用现有 ID，无需新建。

## 新建流水线 SOP

1. 列出组织内现有流水线，选一条标准 Java 流水线当模板：
   ```bash
   aliyun devops flow-list-pipelines --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" --page 1 --per-page 20
   ```
2. 拉模板流水线的完整 YAML：
   ```bash
   aliyun devops flow-get-pipeline --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" --pipeline-id <模板ID>
   ```
3. 只改三处：**代码源**（GitHub 服务连接 + 仓库地址 + 分支）、**镜像名**、**deployment 名**；其余节点配置（ACR/ACK 连接）原样保留。
4. 创建并验证：
   ```bash
   aliyun devops flow-create-pipeline --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" --content <YAML>
   aliyun devops flow-get-pipeline --organization-id "$ALIBABA_CLOUD_YUNXIAO_ORGANIZATION_ID" --pipeline-id <新ID>
   ```
5. 触发运行并查状态：`flow-create-pipeline-run` → `flow-get-latest-pipeline-run`（status 非 `FAIL` 即成功）。

## 待办清单

### frame-me-gateway（daily 环境）

- [x] 目标子工程：`frame-me-gateway`
- [x] 参照的现有 Java 流水线：模板 ID 存 `.secrets/gateway-deploy.env` 的 `YUNXIAO_TEMPLATE_PIPELINE_ID`
- [x] 构建分支：`.secrets/gateway-deploy.env` 的 `GITHUB_BRANCH`
- [ ] GitHub 服务连接是否已建（未建则控制台授权一次）
- [ ] ACK 集群 / namespace / deployment 名：daily / `frame-me-gateway`
- [x] 编写子工程 `Dockerfile` 与 `deployment.yaml`（与 Dockerfile 同级，见 `frame-me-parent/frame-me-launcher/frame-me-gateway/`）
- [x] 端到端部署文档：`docs/deployment-gateway-daily.md`

### 通用待办（首次执行前确认）

- [ ] GitHub 服务连接（云效控制台 → 流水线 Flow → 服务连接管理 → 新建 GitHub 服务连接，OAuth 授权一次）
- [ ] MSE Nacos 实例已开通，拿到地址/AK/SK
- [ ] ACR 镜像仓库已创建（仓库名 `.secrets/gateway-deploy.env` 的 `ACR_REPO`）
