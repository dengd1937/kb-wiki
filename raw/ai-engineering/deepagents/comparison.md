# DeepAgents vs Claude Agent SDK 对比

> Source: https://docs.langchain.com/oss/python/deepagents/comparison
> Collected: 2026-05-24
> Maintained by: LangChain

## 核心一句话

> **如果你把 agent 嵌入产品，选 DeepAgents；如果你想要自己用的最佳终端体验，Claude Code 仍然赢。**

## 5 维差异

### 1. Execution Model（执行模式）

| | DeepAgents | Claude Agent SDK |
|---|-----------|------------------|
| Agent **在 sandbox 内**运行 | ✓ | ✓ |
| 把 sandbox **作为工具**给外部 agent 用 | ✓ | ✗ |

DeepAgents 同时支持"agent 在 sandbox 内"和"agent 在外、sandbox 作为工具"两种模式，Claude Agent SDK 只支持第一种。

### 2. Provider Flexibility（模型 provider 灵活度）

| | DeepAgents | Claude Agent SDK |
|---|-----------|------------------|
| 支持的模型 | **任意** provider（Anthropic / OpenAI / Google / Baseten / Fireworks / Ollama / 自部署 vLLM 等） | **仅** Claude（Anthropic API / AWS Bedrock / GCP Vertex AI） |

### 3. Deployment Options（部署形态）

| | DeepAgents | Claude Agent SDK |
|---|-----------|------------------|
| 托管部署 | LangSmith Cloud 一键托管 | 无（仅自托管） |
| 自托管 | ✓ | ✓ |
| 托管 ↔ 自托管切换 | **零代码改动** | 不适用 |

### 4. Multi-Tenancy（多租户）

| | DeepAgents | Claude Agent SDK |
|---|-----------|------------------|
| Per-user 隔离 | **内置** | 自己实现 |
| Scoped threads | **内置** | 自己实现 |
| RBAC（基于角色访问控制） | **内置** | 自己实现 |

### 5. Infrastructure / Backends（基础设施 / 后端）

| | DeepAgents | Claude Agent SDK |
|---|-----------|------------------|
| 文件系统后端 | **可插拔**：local / virtual filesystem / 远程 sandbox / 自定义 | 仅 sandbox 本地文件系统 |
| 后端组合 | CompositeBackend 路由 | 不支持 |
| 自定义后端 | 实现 `BackendProtocol` | 不支持 |

## 选型建议

### 选 **DeepAgents** 如果你：

- 需要**模型和基础设施的灵活度**
- 需要**内置的多租户部署**
- 倾向于"托管或自托管均可、无需改代码"的部署模式
- 想要 SaaS 产品形态（per-user 隔离 + RBAC 是刚需）
- 想要**完全开源**的 harness（MIT 协议，包括内核）
- 想要**自定义/扩展 harness 源码**

### 选 **Claude Agent SDK** 如果你：

- 已经**深度投入 Anthropic 生态**（已用 Claude 模型、已接 Anthropic API）
- 偏好**自托管 + 自己搭建 API / 认证 / 流式层**
- 想要 Anthropic 内部**踩过所有坑的最强打磨度**
- 主要是**单人/小团队的终端体验**（个人使用 Claude Code）

## 一个被低估的差异

**开源彻底度**：

- **DeepAgents**：全开源（MIT），包括内核
- **Claude Agent SDK**：SDK 包装层开源（MIT），但**依赖闭源的 Claude Code binary 内核**——你能 fork SDK 但不能 fork 整个 harness

这是"我能多大程度地理解/改造这个 harness"的天花板差异。

## 一个常见误解

> "DeepAgents 既然模型无关，那应该在所有模型上效果差不多。"

**不**。LangChain 自己的实验表明：同一套 harness 在 **GPT-5.2-Codex 上得 66.5%**，搬到 **Claude Opus 4.6 上只有 59.6%**——因为**没有针对 Claude 重做调优循环**。

所以"模型无关"= **支持任意模型** ≠ **在任意模型上效果相同**。每个模型需要重调一次。详见 [LangChain 02-17 那篇 Terminal Bench 实验](../2026-02-17-deep-agents-harness-engineering.md)。
