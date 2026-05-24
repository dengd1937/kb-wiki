# Deep Agents Code（dcode）

> Source: https://docs.langchain.com/oss/python/deepagents/code
> Collected: 2026-05-24

**Deep Agents Code（`dcode`）** 是基于 Deep Agents SDK 构建的开源终端编码 agent，作为多 provider LLM 命令行助手运行。

## 安装

```bash
curl -LsSf https://langch.in/dcode | bash
```

默认 provider 包括 OpenAI / Anthropic / Google。**可选 extras** 支持 Ollama、Groq、xAI 等。

> ⚠️ Windows **未官方支持**，但有 WSL workaround。

## 配置

### 凭证认证

通过交互界面里的 `/auth` slash command 配置 API 凭证。

### 配置目录

设置存在 `~/.deepagents/`，每个 agent 一个子目录：
- `config.toml`：模型默认值、provider 设置、主题偏好
- `.env`：API key

**项目级配置**：git 仓库内的 `.deepagents/` 目录加载项目特定配置。

## 核心能力

| 能力 | 工具 |
|------|------|
| 文件管理 | `ls`、`read_file`、`write_file`、`edit_file` + 基于 diff 的审批工作流 |
| Shell 执行 | `execute`——做测试、构建、版本控制 |
| 远程 sandbox | LangSmith / Daytona / Modal / Runloop / AgentCore |
| 网络搜索 | Tavily 驱动的 `web_search`（需要独立 API key 配置）|
| HTTP 请求 | `fetch_url`——API 交互、数据拉取 |
| 任务规划 | `task`——通过自定义 `AGENTS.md` 委派 subagent |
| 记忆与上下文 | 跨会话持久化 + 自动摘要 |
| MCP 工具 | 集成 Model Context Protocol server 的外部工具 |
| 用户询问 | `ask_user` |

## 使用模式

### 交互模式（Interactive）

类似聊天的界面，带 slash command 和键盘快捷键：

| 命令 | 用途 |
|------|------|
| `/auth` | 配置 provider 凭证 |
| `/model` | 切换模型（会话中可换）|
| `/remember` | 写记忆 |
| `/tokens` | 看 token 使用 |
| `/threads` | 管理线程 |
| `/trace` | 查看 trace |

用户可以**会话中切模型、管理 agent、调用 skill**。

### 非交互模式（Non-Interactive）

通过 `-n` flag 执行单任务，适合 CI/CD pipeline：

```bash
dcode -n "Add tests for the auth module"
```

> **stdin 自动触发非交互模式**——管道输入会被识别。
>
> ⚠️ **非交互模式下 shell 命令默认禁用**——需要显式 allowlist。

## 关键 CLI flag

| Flag | 作用 |
|------|------|
| `-n` | 非交互模式（单任务）|
| `-y` / `--auto-approve` | 自动批准潜在破坏性操作（默认要求人工审批）|
| `-S` | Shell 命令 allowlist 控制：`recommended` 或 `all` |
| `--max-turns` | 限制 turn 数防止 runaway |
| `--timeout` | 总预算超时 |
| `--model` | 选 LLM |
| `--sandbox` | 启用 sandbox 模式 |
| `--sandbox-id` | 指定 sandbox ID |
| `--sandbox-setup` | sandbox 初始化脚本 |

## 安全 / 控制

**默认要求人工审批**潜在破坏性操作。`Shift+Tab` 或 `-y`/`--auto-approve` 可以切换自动批准。

**Turn count 限制**（`--max-turns`）+ **超时预算**（`--timeout`）防止自动化环境里 agent 失控。

## 可观测性

**LangSmith tracing** 捕获 agent 操作、tool call、决策。配置：

```bash
# ~/.deepagents/.env
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=...
LANGSMITH_PROJECT=my-project
```

通过 `DEEPAGENTS_CODE_LANGSMITH_PROJECT` 隔离独立项目。

## 远程 Sandbox

`dcode` 内置 sandbox 支持：

```bash
dcode --sandbox daytona "Build and test this Rust project"
```

支持的 provider：LangSmith / Daytona / Modal / Runloop / AgentCore。详见 [Sandboxes](sandboxes.md)。
