# Backends（文件系统后端）

> Source: https://docs.langchain.com/oss/python/deepagents/backends
> Collected: 2026-05-24

Deep Agents 通过**可插拔后端**暴露文件系统操作，支持工具：`ls`、`read_file`、`write_file`、`edit_file`、`glob`、`grep`。图片文件（`.png`、`.jpg`、`.jpeg`、`.gif`、`.webp`）作为多模态内容返回。Sandbox 和 `LocalShellBackend` 还提供 `execute` 工具。

## 内置后端（6 种）

### StateBackend（默认）

存在 LangGraph agent state 里的文件，**线程作用域**，通过 checkpointer 保存。文件跨 turn 持久化但**不跨线程共享**。最适合 scratch pad 和中间结果。

```python
from deepagents.backends import StateBackend

agent = create_deep_agent(model="google_genai:gemini-3.5-flash")  # 默认就是 StateBackend
```

### FilesystemBackend

在可配置 root 目录下读写真实文件，可选 `virtual_mode=True` 做路径沙盒化。**需要谨慎**——agent 可能访问密钥。**仅推荐本地开发 CLI 用**，包在 `CompositeBackend` 里分离项目文件和 agent 内部数据。

```python
from deepagents.backends import FilesystemBackend

agent = create_deep_agent(
    model="...",
    backend=FilesystemBackend(root_dir=".", virtual_mode=True),
)
```

> **推荐模式**：用 CompositeBackend 隔离项目文件
>
> ```python
> CompositeBackend(
>     default=StateBackend(),
>     routes={
>         "/workspace/": FilesystemBackend(root_dir="/path/to/project", virtual_mode=True),
>     },
> )
> ```

### LocalShellBackend

扩展 `FilesystemBackend`，额外通过 `execute` 工具提供**不受限制的 shell 命令执行**。**仅在可信开发环境用，且启用 Human-in-the-Loop middleware**。

```python
from deepagents.backends import LocalShellBackend

agent = create_deep_agent(
    model="...",
    backend=LocalShellBackend(root_dir=".", virtual_mode=True, env={"PATH": "/usr/bin:/bin"}),
)
```

### StoreBackend

通过 LangGraph `BaseStore` 提供持久化**跨线程**存储。支持 namespace factory 做多用户隔离。v0.5.0+ 需要必传参数。

```python
from deepagents.backends import StoreBackend
from langgraph.store.memory import InMemoryStore

agent = create_deep_agent(
    model="...",
    backend=StoreBackend(
        namespace=lambda rt: (rt.server_info.user.identity,),
    ),
    store=InMemoryStore(),  # LangSmith Deployment 时省略
)
```

### ContextHubBackend

把文件持久化到 LangSmith Hub 仓库。适合**受益于 commit 历史**的工作流，且不需要单独的 store 配置。

```python
from deepagents.backends import ContextHubBackend

agent = create_deep_agent(
    model="...",
    backend=ContextHubBackend("my-agent"),
)
```

### CompositeBackend

按前缀匹配把文件系统路径路由到不同后端。**常用于把 `/memories/` 持久化、其他路径用线程作用域存储**。

```python
from deepagents.backends import CompositeBackend, StateBackend, StoreBackend
from langgraph.store.memory import InMemoryStore

agent = create_deep_agent(
    model="...",
    backend=CompositeBackend(
        default=StateBackend(),
        routes={
            "/memories/": StoreBackend(namespace=lambda _rt: ("memories",)),
        },
    ),
    store=InMemoryStore(),
)
```

## 关键配置模式

### 路由内部数据与项目文件

```python
CompositeBackend(
    default=StateBackend(),
    routes={
        "/workspace/": FilesystemBackend(root_dir="/path/to/project", virtual_mode=True),
    },
)
```

### 多用户隔离（StoreBackend namespace factory）

```python
StoreBackend(namespace=lambda rt: (rt.server_info.user.identity,))
```

## 自定义后端

实现 `BackendProtocol`，要求方法：`ls()`、`read()`、`grep()`、`glob()`、`write()`、`edit()`。

外部后端的 write/edit 结果应返回 `files_update=None`。设计时考虑**高效过滤**（尽量服务端过滤），返回带 error 字段的结构化结果类型。

## 安全与权限

### FilesystemBackend 风险

Agent 访问可读文件（包括密钥）；和网络工具结合时，密钥可能被外泄。**总是用 `virtual_mode=True` + 包在权限控制里**。

### LocalShellBackend 风险

- 任意命令执行（用用户权限）
- 永久文件修改
- 无限资源消耗

### 配套安全机制

- **`FilesystemPermission` 规则**：声明式访问控制（详见 [Permissions](permissions.md)）
- **Policy hooks**：通过子类化或包装后端实现自定义验证逻辑（路径规则之外）

## 迁移说明

**v0.5.0** 起，backend factory pattern（传 callable）**已弃用**。直接传预构造的实例——backend 现在内部解析 runtime context。
