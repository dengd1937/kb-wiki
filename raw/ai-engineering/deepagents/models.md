# Models

> Source: https://docs.langchain.com/oss/python/deepagents/models
> Collected: 2026-05-24

为 Deep Agents 配置模型 provider 和参数。

Deep Agents 支持**任何带 tool calling 能力的 LangChain 聊天模型**。

## Supported models

用 `provider:model` 格式指定模型（如 `google_genai:gemini-3.5-flash`、`openai:gpt-5.4`、`anthropic:claude-sonnet-4-6`）。Provider 前缀选择 LangChain 集成，冒号后的部分原样传给该 provider 作为模型标识符。完整 provider 字符串见 `init_chat_model` 的 `model_provider` 参数文档。

模型标识符必须匹配 provider 期望的格式。有些 provider 用简单名字（如 `gpt-5.4`），其他用命名空间 ID 或部署路径（如 `zai-org/GLM-5.1`），所以 Deep Agents 字符串就是 `baseten:zai-org/GLM-5.1`。**总是查 provider 模型目录或集成文档**找当前标识符。

### Suggested models（推荐模型）

这些模型在 Deep Agents eval suite 上表现良好（测试基本 agent 操作）。**通过这些 eval 是必要但不充分**——更长更复杂任务的强表现还需要额外验证。

| Provider | Models |
|----------|--------|
| Google | `gemini-3.1-pro-preview`, `gemini-3-flash-preview` |
| OpenAI | `gpt-5.4`, `gpt-4o`, `gpt-5.4`, `o4-mini`, `gpt-5.2-codex`, `gpt-4o-mini`, `o3` |
| Anthropic | `claude-opus-4-6`, `claude-opus-4-5`, `claude-sonnet-4-6`, `claude-sonnet-4`, `claude-sonnet-4-5`, `claude-haiku-4-5`, `claude-opus-4-1` |
| Open-weight | `GLM-5`, `Kimi-K2.5`, `MiniMax-M2.5`, `qwen3.5-397B-A17B`, `devstral-2-123B` |

开源权重模型通过 Baseten、Fireworks、OpenRouter、Ollama 等 provider 访问。

### Model evaluations

Deep Agents eval suite 测试主流模型，覆盖类别：Overall / File Ops / Retrieval / Tool Use / Memory / Conversation / Summarization。

完整评测表（13 个模型 × 各类别分数）见官方 [Eval runs](https://docs.langchain.com/oss/python/deepagents/models)。

## Configure model parameters

把 `provider:model` 格式字符串传给 `create_deep_agent`，或传配置好的模型实例做完全控制。底层用 `init_chat_model` 解析模型字符串。

要配置模型特定参数，使用 `init_chat_model` 或直接实例化 provider 模型类：

```python
from langchain.chat_models import init_chat_model
from deepagents import create_deep_agent

model = init_chat_model(
    model="google_genai:gemini-3.5-flash",
    thinking_level="medium",
)
agent = create_deep_agent(model=model)
```

```python
from langchain_google_genai import ChatGoogleGenerativeAI
from deepagents import create_deep_agent

model = ChatGoogleGenerativeAI(
    model="gemini-3.1-pro-preview",
    thinking_level="medium",
)
agent = create_deep_agent(model=model)
```

可用参数因 provider 而异。详见各 provider 的聊天模型集成页。

### Provider profiles

`ProviderProfile` 打包**当你用 `provider:model` 字符串创建 deep agent 时**应用的初始化参数。**对 `init_chat_model` 已预配置的模型不适用**。

可以在两个级别注册（两者可共存）：

- **Provider 级**——裸 provider key 如 `"openai"`，应用于所有 `openai` provider 模型
- **Model 级**——`provider:model` key 如 `"openai:gpt-5.4"`，仅应用于该模型，**合并在 provider 级 profile 之上**

```python
from deepagents import ProviderProfile, register_provider_profile

# Provider 级默认：所有 openai 模型 temperature=0
register_provider_profile(
    "openai",
    ProviderProfile(init_kwargs={"temperature": 0}),
)

# Model 级覆盖：gpt-5.4 额外设特定推理强度
# 继承上面 provider 级的 temperature=0
register_provider_profile(
    "openai:gpt-5.4",
    ProviderProfile(init_kwargs={"reasoning_effort": "medium"}),
)
```

完整字段、合并语义、插件打包见 [Profiles](https://docs.langchain.com/oss/python/deepagents/profiles)。

> 想塑造 agent 构建**模型之后**的行为，用 **harness profile**（不是 provider profile）。

## Select a model at runtime（运行时选模型）

如果你的应用让用户选模型（如 UI 下拉），用 middleware **在每次调用时切换模型**，不用重建 agent。

把用户的模型选择通过 runtime context 传入，然后用 `wrap_model_call` middleware 在每次调用时覆盖模型：

```python
from dataclasses import dataclass
from langchain.chat_models import init_chat_model
from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
from deepagents import create_deep_agent
from typing import Callable


@dataclass
class Context:
    model: str


@wrap_model_call
def configurable_model(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse],
) -> ModelResponse:
    model_name = request.runtime.context.model
    model = init_chat_model(model_name)
    return handler(request.override(model=model))


agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    middleware=[configurable_model],
    context_schema=Context,
)

# 用用户的模型选择调用
result = agent.invoke(
    {"messages": [{"role": "user", "content": "Hello!"}]},
    context=Context(model="openai:gpt-5.4"),
)
```

更动态的模型模式（如基于对话复杂度路由、成本优化）见 LangChain agents 指南的 [Dynamic model](https://docs.langchain.com/oss/python/langchain/agents)。

## 延伸阅读

- [Models in LangChain](https://docs.langchain.com/oss/python/langchain/models)：tool calling / 结构化输出 / 多模态
