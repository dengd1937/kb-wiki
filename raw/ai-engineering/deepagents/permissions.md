# Permissions（文件系统权限）

> Source: https://docs.langchain.com/oss/python/deepagents/permissions
> Collected: 2026-05-24

用声明式权限规则控制 agent 可读/可写哪些文件和目录。

把规则列表传给 `permissions=`，agent 的内置文件系统工具就会遵守。

> 需要 `deepagents>=0.5.2`。

⚠️ **权限仅适用于内置文件系统工具**（`ls`、`read_file`、`glob`、`grep`、`write_file`、`edit_file`）。**自定义工具和 MCP 工具不在覆盖范围**。权限也**不适用于 sandbox backend**——sandbox 支持通过 `execute` 工具任意命令执行。

**何时用：**
- 需要**基于路径的 allow/deny 规则**作用于内置文件系统工具 → 用 `permissions`
- 需要**自定义验证逻辑**（速率限制、审计日志、内容检查）或控制自定义工具 → 用 backend policy hook

## 基础用法

把 `FilesystemPermission` 规则列表传给 `create_deep_agent`。规则**按声明顺序求值**，第一个匹配的规则胜出。如果没规则匹配，操作**被允许**。

```python
from deepagents import FilesystemPermission, create_deep_agent

# 只读 agent：deny 所有写
agent = create_deep_agent(
    model=model,
    backend=backend,
    permissions=[
        FilesystemPermission(
            operations=["write"],
            paths=["/**"],
            mode="deny",
        ),
    ],
)
```

## 规则结构

每个 `FilesystemPermission` 有 3 个字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| `operations` | `list["read" \| "write"]` | 规则适用于哪些操作。`"read"` 覆盖 `ls`/`read_file`/`glob`/`grep`；`"write"` 覆盖 `write_file`/`edit_file` |
| `paths` | `list[str]` | 匹配文件路径的 glob 模式（如 `["/workspace/**"]`）。支持 `**` 递归匹配和 `{a,b}` 替代 |
| `mode` | `"allow" \| "deny"` | 允许还是拒绝匹配的操作。默认 `"allow"` |

规则用**首匹配胜出（first-match-wins）**求值：第一个 `operations` 和 `paths` 匹配当前调用的规则决定结果。**没规则匹配则默认允许**（permissive 默认值）。

## 示例

### 隔离到 workspace 目录

只允许在 `/workspace/` 下读写，拒绝其他：

```python
agent = create_deep_agent(
    model=model,
    backend=backend,
    permissions=[
        FilesystemPermission(
            operations=["read", "write"],
            paths=["/workspace/**"],
            mode="allow",
        ),
        FilesystemPermission(
            operations=["read", "write"],
            paths=["/**"],
            mode="deny",
        ),
    ],
)
```

### 保护特定文件

```python
agent = create_deep_agent(
    model=model,
    backend=backend,
    permissions=[
        FilesystemPermission(
            operations=["read", "write"],
            paths=["/workspace/.env", "/workspace/examples/**"],
            mode="deny",
        ),
        FilesystemPermission(
            operations=["read", "write"],
            paths=["/workspace/**"],
            mode="allow",
        ),
        FilesystemPermission(
            operations=["read", "write"],
            paths=["/**"],
            mode="deny",
        ),
    ],
)
```

### 只读 memory

允许 agent 读 memory 文件但不能改。适合**组织级策略**或**共享知识库**——只能被应用代码更新。

```python
from deepagents.backends import CompositeBackend, StateBackend, StoreBackend

agent = create_deep_agent(
    model=model,
    backend=CompositeBackend(
        default=StateBackend(),
        routes={
            "/memories/": StoreBackend(
                namespace=lambda rt: (rt.server_info.user.identity,),
            ),
            "/policies/": StoreBackend(
                namespace=lambda rt: (rt.context.org_id,),
            ),
        },
    ),
    permissions=[
        FilesystemPermission(
            operations=["write"],
            paths=["/memories/**", "/policies/**"],
            mode="deny",
        ),
    ],
)
```

