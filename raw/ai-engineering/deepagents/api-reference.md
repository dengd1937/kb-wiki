# `deepagents` API Reference 速查

> Source: https://reference.langchain.com/python/deepagents/
> Collected: 2026-05-24

`deepagents` 包完整 API 表面结构化清单。完整签名和参数详情请回到官方 reference。

## 核心函数

### Agent 创建

| 函数 | 说明 |
|------|------|
| **`create_deep_agent()`** | 主入口，实例化 deep agent |
| `get_default_model()` | 获取默认语言模型配置 |

### 模型解析

| 函数 | 说明 |
|------|------|
| `resolve_model()` | 把模型规格转为可用实例 |
| `get_model_identifier()` | 提取模型标识符字符串 |
| `get_model_provider()` | 判断模型 provider |
| `model_matches_spec()` | 按规格校验模型 |

## 核心类

### Agent 与 Subagent 架构

| 类 | 说明 |
|----|------|
| `SubAgent` | 单个特化 agent 组件 |
| `CompiledSubAgent` | 处理后可执行的 subagent |
| `AsyncSubAgent` | 非阻塞 subagent 变体 |
| `TaskToolSchema` | 任务执行 schema |

### 流式处理

| 类 | 说明 |
|----|------|
| `SubagentRunStream` | subagent 操作的同步流处理器 |
| `AsyncSubagentRunStream` | 异步流处理器 |
| `SubagentTransformer` | 变换 subagent 执行流 |

## Middleware 组件

### 文件系统操作

| 类 | 说明 |
|----|------|
| **`FilesystemMiddleware`** | 管理文件系统交互 |
| **`FilesystemPermission`** | 控制访问层级 |

**Schema**：`LsSchema`、`ReadFileSchema`、`WriteFileSchema`、`EditFileSchema`、`GlobSchema`、`GrepSchema`、`ExecuteSchema`

### Subagent 管理

| 类 | 说明 |
|----|------|
| **`SubAgentMiddleware`** | 编排 subagent 生命周期 |
| `AsyncSubAgentMiddleware` | 异步 subagent 操作 |
| `AsyncTask` | 异步任务支持类 |
| `AsyncSubAgentState` | 异步 subagent 状态 |

### 其他 Middleware

| 类 | 说明 |
|----|------|
| `MemoryMiddleware` | 维护对话状态 |
| `SkillsMiddleware` | 管理可用能力 |
| `SummarizationToolMiddleware` | 压缩对话历史 |
| `PatchToolCallsMiddleware` | 规范化 tool 调用 |

## Backend 系统

### 状态管理

| 类 | 说明 |
|----|------|
| **`StateBackend`** | 内存状态持久化（默认） |

### 文件操作

| 类 | 说明 |
|----|------|
| **`FilesystemBackend`** | 直接文件系统访问层 |

### 存储

| 类 | 说明 |
|----|------|
| **`StoreBackend`** | 持久化 key-value 存储 |
| `BackendContext` | 存储操作 context |

### Sandbox 环境

| 类 | 说明 |
|----|------|
| `BaseSandbox` | 隔离执行基础类 |
| **`LocalShellBackend`** | 本地命令执行 |
| **`LangSmithSandbox`** | LangSmith 集成 sandbox |

### 其他 Backend

| 类 | 说明 |
|----|------|
| **`ContextHubBackend`** | Context 聚合层（LangSmith Hub）|
| **`CompositeBackend`** | 组合多个 backend 实现 |

## Profile 与配置

### Harness Profile

| 类 | 说明 |
|----|------|
| `HarnessProfile` | 基础 profile 配置 |
| `HarnessProfileConfig` | Profile 设置容器 |
| `GeneralPurposeSubagentProfile` | 预配置通用 profile |

### Provider Profile

| 类 | 说明 |
|----|------|
| `ProviderProfile` | Vendor 特定配置（OpenAI / OpenRouter / Anthropic）|

## 协议与数据结构

### 文件操作

| 类 | 说明 |
|----|------|
| `FileInfo` | 文件元信息 |
| `FileData` | 文件数据 |
| `FileDownloadResponse` | 下载响应 |
| `FileUploadResponse` | 上传响应 |
| `ReadResult` | 读取结果 |
| `WriteResult` | 写入结果 |
| `EditResult` | 编辑结果 |
| `LsResult` | ls 结果 |

### 搜索与执行

| 类 | 说明 |
|----|------|
| `GrepMatch` | grep 匹配项 |
| `GrepResult` | grep 结果 |
| `GlobResult` | glob 结果 |
| `ExecuteResponse` | 执行响应 |
| `SandboxBackendProtocol` | Sandbox backend 协议 |

### 异步任务

| Schema | 说明 |
|--------|------|
| `StartAsyncTaskSchema` | 启动异步任务 |
| `CheckAsyncTaskSchema` | 检查异步任务状态 |
| `UpdateAsyncTaskSchema` | 更新异步任务 |
| `CancelAsyncTaskSchema` | 取消异步任务 |
| `ListAsyncTasksSchema` | 列异步任务 |

## 速查表（最常用 import）

```python
# 主入口
from deepagents import create_deep_agent

# 权限
from deepagents import FilesystemPermission

# Backend
from deepagents.backends import (
    StateBackend,
    FilesystemBackend,
    LocalShellBackend,
    StoreBackend,
    ContextHubBackend,
    CompositeBackend,
    LangSmithSandbox,
)

# Backend 工具
from deepagents.backends.utils import create_file_data

# Profile（高级定制）
from deepagents import (
    ProviderProfile,
    HarnessProfile,
    register_provider_profile,
    register_harness_profile,
)

# Sandbox 集成（独立包）
from langchain_modal import ModalSandbox
from langchain_daytona import DaytonaSandbox
from langchain_runloop import RunloopSandbox

# Interpreter（独立包）
from langchain_quickjs import CodeInterpreterMiddleware

# LangChain middleware（用于自定义 hook）
from langchain.agents.middleware import (
    AgentMiddleware,
    wrap_tool_call,
    wrap_model_call,
    ModelRequest,
    ModelResponse,
)
```

---

完整的方法签名、参数详情、返回值规格请查阅：
<https://reference.langchain.com/python/deepagents/>
