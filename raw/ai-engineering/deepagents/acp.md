# Agent Client Protocol（ACP）

> Source: https://docs.langchain.com/oss/python/deepagents/acp
> Collected: 2026-05-24

通过 **Agent Client Protocol (ACP)** 暴露 Deep Agents，集成到代码编辑器和 IDE。

## 概览

ACP **标准化了编码 agent 与代码编辑器/IDE 之间的通信**。通过 ACP，自定义 deep agent 可以和任意兼容客户端协作——编辑器提供项目 context，agent 返回详细更新。

> ⚠️ **ACP 用于 agent ↔ 编辑器**集成。如果你想让 agent 调用**外部服务器**托管的工具，看 [Model Context Protocol (MCP)](https://docs.langchain.com/oss/python/integrations/mcp)。

## Quickstart

### 安装

```bash
pip install deepagents-acp
```

```bash
uv add deepagents-acp
```

### 服务器实现

```python
import asyncio

from acp import run_agent
from deepagents import create_deep_agent
from langgraph.checkpoint.memory import MemorySaver

from deepagents_acp.server import AgentServerACP


async def main() -> None:
    agent = create_deep_agent(
        model="google_genai:gemini-3.5-flash",
        # 在这里定制 deep agent：设自定义 prompt、加自己的工具、
        # 挂 middleware、组合 subagent。
        system_prompt="You are a helpful coding assistant",
        checkpointer=MemorySaver(),
    )

    server = AgentServerACP(agent)
    await run_agent(server)


if __name__ == "__main__":
    asyncio.run(main())
```

> **示例编码 agent**：`deepagents-acp` 包含一个**带文件系统和 shell 的开箱即用编码 agent**，见 <https://github.com/langchain-ai/deepagents/blob/main/libs/acp/examples/demo_agent.py>。

## 兼容客户端

Deep Agents 部署在你运行 ACP server 的任何地方。主要兼容客户端：

| 编辑器 | 链接 |
|--------|------|
| **Zed** | <https://zed.dev/docs/ai/external-agents> |
| **JetBrains IDEs** | <https://www.jetbrains.com/help/ai-assistant/acp.html> |
| **VS Code（通过 vscode-acp）** | <https://github.com/formulahendry/vscode-acp> |
| **Neovim（通过兼容插件）** | — |

## Zed 配置

### Step 1：仓库设置

```bash
git clone https://github.com/langchain-ai/deepagents.git
cd deepagents/libs/acp
uv sync --all-groups
chmod +x run_demo_agent.sh
```

### Step 2：凭证配置

```bash
cp .env.example .env
```

在 `.env` 里设 `ANTHROPIC_API_KEY`。

### Step 3：Zed `settings.json` 配置

```json
{
  "agent_servers": {
    "DeepAgents": {
      "type": "custom",
      "command": "/your/absolute/path/to/deepagents/libs/acp/run_demo_agent.sh"
    }
  }
}
```

### Step 4：使用

打开 Zed 的 **Agents 面板**，开始 Deep Agents thread。

## 本地开发：Toad

如果想把 ACP agent server 作为**本地开发工具**运行，可以用 **Toad** 管理进程：

```bash
uv tool install -U batrachian-toad

toad acp "python path/to/your_server.py" .
# 或
toad acp "uv run python path/to/your_server.py" .
```

## 参考资料

- **ACP 介绍**：<https://agentclientprotocol.com/get-started/introduction>
- **客户端/编辑器列表**：<https://agentclientprotocol.com/get-started/clients>