### 拒绝所有访问

屏蔽所有读写。作为**严格 baseline**，你可以在上面叠更具体的 allow 规则：

```python
agent = create_deep_agent(
    model=model,
    backend=backend,
    permissions=[
        FilesystemPermission(
            operations=["read", "write"],
            paths=["/**"],
            mode="deny",
        ),
    ],
)
```

### 规则顺序

因为是**首匹配胜出**，规则顺序很重要。**把更具体的规则放在更广的之前**：

```python
# ✅ 正确：先 deny .env，再 allow workspace，再 deny 其他
correct_permissions = [
    FilesystemPermission(
        operations=["read", "write"],
        paths=["/workspace/.env"],
        mode="deny",
    ),
    FilesystemPermission(
        operations=["read", "write"],
        paths=["/workspace/**"],
        mode="allow",
    ),
    FilesystemPermission(
        operations=["read", "write"],
        paths=["/**"],
        mode="deny",
    ),
]

# ❌ Bug：/workspace/** 先匹配到 .env，导致 deny 永远不触发
incorrect_permissions = [
    FilesystemPermission(
        operations=["read", "write"],
        paths=["/workspace/**"],
        mode="allow",
    ),
    FilesystemPermission(
        operations=["read", "write"],
        paths=["/workspace/.env"],
        mode="deny",  # 永远不会到达
    ),
    FilesystemPermission(
        operations=["read", "write"],
        paths=["/**"],
        mode="deny",
    ),
]
```

## Subagent 权限

Subagent **默认继承父 agent 权限**。要给 subagent 不同权限，在它的 spec 里设 `permissions` 字段——这会**整体替换**父规则。

```python
agent = create_deep_agent(
    model=model,
    backend=backend,
    permissions=[
        FilesystemPermission(operations=["read", "write"], paths=["/workspace/**"], mode="allow"),
        FilesystemPermission(operations=["read", "write"], paths=["/**"], mode="deny"),
    ],
    subagents=[
        {
            "name": "auditor",
            "description": "Read-only code reviewer",
            "system_prompt": "Review the code for issues.",
            "permissions": [
                FilesystemPermission(operations=["write"], paths=["/**"], mode="deny"),
                FilesystemPermission(operations=["read"], paths=["/workspace/**"], mode="allow"),
                FilesystemPermission(operations=["read"], paths=["/**"], mode="deny"),
            ],
        }
    ],
)
```

## Composite Backend

用带 sandbox 默认的 `CompositeBackend` 时，**每个权限路径必须在某个已知路由前缀下**。Sandbox 支持任意命令执行，所以**基于路径的限制不能防止通过 shell 命令访问文件系统**。把权限作用域限制到具体路由 backend 避免这个冲突。

```python
from deepagents.backends import CompositeBackend

composite = CompositeBackend(
    default=sandbox,
    routes={"/memories/": memories_backend},
)

# ✅ 可以：权限作用域在 /memories/ 路由
agent = create_deep_agent(
    model=model,
    backend=composite,
    permissions=[
        FilesystemPermission(operations=["write"], paths=["/memories/**"], mode="deny"),
    ],
)
```

包含**任何路由之外**路径的权限会抛 `NotImplementedError`：

```python
# ❌ 抛 NotImplementedError：/workspace/** 落到 sandbox 默认
try:
    create_deep_agent(
        model=model,
        backend=composite,
        permissions=[
            FilesystemPermission(operations=["write"], paths=["/workspace/**"], mode="deny"),
        ],
    )
except NotImplementedError:
    pass

# ❌ 也抛：/** 覆盖了所有路由和默认
try:
    create_deep_agent(
        model=model,
        backend=composite,
        permissions=[
            FilesystemPermission(operations=["read"], paths=["/**"], mode="deny"),
        ],
    )
except NotImplementedError:
    pass
```
