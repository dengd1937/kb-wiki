# DeepAgents 概览（Deep Agents Overview）

> Source: https://docs.langchain.com/oss/python/deepagents/overview
> Collected: 2026-05-24
> Maintained by: LangChain
> License: MIT

## 一句话定位

构建能够**规划、使用子 agent、并通过文件系统处理复杂任务**的 agent。

DeepAgents 是构建 agent 和 LLM 应用最简单的方式——**内置**任务规划、上下文管理用的文件系统、子 agent 派生、长期记忆。你可以用 deep agent 处理任何任务，包括复杂的多步骤任务。

> 我们把 `deepagents` 视为一个 **agent harness**——核心工具调用循环和其他 agent 框架是一样的，但**内置了一套工具和能力**。

LangSmith Engine 能检测 Deep Agents trace 里的问题并提出修复方案，你可以直接从 Engine tab 提交修复 PR。

## 仓库构成

`deepagents` 是一个独立库，构建在 LangChain agent 核心积木之上，使用 LangGraph runtime 提供**持久化执行、流式输出、人机协同**等能力。

仓库包含三个组件：

- **Deep Agents SDK**：构建任意任务 agent 的核心包
- **Deep Agents Code**：基于 SDK 构建的终端编码 agent
- **ACP 集成**：Agent Client Protocol 连接器，用于在 Zed 等代码编辑器中使用 deep agent

> LangChain 提供 agent 的核心积木（framework）；DeepAgents 是它之上的 harness。
> 对三层关系详见 [Frameworks, runtimes, and harnesses](../2025-10-25-agent-frameworks-runtimes-harnesses.md)；与 Anthropic harness 的对比详见 [comparison.md](comparison.md)。

## 快速上手：创建一个 deep agent

### Google

```python
# pip install -qU deepagents langchain-google-genai
from deepagents import create_deep_agent

def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)

agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather in sf"}]}
)
```

### OpenAI

```python
# pip install -qU deepagents langchain-openai
from deepagents import create_deep_agent

def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

agent = create_deep_agent(
    model="openai:gpt-5.4",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)

agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather in sf"}]}
)
```

### Anthropic

```python
# pip install -qU deepagents langchain-anthropic
from deepagents import create_deep_agent

def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

agent = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)

agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather in sf"}]}
)
```

### OpenRouter

```python
# pip install -qU deepagents langchain-openrouter
from deepagents import create_deep_agent

agent = create_deep_agent(
    model="openrouter:anthropic/claude-sonnet-4-6",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)
```

### Fireworks

```python
# pip install -qU deepagents langchain-fireworks
agent = create_deep_agent(
    model="fireworks:accounts/fireworks/models/qwen3p5-397b-a17b",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)
```

### Baseten

```python
# pip install -qU deepagents langchain-baseten
agent = create_deep_agent(
    model="baseten:zai-org/GLM-5",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)
```

### Ollama

```python
# pip install -qU deepagents langchain-ollama
agent = create_deep_agent(
    model="ollama:devstral-2",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)
```

详见 [Quickstart](quickstart.md) 和 [Customization](customization.md)。

通过 LangSmith 追踪请求、调试 agent 行为、评估输出（按 tracing quickstart 设置）。生产环境可以部署到 **LangSmith Cloud** 托管。

## 何时使用 Deep Agents

当你需要构建能做以下事情的 agent 时，使用 **Deep Agents SDK**：

- **处理复杂的多步骤任务**——需要规划与分解
- **管理大量上下文**——通过文件系统工具和摘要
- **切换文件系统后端**——in-memory state、本地磁盘、持久化 store、sandbox 或自定义后端
- **执行 shell 命令**——sandbox 后端下的 `execute` 工具
- **运行 interpreter 代码**——用 interpreters 做工具组合、子 agent 编排、结构化数据变换
- **委派工作**——给特化 subagent 做 context 隔离
- **持久化记忆**——跨对话和线程
- **控制文件系统访问**——声明式权限规则限制可读/可写文件
- **要求人工审批**——敏感操作走 human-in-the-loop
- **使用任意模型**——provider 无关，前沿模型和开源模型都行

> 构建更简单的 agent，请考虑用 LangChain 的 `create_agent` 或自定义 LangGraph 工作流。

## 核心能力（11 项）

### 1. Planning and task decomposition（规划与任务分解）

Deep Agents 内置 `write_todos` 工具，让 agent 把复杂任务拆成离散步骤，跟踪进度，并随新信息浮现而调整计划。

### 2. Context management（上下文管理）

文件系统工具（`ls`、`read_file`、`write_file`、`edit_file`）允许 agent **把大上下文 offload 到内存或文件系统**，防止 context window 溢出，使变长工具结果也能工作。**自动摘要**会在 context window 变长时压缩旧消息，保持 agent 在长会话中持续有效。

### 3. Shell execution（Shell 执行）

使用 sandbox 后端时，agent 获得 `execute` 工具，可以运行 shell 命令做测试、构建、git 操作、系统任务。Sandbox 后端提供隔离，agent 执行代码不会危及宿主系统。

### 4. Interpreters（解释器）

加上一个 interpreter，agent 就能在**内存 JavaScript runtime**里跑代码。Interpreters 让 agent 以编程方式组合工具、编排 subagent、变换结构化数据——**不需要完整 shell 环境**。

### 5. Pluggable filesystem backends（可插拔文件系统后端）

虚拟文件系统由可插拔后端驱动，可以按需切换：
- in-memory state
- 本地磁盘
- LangGraph store（跨线程持久化）
- sandbox（隔离代码执行，支持 Modal、Daytona、Deno）
- 用 composite routing 组合多个后端
- 自己实现自定义后端

### 6. Subagent spawning（子 agent 派生）

内置 `task` 工具让 agent 派生特化 subagent 做 **context 隔离**——保持主 agent 的 context 干净，同时在特定子任务上深挖。

### 7. Long-term memory（长期记忆）

通过 LangGraph 的 Memory Store 扩展 agent 的**跨线程持久化记忆**。Agent 可以保存并检索之前对话的信息。

### 8. Filesystem permissions（文件系统权限）

声明式权限规则控制 agent 可读/可写哪些文件和目录。Subagent 可以**继承或覆盖**父 agent 的规则。

### 9. Human-in-the-loop（人机协同）

通过 LangGraph 的 interrupt 能力，为敏感工具操作配置人工审批。控制哪些工具需要确认才能执行。

### 10. Skills（技能）

用可复用的 **skills** 扩展 agent，提供特化工作流、领域知识、自定义指令。

### 11. Smart defaults（智能默认值）

附带 opinionated 的 system prompt，教模型如何有效使用工具——**行动前规划、验证工作、管理上下文**。可按需自定义或替换默认值。

## 开始

- **[Quickstart](quickstart.md)** - 构建你的第一个 deep agent
- **[Customization](customization.md)** - 定制选项详解
- **[Models](models.md)** - 配置模型和 provider
- **[Backends](backends.md)** - 选择和配置可插拔文件系统后端
- **[Sandboxes](sandboxes.md)** - 在隔离环境中执行代码
- **[Interpreters](interpreters.md)** - 在 QuickJS 中组合工具和变换数据
- **[Permissions](permissions.md)** - 用权限规则控制文件系统访问
- **[Human-in-the-loop](human-in-the-loop.md)** - 为敏感操作配置审批
- **[Code](code.md)** - 使用 Deep Agents Code
- **[ACP](acp.md)** - 通过 ACP 在代码编辑器中使用 deep agent
- **[Reference](api-reference.md)** - `deepagents` API 参考
