# Cloud Moltbot (Cloudflare + Moltbot)

**Cloud Moltbot** 是一个结合了 Cloudflare 强大基础设施与 Moltbot 智能能力的容器化 AI 助手解决方案。

这是一个基于 Cloudflare Workers 和 Cloudflare Containers 的 TypeScript 项目。它利用 Cloudflare 的基础设施来运行和管理容器化工作负载。

[English](README.md) | 简体中文

---

## 🛠 技术栈

- **运行时**: Cloudflare Workers + Containers
- **语言**: TypeScript (ES2024)
- **包管理器**: pnpm
- **容器规格**: 1 vCPU, 4GB RAM, 8GB 磁盘
- **核心库**:
  - `cloudflare:workers`: Workers 标准库
  - `@cloudflare/containers`: 容器管理
- **容器基础镜像**: `nikolaik/python-nodejs:python3.12-nodejs22-bookworm`
- **存储**: TigrisFS 挂载 S3/R2

## 🚀 快速开始

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/miantiao-me/cloud-moltbot)

### 前置要求

- Node.js (v22+)
- pnpm (v10.28.2+)
- Wrangler CLI (`npm i -g wrangler`)

### 安装依赖

```bash
pnpm install
```

### 本地开发

启动本地开发服务器：

```bash
pnpm dev
```

### 代码检查

运行格式化工具 (oxfmt) 和代码检查 (oxlint)：

```bash
pnpm lint
```

### 生成类型定义

如果你修改了 `wrangler.jsonc` 中的 bindings，需要重新生成类型文件：

```bash
pnpm cf-typegen
```

## 📦 部署

部署代码到 Cloudflare 全球网络：

```bash
pnpm deploy
```

## 📂 项目结构

```
.
├── src/
│   ├── index.ts        # Workers 入口文件 (ExportedHandler)
│   └── container.ts    # AgentContainer 类定义 (继承自 Container)
├── worker-configuration.d.ts # 自动生成的环境绑定类型
├── wrangler.jsonc      # Wrangler 配置文件
├── tsconfig.json       # TypeScript 配置
└── package.json
```

## 💾 数据持久化 (S3/R2)

容器内置了对 S3 兼容存储（如 Cloudflare R2, AWS S3）的支持，通过 `TigrisFS` 将对象存储挂载为本地文件系统，实现数据的持久化保存。

### 环境变量配置

要启用数据持久化，需要在容器运行环境中配置以下环境变量：

| 变量名                  | 描述                                     | 是否必须 | 默认值   |
| ----------------------- | ---------------------------------------- | -------- | -------- |
| `S3_ENDPOINT`           | S3 API 端点地址                          | ✅ 是    | -        |
| `S3_BUCKET`             | 存储桶名称                               | ✅ 是    | -        |
| `S3_ACCESS_KEY_ID`      | 访问密钥 ID                              | ✅ 是    | -        |
| `S3_SECRET_ACCESS_KEY`  | 访问密钥 Secret                          | ✅ 是    | -        |
| `S3_REGION`             | 存储区域                                 | ❌ 否    | `auto`   |
| `S3_PATH_STYLE`         | 是否使用 Path Style 访问                 | ❌ 否    | `false`  |
| `S3_PREFIX`             | 存储桶内的路径前缀（子目录）             | ❌ 否    | (根目录) |
| `TIGRISFS_ARGS`         | 传递给 TigrisFS 的额外挂载参数           | ❌ 否    | -        |
| `MOLTBOT_GATEWAY_TOKEN` | Gateway 访问令牌（用于 Web UI 连接验证） | ✅ 是    | -        |

### 工作原理

1. **挂载点**: 容器启动时，会将 S3 存储桶挂载到 `/data`。
2. **工作目录**: 实际的工作空间位于 `/data/workspace`。
3. **Moltbot 配置**: Moltbot 的配置文件会存储在 `/data/.moltbot` 中，确保状态持久化。
4. **初始化**:
   - 如果 S3 存储桶（或指定的前缀路径）为空，容器会自动将预置的目录结构初始化。
   - 如果 S3 配置缺失，容器将回退到非持久化的本地目录模式。

### Web UI 初始化

首次启动后，Moltbot 需要通过 Web UI 进行初始化配置。
请访问部署后的 URL（例如 `https://your-worker.workers.dev`），按照屏幕提示完成设置。

## 📝 开发规范

详细的开发规范、代码风格和 AI Agent 行为准则，请参考 [AGENTS.md](./AGENTS.md)。
