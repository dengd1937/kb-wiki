# Human-in-the-loop（人机协同）

> Source: https://docs.langchain.com/oss/python/deepagents/human-in-the-loop
> Collected: 2026-05-24

某些工具操作可能敏感、需要执行前的人工审批。Deep Agents 用 LangGraph 的 interrupt 能力实现这一点。

## 核心概念

通过 `interrupt_on` 参数为每个工具配置审批行为。

`interrupt_on` 接受三种值：

- `True`——默认行为（所有 decision 类型都允许）
- `False`——禁用中断
- **dict**——用 `"allowed_decisions"` 列表做精细控制

## 4 种 Decision 类型

| Decision | 说明 |
|----------|------|
| **`approve`** | 用**原始参数**执行工具 |
| **`edit`** | **修改参数后**执行工具 |
| **`reject`** | **跳过**工具执行 |
| **`respond`** | 返回**人工消息**作为工具结果（适合 user-query 类工具）|

## 基础示例

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


# ⚠️ Checkpointer 对 HITL 是必需的
checkpointer = MemorySaver()

agent = create_deep_agent(
    model="google_genai:gemini-3.5-flash",
    tools=[remove_file, fetch_file, notify_email],
    interrupt_on={
        "remove_file": True,                                            # 默认：4 种 decision 都允许
        "fetch_file": False,                                            # 不需要中断
        "notify_email": {"allowed_decisions": ["approve", "reject"]},  # 不允许 edit/respond
    },
    checkpointer=checkpointer,
)
```

## 关键要求

### 1. Checkpointer 必需

HITL 工作流**强制要求 checkpointer**，用来在中断和恢复周期之间**持久化 agent 状态**。

### 2. Thread ID 一致性

恢复执行时，必须用**相同 config + 相同 `thread_id`**。

```python
config = {"configurable": {"thread_id": "user-123-session-456"}}

result = agent.invoke({"messages": [...]}, config=config)

# 如果有中断：
if result.get("interrupts"):
    decisions = collect_human_decisions(result["interrupts"])
    # 用相同 config 恢复
    result = agent.invoke(
        Command(resume={"decisions": decisions}),
        config=config,  # ← 同样的 config
    )
```

### 3. Decision 顺序匹配

`decisions` 列表**必须按顺序对应** `action_requests`。

## 中断处理模式

```python
from langgraph.types import Command

result = agent.invoke({"messages": [...]}, config=config)

# 检查是否有中断
if result.get("interrupts"):
    action_requests = result["interrupts"][0]["action_requests"]
    
    # 展示给用户、收集决定（按 action_requests 顺序）
    decisions = [
        {"type": "approve"},
        {"type": "edit", "args": {"to": "fixed@example.com", "subject": "...", "body": "..."}},
        {"type": "reject"},
    ]
    
    # 恢复执行
    result = agent.invoke(
        Command(resume={"decisions": decisions}),
        config=config,
    )
```

## 高级特性

### 批量审批

多个 tool 调用可以**批处理**到单个 interrupt——用户一次审批多个操作。

### 编辑工具参数

允许 `edit` 时，用户可以在执行前**修改参数**：

```python
{
    "type": "edit",
    "args": {"path": "/safe/path/file.txt"},  # 改了原本危险的 path
}
```

### Subagent 特定中断

Subagent 可以**覆盖父 agent 的 interrupt 配置**：

```python
agent = create_deep_agent(
    model=model,
    interrupt_on={"remove_file": True},
    subagents=[
        {
            "name": "auditor",
            "system_prompt": "...",
            "interrupt_on": {
                "remove_file": False,  # auditor 不需要审批就能删
            },
        }
    ],
    checkpointer=checkpointer,
)
```

### 在 subagent 工具内部用 `interrupt()`

可以在工具实现**内部**直接调用 `interrupt()` 原语请求审批：

```python
from langgraph.types import interrupt

@tool
def dangerous_action(target: str) -> str:
    """Do something risky."""
    # 工具内部主动请求审批
    decision = interrupt({
        "type": "confirm_action",
        "target": target,
        "message": f"Really do this to {target}?",
    })
    
    if decision.get("approved"):
        return f"Done: {target}"
    return "Cancelled"
```

## 最佳实践（按风险分级）

| 风险等级 | 推荐配置 |
|---------|---------|
| **高风险**（删除、汇款、发邮件） | `True`——允许全部 4 种 decision |
| **中风险**（修改文件、调外部 API）| `{"allowed_decisions": ["approve", "reject"]}` |
| **低风险**（读文件、搜索）| `False`——跳过中断 |
