# DeepAgents Quickstart

> Source: https://docs.langchain.com/oss/python/deepagents/quickstart
> Collected: 2026-05-24

几分钟内构建你的第一个 deep agent。本指南演示创建一个带规划、文件系统工具、子 agent 能力的**研究 agent**。

## 前置要求

你需要一个模型 provider 的 API key（Gemini、Anthropic、OpenAI 等）。Deep Agents 需要**支持 tool calling** 的模型。

## Step 1：安装依赖

**pip：**

```bash
pip install deepagents tavily-python
```

**uv：**

```bash
uv init
uv add deepagents tavily-python
uv sync
```

> 这里用 Tavily 做示例搜索工具，你也可以换成 DuckDuckGo、SerpAPI、Brave Search。

## Step 2：设置 API keys

需要的环境变量：
- 你选择的 provider 的 API key（任选其一）：
  - `GOOGLE_API_KEY`
  - `OPENAI_API_KEY`
  - `ANTHROPIC_API_KEY`
  - `OPENROUTER_API_KEY`
  - `FIREWORKS_API_KEY`
  - `BASETEN_API_KEY`
  - `OLLAMA_API_KEY`
- `TAVILY_API_KEY`

## Step 3：创建一个搜索工具

```python
import os
from typing import Literal
from tavily import TavilyClient
from deepagents import create_deep_agent

tavily_client = TavilyClient(api_key=os.environ["TAVILY_API_KEY"])

def internet_search(
    query: str,
    max_results: int = 5,
    topic: Literal["general", "news", "finance"] = "general",
    include_raw_content: bool = False,
):
    """Run a web search"""
    return tavily_client.search(
        query,
        max_results=max_results,
        include_raw_content=include_raw_content,
        topic=topic,
    )
```

## Step 4：创建一个 deep agent

`model` 参数接受 `provider:model` 字符串，也接受已初始化的模型实例。

```python
research_instructions = """You are an expert researcher. Your job is to conduct 
thorough research and then write a polished report.

You have access to an internet search tool as your primary means of 
gathering information.

## `internet_search`

Use this to run an internet search for a given query. You can specify 
the max number of results to return, the topic, and whether raw content 
should be included.
"""

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    tools=[internet_search],
    system_prompt=research_instructions,
)
```

> OpenAI / Anthropic / OpenRouter / Fireworks / Baseten / Ollama 同结构，只换 `model=` 字符串。

## Step 5：跑起来

```python
result = agent.invoke({"messages": [{"role": "user", "content": "What is langgraph?"}]})
print(result["messages"][-1].content)
```

## How It Works（运行原理）

你的 deep agent 会自动：

1. **规划**——使用内置的 `write_todos` 工具
2. **做研究**——通过 `internet_search` 工具
3. **管理上下文**——用 `write_file` 和 `read_file` 存大结果
4. **派生 subagent**——处理复杂子任务
5. **综合报告**——整合所有发现

## 示例集合

完整示例见：<https://github.com/langchain-ai/deepagents/tree/main/examples>

## 流式输出

Deep Agents 内置流式支持（通过 LangGraph），实时观察 tool calls、结果、LLM 响应。

## 下一步

- **[Customization](customization.md)**——自定义 prompt、tools、subagent
- 通过 [Memory](customization.md#memory) 添加跨对话的持久化记忆
- 用 **LangSmith 的 Managed Deep Agents** 部署到生产环境
