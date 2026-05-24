# Sandboxes

> Source: https://docs.langchain.com/oss/python/deepagents/sandboxes
> Collected: 2026-05-24

在隔离环境中执行代码。

Agent 会生成代码、与文件系统交互、运行 shell 命令。因为我们无法预测 agent 会做什么，**它的环境必须隔离**——这样它就访问不到凭证、文件或网络。Sandbox 通过在 agent 执行环境和你的宿主系统之间建立**边界**来提供这种隔离。

在 Deep Agents 里，**sandbox 就是 [backend](backends.md)**——定义 agent 操作的环境。其他 backend（State、Filesystem、Store）只暴露文件操作，sandbox backend 额外给 agent 一个 `execute` 工具运行 shell 命令。配置 sandbox backend 后，agent 获得：

- 所有标准文件系统工具（`ls`、`read_file`、`write_file`、`edit_file`、`glob`、`grep`）
- `execute` 工具运行 sandbox 内任意 shell 命令
- 保护宿主系统的安全边界

## 为什么用 sandbox

Sandbox 用于**安全**。它们让 agent 执行任意代码、访问文件、使用网络——而不危及凭证、本地文件或宿主系统。当 agent 自主运行时，这种隔离至关重要。

Sandbox 特别适合：

- **编码 agent**：自主运行的 agent 可以用 shell、git、clone 仓库（很多 provider 提供原生 git API，如 [Daytona git 操作](https://www.daytona.io/docs/en/git-operations/)），运行 Docker-in-Docker 做构建测试 pipeline
- **数据分析 agent**：加载文件、安装数据分析库（pandas、numpy 等）、运行统计计算、创建 PowerPoint 等输出——在安全隔离环境

> **使用 Deep Agents Code 的话**：内置 sandbox 支持，通过 `--sandbox` flag。详见 [Use remote sandboxes](https://docs.langchain.com/oss/python/deepagents/code/remote-sandboxes)。

## 基础用法

下面的例子假设你已经用 provider SDK 创建好 sandbox/devbox 且配好凭证。

### Modal

```bash
pip install langchain-modal
```

```python
import modal
from deepagents import create_deep_agent
from langchain_anthropic import ChatAnthropic
from langchain_modal import ModalSandbox

app = modal.App.lookup("your-app")
modal_sandbox = modal.Sandbox.create(app=app)
backend = ModalSandbox(sandbox=modal_sandbox)

agent = create_deep_agent(
    model=ChatAnthropic(model="claude-sonnet-4-6"),
    system_prompt="You are a Python coding assistant with sandbox access.",
    backend=backend,
)
try:
    result = agent.invoke({
        "messages": [{"role": "user", "content": "Create a small Python package and run pytest"}]
    })
finally:
    modal_sandbox.terminate()
```

### Runloop

```bash
pip install langchain-runloop
```

```python
import os
from deepagents import create_deep_agent
from langchain_anthropic import ChatAnthropic
from langchain_runloop import RunloopSandbox
from runloop_api_client import RunloopSDK

client = RunloopSDK(bearer_token=os.environ["RUNLOOP_API_KEY"])
devbox = client.devbox.create()
backend = RunloopSandbox(devbox=devbox)

agent = create_deep_agent(
    model=ChatAnthropic(model="claude-sonnet-4-6"),
    system_prompt="You are a Python coding assistant with sandbox access.",
    backend=backend,
)

try:
    result = agent.invoke({
        "messages": [{"role": "user", "content": "Create a small Python package and run pytest"}]
    })
finally:
    devbox.shutdown()
```

### Daytona

```bash
pip install langchain-daytona
```

```python
from daytona import Daytona
from deepagents import create_deep_agent
from langchain_anthropic import ChatAnthropic
from langchain_daytona import DaytonaSandbox

sandbox = Daytona().create()
backend = DaytonaSandbox(sandbox=sandbox)

agent = create_deep_agent(
    model=ChatAnthropic(model="claude-sonnet-4-6"),
    system_prompt="You are a Python coding assistant with sandbox access.",
    backend=backend,
)

try:
    result = agent.invoke({
        "messages": [{"role": "user", "content": "Create a small Python package and run pytest"}]
    })
finally:
    sandbox.stop()
```

### LangSmith（私有 beta）

```bash
pip install "langsmith[sandbox]"
```

```python
from deepagents import create_deep_agent
from deepagents.backends import LangSmithSandbox
from langchain_anthropic import ChatAnthropic
from langsmith.sandbox import SandboxClient

client = SandboxClient()
ls_sandbox = client.create_sandbox()
backend = LangSmithSandbox(sandbox=ls_sandbox)

agent = create_deep_agent(
    model=ChatAnthropic(model="claude-sonnet-4-6"),
    system_prompt="You are a Python coding assistant with sandbox access.",
    backend=backend,
)
try:
    result = agent.invoke({
        "messages": [{"role": "user", "content": "Create a small Python package and run pytest"}]
    })
finally:
    client.delete_sandbox(ls_sandbox.name)
```

## 生命周期与作用域

大多数应用选择**每个 [thread](https://docs.langchain.com/langsmith/use-threads) 一个 sandbox（thread-scoped）**或**同一 [assistant](https://docs.langchain.com/langsmith/assistants) 上所有 thread 共享一个 sandbox（assistant-scoped）**。

⚠️ **Sandbox 消耗资源和金钱直到关闭**。不再使用时务必关闭。

### Thread-scoped（默认）

每个对话有自己的 sandbox。第一次运行创建它，同一 thread 后续 turn 复用它。Thread 结束或 sandbox TTL 过期时环境消失。

> **用户可能在空闲后回来时**：在 sandbox 上配 TTL，让 provider 自动删除或归档空闲环境。

```python
from daytona import CreateSandboxFromSnapshotParams, Daytona
from deepagents import create_deep_agent
from langchain_core.runnables import RunnableConfig
from langchain_daytona import DaytonaSandbox

client = Daytona()


async def agent(config: RunnableConfig):
    thread_id = config["configurable"]["thread_id"]
    try:
        sandbox = await client.find_one(labels={"thread_id": thread_id})
    except Exception:
        sandbox = await client.create(
            CreateSandboxFromSnapshotParams(
                labels={"thread_id": thread_id},
                auto_delete_interval=3600,  # TTL：空闲时清理
            )
        )
    return create_deep_agent(
        model="google_genai:gemini-3.5-flash",
        backend=DaytonaSandbox(sandbox=sandbox),
    )
```

### Assistant-scoped

同一 assistant 上每个 thread 复用一个 sandbox。文件、安装的包、克隆的仓库**跨对话持久化**。

⚠️ **Assistant-scoped sandbox 会随时间累积状态**。配 TTL、用 snapshot 周期性重置、或实现清理逻辑——避免磁盘和内存无限增长。

```python
async def agent(config: RunnableConfig):
    assistant_id = config["configurable"]["assistant_id"]
    try:
        sandbox = await client.find_one(labels={"assistant_id": assistant_id})
    except Exception:
        sandbox = await client.create(
            CreateSandboxFromSnapshotParams(labels={"assistant_id": assistant_id})
        )
    return create_deep_agent(
        model="google_genai:gemini-3.5-flash",
        backend=DaytonaSandbox(sandbox=sandbox),
    )
```

## 集成模式（两种架构）

### Agent in sandbox 模式

Agent **在 sandbox 内**运行，你通过网络和它通信。构建带预装 agent 框架的 Docker/VM image，在 sandbox 内运行，从外面发消息。

| 优点 | 缺点 |
|------|------|
| ✅ 接近本地开发 | 🔴 API key 必须在 sandbox 里（安全风险） |
| ✅ Agent 和环境紧耦合 | 🔴 更新要重建 image |
| | 🔴 需要通信基础设施（WebSocket 或 HTTP 层） |

```dockerfile
FROM python:3.11
RUN pip install deepagents-code
```

### Sandbox as tool 模式（推荐）

Agent 在**你的机器或服务器**上运行。需要执行代码时，它调用 sandbox 工具（`execute`、`read_file`、`write_file` 等），这些工具调用 provider API 在远程 sandbox 操作。

| 优点 | 缺点 |
|------|------|
| ✅ 即时更新 agent 代码，无需重建 image | 🔴 每次执行调用有网络延迟 |
| ✅ Agent 状态和执行分离更干净 | |
| ✅ API key 留在 sandbox 外 | |
| ✅ Sandbox 失败不丢 agent 状态 | |
| ✅ 可在多 sandbox 并行 | |
| ✅ 只为执行时间付费 | |

**何时选哪个：**
- **Agent in sandbox**：provider SDK 处理通信层 + 想让生产和本地一致
- **Sandbox as tool**：快速迭代 agent 逻辑 + API key 留在 sandbox 外 + 偏好关注点分离

## Sandbox 工作原理

### 隔离边界

所有 sandbox provider 都保护宿主系统免受 agent 文件系统和 shell 操作影响。Agent **无法**读取本地文件、访问机器环境变量、干扰其他进程。

⚠️ 但 sandbox **不**保护：

- **Context injection**：攻击者控制部分 agent 输入，可以指示它在 sandbox 内运行任意命令。Sandbox 是隔离的，但 agent 在里面有完全控制。
- **网络外泄**：除非屏蔽网络，否则被 context-injection 的 agent 可以通过 HTTP 或 DNS 发数据出 sandbox。一些 provider 支持屏蔽网络（如 Modal 的 `blockNetwork: true`）。

### `execute` 方法

Sandbox backend 架构简单：**provider 必须实现的唯一方法是 `execute()`**——运行 shell 命令并返回输出。每个其他文件系统操作（`read`、`write`、`edit`、`ls`、`glob`、`grep`）都由 [`BaseSandbox`](https://reference.langchain.com/python/deepagents/backends/sandbox/BaseSandbox) 基类**基于 `execute()` 构建**。

这种设计意味着：

- **加新 provider 很简单**——实现 `execute()`，基类处理其他一切
- **`execute` 工具条件性可用**——每次模型调用时，harness 检查 backend 是否实现 [`SandboxBackendProtocol`](https://reference.langchain.com/python/deepagents/backends/protocol/SandboxBackendProtocol)。如果没实现，工具被过滤掉，agent 永远看不到它

Agent 调 `execute` 时提供 `command` 字符串，得到合并的 stdout/stderr、exit code、以及输出过大时的截断通知。

你也可以**直接在应用代码里**调 backend `execute()` 方法：

```python
from daytona import Daytona
from langchain_daytona import DaytonaSandbox

sandbox = Daytona().create()
backend = DaytonaSandbox(sandbox=sandbox)

result = backend.execute("python --version")
print(result.output)
```

### 两个文件访问平面

文件进出 sandbox 有两种**截然不同**的方式：

**Agent 文件系统工具**：`read_file`、`write_file`、`edit_file`、`ls`、`glob`、`grep` 和 `execute` 是 LLM 在执行时调用的工具。它们通过 sandbox 内 `execute()` 运行。Agent 用它们读代码、写文件、运行命令。

**文件传输 API**：你的应用代码调用的 `upload_files()` 和 `download_files()` 方法。它们用 provider **原生文件传输 API**（不是 shell 命令）在宿主和 sandbox 之间移动文件。用于：

- **给 sandbox 种子**：agent 运行前放入源码、配置、数据
- **取回 artifacts**：agent 完成后取生成代码、构建输出、报告
- **预装依赖**：agent 将需要的依赖

## 文件操作

### 种子 sandbox（upload_files）

路径必须是**绝对路径**，内容是 `bytes`：

```python
backend.upload_files([
    ("/src/index.py", b"print('Hello')\n"),
    ("/pyproject.toml", b"[project]\nname = 'my-app'\n"),
])
```

### 取回 artifacts（download_files）

```python
results = backend.download_files(["/src/index.py", "/output.txt"])
for result in results:
    if result.content is not None:
        print(f"{result.path}: {result.content.decode()}")
    else:
        print(f"Failed to download {result.path}: {result.error}")
```

## 安全考量

Sandbox 隔离代码执行，但**不保护 context injection**。控制部分 agent 输入的攻击者可以指示它从 sandbox 内读文件、运行命令、外泄数据。**这让 sandbox 内的凭证特别危险**。

### ⚠️ 永远不要把密钥放进 sandbox

API key、token、数据库凭证等通过环境变量、挂载文件、`secrets` 选项注入 sandbox 的密钥，**都能被 context-injected agent 读取和外泄**。即便是短期或受限凭证——只要 agent 能访问，攻击者也能。

### 安全处理密钥的两种方式

1. **把密钥放在 sandbox 外的工具里**（推荐）。在宿主环境（不在 sandbox 里）定义工具，处理认证。Agent 按名字调这些工具，但**永远看不到凭证**。

2. **用注入凭证的网络代理**。一些 sandbox provider 支持代理：拦截 sandbox 出站 HTTP 请求，在转发前附加凭证（如 `Authorization` header）。Agent 永远看不到密钥——只是向 URL 发普通请求。这种方式目前未普遍可用。

### 必须注入密钥时（不推荐）

- 给**所有** tool call 启用 [human-in-the-loop](human-in-the-loop.md) 审批，不只是敏感工具
- 屏蔽或限制 sandbox 网络访问
- 用最窄凭证作用域 + 最短生命周期
- 监控 sandbox 网络流量

⚠️ 即使有这些保护，这仍是不安全 workaround。足够创意的 context injection 攻击能绕过输出过滤和 HITL 审查。

### 通用最佳实践

- 在你的应用基于 sandbox 输出行动前**审查输出**
- 不需要时**屏蔽 sandbox 网络访问**
- 用 middleware **过滤或编辑工具输出中的敏感模式**
- 把 sandbox 内产生的一切**当作不可信输入**
