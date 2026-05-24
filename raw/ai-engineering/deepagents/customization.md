# 定制 Deep Agents（Customize Deep Agents）

> Source: https://docs.langchain.com/oss/python/deepagents/customization
> Collected: 2026-05-24

学习如何用 system prompt、tools、subagents 等定制 Deep Agents。

`create_deep_agent` 的核心配置选项：

- [Model](#model)
- [Tools](#tools)
- [System Prompt](#system-prompt)
- [Middleware](#middleware)
- [Interpreters](#interpreters)
- [Subagents](#subagents)
- [Backends（虚拟文件系统）](#backends)
- [Human-in-the-loop](#human-in-the-loop)
- [Skills](#skills)
- [Memory](#memory)
- [Profiles](#profiles)

## 完整签名

```python
create_deep_agent(
    model: str | BaseChatModel | None = None,
    tools: Sequence[BaseTool | Callable | dict[str, Any]] | None = None,
    *,
    system_prompt: str | SystemMessage | None = None,
    middleware: Sequence[AgentMiddleware] = (),
    subagents: Sequence[SubAgent | CompiledSubAgent | AsyncSubAgent] | None = None,
    skills: list[str] | None = None,
    memory: list[str] | None = None,
    permissions: list[FilesystemPermission] | None = None,
    backend: BackendProtocol | BackendFactory | None = None,
    interrupt_on: dict[str, bool | InterruptOnConfig] | None = None,
    response_format: ResponseFormat[ResponseT] | type[ResponseT] | dict[str, Any] | None = None,
    context_schema: type[ContextT] | None = None,
    checkpointer: Checkpointer | None = None,
    store: BaseStore | None = None,
    debug: bool = False,
    name: str | None = None,
    cache: BaseCache | None = None
) -> CompiledStateGraph
```

完整参数列表见 [`create_deep_agent` API 参考](api-reference.md)。

---

## Model

传 `provider:model` 格式字符串，或已初始化的模型实例。详见 [Models](models.md)。

`provider:model` 格式（如 `openai:gpt-5.4`）方便快速切换模型。

### OpenAI

```shell
pip install -U "langchain[openai]"
```

**默认参数：**

```python
import os
from deepagents import create_deep_agent

os.environ["OPENAI_API_KEY"] = "sk-..."

agent = create_deep_agent(model="openai:gpt-5.4")
# 这会以默认参数调用 init_chat_model
# 要用特定参数，直接用 init_chat_model
```

**使用 init_chat_model：**

```python
from langchain.chat_models import init_chat_model
from deepagents import create_deep_agent

model = init_chat_model(model="openai:gpt-5.4")
agent = create_deep_agent(model=model)
```

**使用模型类：**

```python
from langchain_openai import ChatOpenAI
from deepagents import create_deep_agent

model = ChatOpenAI(model="gpt-5.4")
agent = create_deep_agent(model=model)
```

### Anthropic

```shell
pip install -U "langchain[anthropic]"
```

```python
agent = create_deep_agent(model="anthropic:claude-sonnet-4-6")
```

```python
from langchain_anthropic import ChatAnthropic
model = ChatAnthropic(model="claude-sonnet-4-6")
agent = create_deep_agent(model=model)
```

### Azure / Google Gemini / AWS Bedrock / HuggingFace 等

完整 provider 列表见 [Models](models.md)。模式相同：`provider:model-name` 字符串、`init_chat_model()` 或具体的 `Chat<Provider>` 类。

> 聊天模型自动重试瞬时 API 失败（指数退避）。`max_retries` / `timeout` 默认值和调参见 LangChain [Models](https://docs.langchain.com/oss/python/langchain/models#connection-resilience) 页。

---

## Tools

除了 [内置工具](overview.md#核心能力11-项)（规划、文件管理、subagent 派生），你可以提供自定义工具：

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


agent = create_deep_agent(
    model="openai:gpt-5.4",
    tools=[internet_search],
)
```

---

## System Prompt

Deep Agents 自带内置 system prompt。Deep agent 的价值来自 SDK 在模型之上的**编排层**——规划、虚拟文件系统工具、subagent——模型需要知道这些存在以及什么时候用。内置 prompt 教 agent 如何使用这套脚手架，你不必每个项目重新推导一遍。**通过 profile 或自己的 `system_prompt=` 来调，而非整体复制粘贴**。

Middleware 添加特殊工具（如文件系统工具）时，会把它们附加到 system prompt。

每个 deep agent 应该包含**针对自己用例的自定义 system prompt**：

```python
from deepagents import create_deep_agent

research_instructions = """\
You are an expert researcher. Your job is to conduct \
thorough research, and then write a polished report. \
"""

agent = create_deep_agent(
    model="openai:gpt-5.4",
    system_prompt=research_instructions,
)
```

### Prompt 组装（4 段式）

Deep Agents 从最多 4 个命名部分组装 system prompt，让调用者提供的指令、SDK 内置 agent 指南、以及模型特定 [profile](#profiles) 覆盖能**共存且优先级可预测**。没有这种分层，为 Claude 调好的 profile 后缀可能根据调用顺序覆盖或被覆盖 `system_prompt=` 参数；命名 slot 让顺序明确稳定。

实际上，大多数调用者只接触 2 个 slot：**`USER`（你的 `system_prompt=`）和 `BASE`（SDK 默认）**。选择带内置 profile 的模型（目前是 Anthropic 或 OpenAI）会加一个 `SUFFIX`。完整 4 段组装主要在你写自定义 `HarnessProfile` 或调试 profile 文本位置时用。

4 个命名部分（每个都可缺失）：

| 名称 | 来源 | 备注 |
|------|------|------|
| `USER` | `system_prompt=` 参数 | `str` 或 `SystemMessage`；未设时省略 |
| `BASE` | SDK 默认（`BASE_AGENT_PROMPT`） | 总是存在，除非被 profile 的 `CUSTOM` 替换 |
| `CUSTOM` | `HarnessProfile.base_system_prompt` | profile 设置后**整体替换**`BASE` |
| `SUFFIX` | `HarnessProfile.system_prompt_suffix` | profile 设置后**附在最后** |

顺序总是 **`USER` -> (`BASE` 或 `CUSTOM`) -> `SUFFIX`**，用空行（`\n\n`）连接。两条不变量：

1. **`USER` 总在最前**——调用者的文本先于 SDK 或 profile 内容，persona/指令优先于模型选择
2. **`SUFFIX` 总在最后**——profile 后缀挨着对话历史，模型微调指南放这里最稳

组装形态（✓=字段已设，-=未设）：

| `system_prompt=` | profile `CUSTOM` | profile `SUFFIX` | 最终组装 |
|------------------|:---------------:|:----------------:|---------|
| `None` | - | - | `BASE` |
| `None` | - | ✓ | `BASE` + `SUFFIX` |
| `None` | ✓ | - | `CUSTOM` |
| `None` | ✓ | ✓ | `CUSTOM` + `SUFFIX` |
| `str` | - | - | `USER` + `BASE` |
| `str` | - | ✓ | `USER` + `BASE` + `SUFFIX` |
| `str` | ✓ | - | `USER` + `CUSTOM` |
| `str` | ✓ | ✓ | `USER` + `CUSTOM` + `SUFFIX` |

**实例**——内置 profile（Anthropic、OpenAI）只提供 `system_prompt_suffix`，所以典型调用落在 `str` + `-` + `✓` 这一行：

```python
agent = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    system_prompt="You are a customer-support agent for ACME Corp.",
)
# Final = USER + BASE + SUFFIX
#       = "You are a customer-support agent for ACME Corp."
#         + "\n\n"
#         + BASE_AGENT_PROMPT
#         + "\n\n"
#         + <Claude-specific guidance>
```

传 `SystemMessage`（而非字符串）触发另一条拼接路径：右侧组装（`BASE` 或 `CUSTOM` 加任何 `SUFFIX`）作为额外的 text content block 附加到 message 的现有 `content_blocks`。同样的逻辑顺序适用（调用者 block 在前），调用者 block 上的任何 `cache_control` 标记**会被保留**——对放置 Anthropic prompt-cache 断点很有用。

### Subagent prompt

同样的覆盖规则适用于声明式 [subagents](#subagents)——每个 subagent 用**自己的模型**重新做 profile 解析，然后把解析后的 profile 的 `base_system_prompt` / `system_prompt_suffix` 应用到它编写的 `system_prompt`。subagent 的 `system_prompt` 扮演 `BASE` 角色；`CUSTOM` 和 `SUFFIX` 来自匹配 subagent 模型的 profile（可能和主 agent 的 profile 不同）。

subagent 没有 `USER` slot——spec 编写的 `system_prompt` 是最接近的类比，停在 `BASE` slot。

### 通用 subagent prompt

自动添加的[通用 subagent](#subagents) 多一层覆盖：GP 基础 prompt 解析顺序是 **`general_purpose_subagent.system_prompt`（如已设）-> `HarnessProfile.base_system_prompt`（如已设）-> SDK GP 默认**。profile 后缀无论如何都附加。

两个覆盖字段都能承载 base-prompt 替换，但**不可互换**：`general_purpose_subagent.system_prompt` 是 GP 特定配置；`base_system_prompt` 是主要针对主 agent 的全局覆盖。两个都设置时，**GP 特定意图胜出**给 GP subagent——用户调这两个字段时永远不会看到 GP 覆盖被静默丢弃：

```python
register_harness_profile(
    "anthropic",
    HarnessProfile(
        base_system_prompt="You are ACME's support orchestrator.",  # 主 agent
        general_purpose_subagent=GeneralPurposeSubagentProfile(
            system_prompt="You are a research subagent. Cite sources.",  # GP subagent
        ),
        system_prompt_suffix="Always think step by step.",
    ),
)
```

| 栈 | 最终 system prompt |
|----|-------------------|
| 主 agent | `"You are ACME's support orchestrator." + SUFFIX` |
| GP subagent | `"You are a research subagent. Cite sources." + SUFFIX` |

---

## Middleware

Deep Agents 支持任意 [middleware](https://docs.langchain.com/oss/python/langchain/middleware/overview)，包括下面列出的内置 middleware、LangChain 预构建 middleware、provider 特定 middleware、以及你自己写的自定义 middleware。通过 `create_deep_agent` 的 `middleware` 参数传入。

### 默认 Middleware

默认情况下，Deep Agents 可以访问以下 middleware：

- **`TodoListMiddleware`**：跟踪和管理 todo 清单，组织 agent 任务
- **`FilesystemMiddleware`**：处理文件系统操作（读、写、目录导航）
- **`SubAgentMiddleware`**：派生和协调 subagent，把任务委派给特化 agent
- **`SummarizationMiddleware`**：会话变长时压缩消息历史以保持 context 限制内
- **`AnthropicPromptCachingMiddleware`**：使用 Anthropic 模型时自动减少冗余 token 处理
- **`PatchToolCallsMiddleware`**：tool call 在收到结果前被中断或取消时自动修复消息历史

如果使用 memory、skills 或 human-in-the-loop，还包括：

- **`MemoryMiddleware`**：`memory` 参数提供时跨会话持久化和检索对话上下文
- **`SkillsMiddleware`**：`skills` 参数提供时启用自定义 skills
- **`HumanInTheLoopMiddleware`**：`interrupt_on` 参数提供时在指定点暂停等待人工

### 预构建 Middleware

LangChain 暴露额外的预构建 middleware 让你加入各种功能（重试、降级、PII 检测等）。详见 [Prebuilt middleware](https://docs.langchain.com/oss/python/langchain/middleware/built-in)。

`deepagents` 库还暴露 [`create_summarization_tool_middleware`](https://reference.langchain.com/python/deepagents/middleware/summarization/create_summarization_tool_middleware)，让 agent 在合适时机（如任务之间）而非固定 token 间隔触发摘要。

### Provider 特定 Middleware

针对特定 LLM provider 优化的 middleware，详见 [Official integrations](https://docs.langchain.com/oss/python/integrations/middleware#official-integrations) 和 [Community integrations](https://docs.langchain.com/oss/python/integrations/middleware#community-integrations)。

### 自定义 Middleware

你可以提供额外 middleware 扩展功能、添加工具、实现自定义 hook：

```python
from langchain.agents.middleware import wrap_tool_call
from langchain.tools import tool
from deepagents import create_deep_agent


@tool
def get_weather(city: str) -> str:
    """Get the weather in a city."""
    return f"The weather in {city} is sunny."


call_count = [0]  # 用 list 让嵌套函数里能修改


@wrap_tool_call
def log_tool_calls(request, handler):
    """Intercept and log every tool call - demonstrates cross-cutting concern."""
    call_count[0] += 1
    tool_name = request.name if hasattr(request, "name") else str(request)

    print(f"[Middleware] Tool call #{call_count[0]}: {tool_name}")
    print(f"[Middleware] Arguments: {request.args if hasattr(request, 'args') else 'N/A'}")

    result = handler(request)

    print(f"[Middleware] Tool call #{call_count[0]} completed")
    return result


agent = create_deep_agent(
    model="openai:gpt-5.4",
    tools=[get_weather],
    middleware=[log_tool_calls],
)
```

### ⚠️ 不要在初始化后 mutate 属性

如果你需要跨 hook 调用追踪值（如计数器、累积数据），**使用 graph state**。Graph state 按 thread 作用域设计，并发更新安全。

**正确做法：**

```python
from langchain.agents.middleware import AgentMiddleware


class CustomMiddleware(AgentMiddleware):
    def __init__(self):
        pass

    def before_agent(self, state, runtime):
        return {"x": state.get("x", 0) + 1}  # 更新 graph state 而非 self
```

**错误做法：**

```python
class CustomMiddlewareBad(AgentMiddleware):
    def __init__(self):
        self.x = 1

    def before_agent(self, state, runtime):
        self.x += 1  # 突变导致竞态条件
```

原地突变（如 `before_agent` 里改 `self.x` 或 hook 里改其他共享值）会导致微妙的 bug 和竞态条件，因为很多操作并发执行（subagent、并行 tool、不同 thread 上的并行调用）。

---

## Interpreters

使用 [interpreters](interpreters.md) 添加一个 `eval` 工具，在受限 QuickJS runtime 里跑 JavaScript。Interpreter 适合 agent **以编程方式组合工具**、批处理、用代码处理错误、变换结构化数据——而不需要完整 shell 环境：

```python
from deepagents import create_deep_agent
from langchain_quickjs import CodeInterpreterMiddleware

agent = create_deep_agent(
    model="openai:gpt-5.4",
    middleware=[CodeInterpreterMiddleware()],
)
```

---

## Subagents

为了**隔离详细工作避免 context 膨胀**，使用 subagent：

```python
import os
from typing import Literal

from deepagents import create_deep_agent
from tavily import TavilyClient

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


research_subagent = {
    "name": "research-agent",
    "description": "Used to research more in depth questions",
    "system_prompt": "You are a great researcher",
    "tools": [internet_search],
    "model": "openai:gpt-5.4",  # 可选覆盖，默认用主 agent 模型
}
subagents = [research_subagent]

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    subagents=subagents,
)
```

---

## Backends

工具可以使用虚拟文件系统存储、访问、编辑文件。默认使用 [`StateBackend`](backends.md)。完整说明见 [Backends](backends.md)。

### StateBackend（默认）

线程作用域，存在 `langgraph` state 里。文件在 thread 内跨 turn 持久化（通过 checkpointer），不跨线程共享。

```python
from deepagents import create_deep_agent
from deepagents.backends import StateBackend

agent = create_deep_agent(model="google_genai:gemini-3.5-flash")  # 默认就是 StateBackend
```

### FilesystemBackend

本地机器的文件系统。**警告：** 给 agent 直接读写文件系统访问权。请谨慎使用且只在合适环境。

```python
from deepagents.backends import FilesystemBackend

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    backend=FilesystemBackend(root_dir=".", virtual_mode=True),
)
```

> 把 `FilesystemBackend` 包在 `CompositeBackend` 里，**防止 agent 内部数据**（offload 的工具结果、对话历史）**和你的项目文件一起被写入磁盘**。

### LocalShellBackend

带 shell 执行的文件系统。提供文件系统工具加 `execute` 工具运行命令。**警告：** 给 agent 直接文件系统读写权 + 不受限的 shell 执行。极度谨慎。

```python
from deepagents.backends import LocalShellBackend

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    backend=LocalShellBackend(root_dir=".", virtual_mode=True, env={"PATH": "/usr/bin:/bin"}),
)
```

### StoreBackend

提供**跨线程持久化**的长期存储。

```python
from deepagents.backends import StoreBackend
from langgraph.store.memory import InMemoryStore

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    backend=StoreBackend(
        namespace=lambda rt: (rt.server_info.user.identity,),
    ),
    store=InMemoryStore(),  # 本地开发用；LangSmith Deployment 时省略
)
```

> 部署到 [LangSmith Deployment](https://docs.langchain.com/langsmith/deployment) 时省略 `store` 参数，平台自动为 agent 配置 store。
>
> `namespace` 参数控制数据隔离。多用户部署**总是要设 namespace factory** 隔离每个用户/租户的数据。

### ContextHubBackend

LangSmith Hub 仓库里的持久化文件系统存储。

```python
from deepagents.backends import ContextHubBackend

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    backend=ContextHubBackend("my-agent"),
)
```

### CompositeBackend

灵活的后端，可以指定不同路由指向不同后端。

```python
from deepagents.backends import CompositeBackend, StateBackend, StoreBackend
from langgraph.store.memory import InMemoryStore

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    backend=CompositeBackend(
        default=StateBackend(),
        routes={
            "/memories/": StoreBackend(namespace=lambda _rt: ("memories",)),
        },
    ),
    store=InMemoryStore(),  # store 传给 create_deep_agent，不传给 backend
)
```

### Sandboxes

Sandbox 是特化的 [backend](backends.md)，agent 代码在隔离环境里运行，有自己的文件系统和 `execute` 工具运行 shell 命令。详见 [Sandboxes](sandboxes.md)。

支持 Modal / Runloop / Daytona / LangSmith 等，所有 sandbox 通过 `backend=` 参数传入。

---

## Human-in-the-loop

某些工具操作可能敏感，需要人工审批。可以**为每个工具配置审批**：

```python
from langchain.tools import tool
from deepagents import create_deep_agent
from langgraph.checkpoint.memory import MemorySaver


@tool
def remove_file(path: str) -> str:
    """Delete a file from the filesystem."""
    return f"Deleted {path}"


@tool
def fetch_file(path: str) -> str:
    """Read a file from the filesystem."""
    return f"Contents of {path}"


@tool
def notify_email(to: str, subject: str, body: str) -> str:
    """Send an email."""
    return f"Sent email to {to}"


# Checkpointer 对 human-in-the-loop 是必需的
checkpointer = MemorySaver()

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    tools=[remove_file, fetch_file, notify_email],
    interrupt_on={
        "remove_file": True,                                       # 默认：approve/edit/reject/respond
        "fetch_file": False,                                       # 不需要中断
        "notify_email": {"allowed_decisions": ["approve", "reject"]},  # 不允许编辑
    },
    checkpointer=checkpointer,  # 必需！
)
```

可以为 agent 和 subagent 在 tool call 上配置中断，也可以从 tool call **内部**配置。详见 [Human-in-the-loop](human-in-the-loop.md)。

---

## Skills

可以用 [skills](overview.md#核心能力11-项) 为 deep agent 提供新能力和专长。与 [tools](#tools) 倾向于覆盖低层功能（如原生文件系统操作或规划）不同，skills 可以包含**详细的任务完成指令、参考信息、其他资产**（如模板）。

这些文件**只在 agent 判断 skill 对当前 prompt 有用时才加载**。这种**渐进式披露**减少了启动时 agent 需要考虑的 token 和 context 数量。

示例 skill 见 [Deep Agents example skills](https://github.com/langchain-ai/deepagentsjs/tree/main/examples/skills)。

把 skill 作为 `create_deep_agent` 的参数传入：

### StateBackend

```python
from urllib.request import urlopen
from deepagents import create_deep_agent
from deepagents.backends import StateBackend
from deepagents.backends.utils import create_file_data
from langchain_quickjs import CodeInterpreterMiddleware
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()
backend = StateBackend()

skill_url = "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/libs/cli/examples/skills/langgraph-docs/SKILL.md"
with urlopen(skill_url) as response:
    skill_content = response.read().decode('utf-8')

skills_files = {
    "/skills/langgraph-docs/SKILL.md": create_file_data(skill_content),
}

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    backend=backend,
    skills=["/skills/"],
    checkpointer=checkpointer,
    middleware=[CodeInterpreterMiddleware(skills_backend=backend)],  # interpreter skill
)

result = agent.invoke(
    {
        "messages": [{"role": "user", "content": "What is langgraph?"}],
        # 给默认 StateBackend 的 in-state 文件系统种子（虚拟路径必须以 "/" 开头）
        "files": skills_files,
    },
    config={"configurable": {"thread_id": "12345"}},
)
```

### StoreBackend

```python
from urllib.request import urlopen
from deepagents import create_deep_agent
from deepagents.backends import StoreBackend
from deepagents.backends.utils import create_file_data
from langchain_quickjs import CodeInterpreterMiddleware
from langgraph.store.memory import InMemoryStore

store = InMemoryStore()
backend = StoreBackend(namespace=lambda _rt: ("filesystem",))

skill_url = "..."
with urlopen(skill_url) as response:
    skill_content = response.read().decode('utf-8')

store.put(
    namespace=("filesystem",),
    key="/skills/langgraph-docs/SKILL.md",
    value=create_file_data(skill_content),
)

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    backend=backend,
    store=store,
    skills=["/skills/"],
    middleware=[CodeInterpreterMiddleware(skills_backend=backend)],
)
```

---

## Memory

使用 [`AGENTS.md` 文件](https://agents.md/) 为 deep agent 提供额外 context。

把一个或多个文件路径传给 `memory` 参数：

### StateBackend

```python
from urllib.request import urlopen
from deepagents import create_deep_agent
from deepagents.backends.utils import create_file_data
from langgraph.checkpoint.memory import MemorySaver

with urlopen(
    "https://raw.githubusercontent.com/langchain-ai/deepagents/refs/heads/main/examples/text-to-sql-agent/AGENTS.md"
) as response:
    agents_md = response.read().decode("utf-8")

checkpointer = MemorySaver()

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    memory=["/AGENTS.md"],
    checkpointer=checkpointer,
)

result = agent.invoke(
    {
        "messages": [{"role": "user", "content": "Please tell me what's in your memory files."}],
        "files": {"/AGENTS.md": create_file_data(agents_md)},
    },
    config={"configurable": {"thread_id": "123456"}},
)
```

### StoreBackend / FilesystemBackend

类似的模式，详见 [API Reference](api-reference.md) 和示例仓库。

---

## Profiles

Profile 让你用预定义配置为特定模型微调 agent 行为。

详见 [Profiles](https://docs.langchain.com/oss/python/deepagents/profiles)。
