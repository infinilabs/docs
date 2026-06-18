---
title: "Mem0 集成"
date: 0001-01-01
description: "将 Mem0 与 Easysearch 集成，为 AI Agent 提供持久化语义记忆能力。"
summary: "Mem0 集成 #   Mem0 是一个为 AI Agent 提供持久化记忆的开源框架。通过将 Easysearch 作为向量存储后端，Mem0 可以对记忆进行语义检索，让 Agent 在跨会话、跨项目的场景下依然能精准调取历史上下文。
架构概览 #  用户对话 → AI Agent ↓ MCP Server (Mem0 OpenMemory) ↓ ┌─────────────────────┐ │ Easysearch │ │ · 记忆向量索引 │ │ · kNN 语义检索 │ └─────────────────────┘ 整体流程：
 Agent 通过 MCP 协议调用 add_memories 将重要信息写入 Easysearch； Agent 通过 MCP 协议调用 search_memory 对 Easysearch 进行 kNN 语义检索，取回相关记忆； 检索结果注入 Prompt，辅助 Agent 决策。  前置条件 #     条件 说明     Easysearch 集群 已部署并可访问的 Easysearch 集群   Python 3."
---


# Mem0 集成

[Mem0](https://mem0.ai) 是一个为 AI Agent 提供持久化记忆的开源框架。通过将 Easysearch 作为向量存储后端，Mem0 可以对记忆进行语义检索，让 Agent 在跨会话、跨项目的场景下依然能精准调取历史上下文。

## 架构概览

```
用户对话 → AI Agent
               ↓
   MCP Server (Mem0 OpenMemory)
               ↓
     ┌─────────────────────┐
     │  Easysearch         │
     │  ·  记忆向量索引      │
     │  ·  kNN 语义检索     │ 
     └─────────────────────┘
```

整体流程：

1. Agent 通过 MCP 协议调用 `add_memories` 将重要信息写入 Easysearch；
2. Agent 通过 MCP 协议调用 `search_memory` 对 Easysearch 进行 kNN 语义检索，取回相关记忆；
3. 检索结果注入 Prompt，辅助 Agent 决策。

## 前置条件

| 条件 | 说明 |
|------|------|
| Easysearch 集群 | 已部署并可访问的 Easysearch 集群 |
| Python 3.10+ | 运行 Mem0 MCP Server |
| OpenAI 兼容的 LLM | 用于整理写入的记忆内容 |
| OpenAI 兼容的 Embedding 模型 | 用于为记忆生成向量 |

## 部署步骤

### 第一步：启动 Easysearch

> 如果你已有可访问的 Easysearch 集群，可跳过此步骤，直接进入[第二步](/docs/integrations/third-party/mem0/#第二步获取-mem0-源码)。

以下演示使用 Docker 快速拉起一个单节点集群：

> 见[Easysearch 快速开始](/docs/quick-start/install/)如何拉起 Easysearch 集群

```sh
# 生成 admin 初始密码
echo "EASYSEARCH_INITIAL_ADMIN_PASSWORD=$(printf "%s%s%s@%s" "$(tr -dc a-z</dev/urandom|head -c4)" "$(tr -dc A-Z</dev/urandom|head -c3)" "$(tr -dc 0-9</dev/urandom|head -c2)" "$(tr -dc a-z0-9</dev/urandom|head -c3)")" | tee .env

# 启动容器
docker run -d --name easysearch \
   --ulimit memlock=-1:-1 \
   --env-file ./.env -p 9200:9200 \
   -v easysearch-data:/app/easysearch/data \
   -v easysearch-config:/app/easysearch/config \
   -v easysearch-logs:/app/easysearch/logs \
   infinilabs/easysearch:latest
```

验证节点正常启动：

```sh
curl -ku 'admin:<admin用户密码>' https://127.0.0.1:9200
```

收到如下响应即代表启动成功：

```json
{
  "name" : "node_f2f876be-851e-413e-b371-a7c08669e418",
  "cluster_name" : "easysearch_c02ed88b-126b-4035-a650-79677be24ece",
  "version" : {
    "distribution" : "easysearch",
    "number" : "2.1.2",
    "distributor" : "INFINI Labs"
  },
  "tagline" : "You Know, For Easy Search!"
}
```

### 第二步：获取 Mem0 源码

```sh
git clone https://github.com/infinilabs/mem0.git && cd mem0
git checkout feat/easysearch
```

### 第三步：配置 MCP Server

编辑 `openmemory/api/.env` 文件：

```ini
USER=<用户名字>

# ----------------------------------------------------
# 配置语言模型（用于整理写入的记忆内容）
# ----------------------------------------------------
LLM_PROVIDER=openai
LLM_MODEL=<语言模型>
LLM_BASE_URL=<模型提供商的 base URL>

# ----------------------------------------------------
# 配置向量模型（为记忆生成向量，支持语义搜索）
# ----------------------------------------------------
EMBEDDER_PROVIDER=openai
EMBEDDER_MODEL=<向量化模型>
EMBEDDER_BASE_URL=<模型提供商的 base URL>
EASYSEARCH_EMBEDDING_DIMS=<向量维度>

# ----------------------------------------------------
# API Key
# ----------------------------------------------------
OPENAI_API_KEY=<模型提供商的 API key>

# ----------------------------------------------------
# Easysearch 连接信息
# ----------------------------------------------------
EASYSEARCH_URL=<Easysearch 地址>
EASYSEARCH_USER=<用户名>
EASYSEARCH_PASSWORD=<密码>
```

以阿里云百炼平台为例的完整配置：

```ini
USER=steve

LLM_PROVIDER=openai
LLM_MODEL=deepseek-r1
LLM_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1

EMBEDDER_PROVIDER=openai
EMBEDDER_MODEL=text-embedding-v4
EMBEDDER_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
EASYSEARCH_EMBEDDING_DIMS=1024

OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx

EASYSEARCH_URL=https://127.0.0.1:9200
EASYSEARCH_USER=admin
EASYSEARCH_PASSWORD=xxxxxxxxxxxxxxxx
```

### 第四步：启动 MCP Server

```sh
cd openmemory/api

# 创建并激活虚拟环境
python3 -m venv .venv
source .venv/bin/activate

# 安装依赖
pip install -r requirements.txt && pip uninstall -y mem0ai && pip install -e "../../[extras]"

# 启动服务
uvicorn main:app --host 0.0.0.0 --port 8765
```

看到如下输出代表启动成功：

```
INFO:     Started server process [88835]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8765 (Press CTRL+C to quit)
```

## 在 AI Agent 工具中配置

### 配置 MCP Server

在 Agent 工具中配置 URL 为 `http://localhost:8765/mcp/<客户端ID>/http/<用户名字>` 的 MCP server

> `<客户端ID>` 用于区分不同的 Agent 工具，可自定义，例如 `cursor`、`vscode` 等。

### 配置 Memory Skill

在 Agent 工具中添加如下全局技能（Skill），指导 Agent 何时写入、何时检索记忆：

```markdown
---
name: memory
description: "MANDATORY MEMORY PROTOCOL. You MUST call `search_memory` before any action and `add_memories` after any significant discovery. This is the primary persistent storage for project context, decisions, and bug fixes."
---

# Memory Skill

## 🚨 MANDATORY EXECUTION PROTOCOL (SOP) 🚨

### STEP 1: Pre-Task Mandatory Retrieval
**Before taking ANY action** (reading files, executing commands, writing code, or answering):
1. You **MUST** call `search_memory`.
2. Search keywords: Main entities, technology stack, and the specific problem mentioned.
3. **DO NOT** assume you know the project context; always check the memory first.

### STEP 2: Task Execution & Monitoring
During execution, pay attention to "Knowledge Nuggets" — information that isn't in the codebase but is crucial for future work.

### STEP 3: Mandatory Post-Task Consolidation (THE "SAVE" RULE)
**Before sending your final response**, you MUST evaluate if you discovered any **New Project Knowledge (NPK)**. If any of the following occurred, call `add_memories`:

- **Decision Logs:** Did you choose a specific library, pattern, or architecture?
- **Bug Root Causes:** Did you debug a tricky issue? Save the fix and the 'why'.
- **Environment/Build Specs:** Did you find a specific command or environment variable needed?
- **Naming Conventions:** Did you establish or discover naming rules in this project?
- **Business Logic:** Did you uncover a hidden rule in the requirement?

## Available MCP Tools
| Tool | Purpose |
|------|---------|
| `add_memories` | Store new information to memory |
| `search_memory` | Semantic search across all memories |
| `list_memories` | List all stored memories |
| `delete_memories` | Delete specific memories by ID |
```


## MCP 工具说明

| 工具 | 说明 |
|------|------|
| `add_memories` | 将文本内容向量化后写入 Easysearch |
| `search_memory` | 对 Easysearch 执行 kNN 语义检索，返回相关记忆 |
| `list_memories` | 列出所有已存储的记忆条目 |
| `delete_memories` | 按 ID 删除指定记忆 |

## 注意事项

1. Embedding 维度: `EASYSEARCH_EMBEDDING_DIMS` 必须与实际模型输出维度一致
2. Easysearch 默认启用 HTTPS，`.env` 中的 URL 请使用 `https://`
3. TLS 证书验证

   Easysearch 默认使用自签名证书。MCP Server 连接 Easysearch 时，默认会跳过证书验证，适合本地开发环境。

   如需启用证书验证（生产环境推荐），有以下两种方法：

   1. 使用受信任 CA 颁发的证书
   2. 将自签名 CA 证书加入系统信任链

## 延伸阅读

- [Mem0 官方文档](https://docs.mem0.ai/)
- [Easysearch 快速开始](/docs/quick-start/install/)
