# Interpreters（解释器）

> Source: https://docs.langchain.com/oss/python/deepagents/interpreters
> Collected: 2026-05-24

在 Deep Agents 里运行**轻量代码**——组合工具、编排 subagent、变换结构化数据。

Interpreter 给 agent 一个**可编程工作区**：可以探索数据、协调 tool call、把中间工作隔在模型 context 之外。Agent 写代码表达意图，**内存 runtime** 执行代码、返回相关结果。

> **vs Sandbox**：[sandbox](sandboxes.md) 是"以代码为中心的方式作用于**环境**"（运行命令、装依赖、编辑文件）；interpreter 是"以代码为中心的方式作用于 **agent 循环内部**"（组合工具、保留状态、决定哪些信息回到模型）。

> ⚠️ Interpreter 是**实验性的**，API 和生命周期行为可能在版本间变化。
>
> 需要：`langchain-quickjs>=0.1.0` + Python `>=3.11`。

## 何时用 interpreter

大多数 agent 工作在**模型推理**和**工具执行**之间交替。简单动作没问题，但 agent 需要组合多步骤、推理结构化数据、管理中间状态时就别扭了。

Interpreter 给 agent 提供这种工作的 runtime。**不再让模型一次只选一个 tool call**，agent 可以写一个小程序，运行控制流、调用允许的工具、存变量、给模型返回紧凑结果。

用 interpreter 的场景：

- 用**代码组合多次 tool call**——含循环、分支、重试、并发
- **从代码协调 subagent**——把工作切成专注调用，存结果，拼成最终综合
- 把**中间值留在 runtime state**——而非每个临时结果都回到模型 context
- **确定性变换结构化数据**——排序、分组、解析、验证、打分、聚合
- 探索大变量空间，**只把选定的证据、摘要或输出**返给模型

```
Model → 写小程序 → Interpreter runtime → 调工具 + 更新变量
                                       → 产出紧凑结果 → Model
```

底层是 [**QuickJS**](https://github.com/quickjs-ng/quickjs)——为嵌入式执行设计的轻量 JS runtime。Runtime 给 agent 一个评估代码的地方，**默认不暴露宿主文件系统、网络、shell、包、时钟 API**。

QuickJS 是 interpreter 代码的执行边界。显式桥接（如**程序化工具调用 PTC**）决定代码能访问什么能力。

## 选对执行路径

| 需求 | 用 |
|------|----|
| 1-2 个简单外部调用 | 普通 tool calling |
| 一段循环/分支/重试/聚合的小程序 | **Interpreter** |
| 多个选定的、应从代码调的工具 | **Interpreter + PTC** |
| 跨线程复用的辅助函数 | **Interpreter + interpreter skills** |
| Shell 命令、装包、跑测试、完整 OS 文件系统 | [Sandbox](sandboxes.md) |

## 添加 interpreter

```bash
pip install -U "deepagents[quickjs]"
```

```python
from deepagents import create_deep_agent
from langchain_quickjs import CodeInterpreterMiddleware

agent = create_deep_agent(
    model="openai:gpt-5.4",
    middleware=[CodeInterpreterMiddleware()],
)
```

## 在 interpreter 里跑代码

Middleware 给 agent 加一个 `eval` 工具。它**在持久 context 里**运行 TypeScript、捕获 `console.log`、返回最后表达式的结果。

Agent 可以这样写：

```javascript
const rows = [
  { team: "alpha", score: 8 },
  { team: "beta", score: 13 },
  { team: "alpha", score: 21 },
];

const totals = rows.reduce((acc, row) => {
  acc[row.team] = (acc[row.team] ?? 0) + row.score;
  console.log(`${row.team} score: ${acc[row.team]}`)
  return acc;
}, {});

totals;
```

**默认**：interpreter 状态在同一 thread 内跨 turn 持久化——每次 agent run 后快照工作状态、下次 run 前恢复。

## 程序化工具调用（Programmatic Tool Calling, PTC）

PTC 把选定的 agent 工具暴露在 interpreter 的 `tools` 全局命名空间下。**不再让模型一次发一个 tool call、等结果、决定下一个**，agent 可以写代码**在循环/分支/重试/并行批处理里调工具**。

适合中间工具结果只是**下一步输入**的场景。Interpreter 可以处理、过滤、聚合那些结果，之后才把任何东西返回模型 context——这能让多工具/多步骤工作流**更省 token**。

PTC 在 Deep Agents 里**模型无关**——由 middleware 实现，不是 provider 特定的代码执行/工具调用 API。

### 工作原理

1. 你用 `ptc` allowlist 选哪些工具 interpreter 能调
2. Middleware 把这些工具暴露为 `tools` 下的**异步 JavaScript 函数**
3. Agent 写 interpreter 代码用 `await` 调这些函数
4. Interpreter 运行工具桥接，收到结果，继续执行
5. 模型收到**最终 interpreter 输出**，而非每个中间值

每个 allowlisted 工具变成 async function。**工具名转为 camel case**，但输入对象仍跟工具的 schema。如 `web_search` 工具变成 `tools.webSearch(...)`：

```typescript
const result: string = await tools.webSearch({
  query: "deepagents interpreters",
});
```

### 有用的模式

| 模式 | Interpreter 能做什么 |
|------|---------------------|
| 批处理 | 循环多输入，每个调一次工具 |
| 并行 | `Promise.all` 处理独立调用 |
| 条件逻辑 | 基于前面结果选下个工具调用 |
| 提前终止 | 成功条件满足就停 |
| 数据过滤 | 只把相关行/片段/错误/摘要返给模型 |
| 递归编排 | 反复调 `task`，从代码组合 subagent 结果 |

### 启用 PTC

```python
from deepagents import create_deep_agent
from langchain_quickjs import CodeInterpreterMiddleware

agent = create_deep_agent(
    model="openai:gpt-5.4",
    middleware=[CodeInterpreterMiddleware(ptc=["task"])],
)
```

PTC 启用后，agent 可以从 interpreter 代码调 allowlisted 工具。这个例子并行启动多个 subagent，组合它们的最终报告：

```javascript
const topics = ["retrieval", "memory", "evaluation"];

const reports = await Promise.all(
  topics.map((topic) =>
    tools.task({
      description: `Research ${topic} in Deep Agents and return three concise findings.`,
      subagent_type: "general-purpose",
    }),
  ),
);

reports.join("\n\n");
```

因为是代码，agent 也可以**本地处理失败**：

```javascript
try {
  const report = await tools.task({
    description: "Check the migration notes and return breaking changes.",
    subagent_type: "general-purpose",
  });
  console.log(report);
} catch (error) {
  console.log(`Subagent failed: ${error.message}`);
}
```

> ⚠️ **PTC 调用走 interpreter 桥接**，**不**走正常 tool calling 路径。所以 `interrupt_on` 审批工作流**不会在每个 PTC 调的工具上强制执行**。

## 递归语言模型（Recursive Language Models, RLM）

递归语言模型把 interpreter 用作**分解工作区**。模型把大输入或工作集留在 runtime 变量里，写代码检查和切分，对小块调 subagent 或其他模型工具，然后用代码拼装回来。

这把**变量空间和 agent context 分开**：变量空间是 interpreter 里存的信息，agent context 是模型实际在下次调用中处理的。模型可以决定哪些片段成为 subagent 任务、哪些结果需要再过一遍、最终综合应该返回到主对话中。

背景论文见 [Recursive Language Models paper](https://arxiv.org/abs/2512.24601)。

在 Deep Agents 里，递归调用常常是通过 PTC 暴露的 `task` 工具。Interpreter 可以对多个切片调 subagent，组合答案，返回单一综合结果：

```javascript
const candidates = notes
  .filter((note) => note.includes("migration"))
  .slice(0, 5);

const riskReports = await Promise.all(
  candidates.map((note) =>
    tools.task({
      description: `Analyze this migration note for release risk. Return risks, affected users, and recommended follow-up:\n\n${note}`,
      subagent_type: "general-purpose",
    }),
  ),
);

const releaseSummary = riskReports
  .map((report, index) => `## Candidate ${index + 1}\n${report}`)
  .join("\n\n");

releaseSummary;
```

## Interpreter Skills

Interpreter skill 是把**代码模块**暴露给 interpreter 的 [skill](https://docs.langchain.com/oss/python/deepagents/skills)。配上 interpreter middleware，agent 可以从代码 import 这些模块用作**确定性辅助逻辑**。

适合 agent 需要可复用辅助函数处理结构化数据流的场景（排序、分组、打分、解析、验证、聚合）。

## Snapshot 与 Time Travel

`CodeInterpreterMiddleware` 默认在每次 agent run 后快照 interpreter 状态、下次 run 前恢复。Snapshot 是 interpreter 内存 JS 状态的**序列化副本**——agent 完成跑代码时的全局变量、变量、函数、import 模块。

跨对话 turn 的生命周期：

1. Turn 开始，middleware **恢复** thread 的最新 interpreter snapshot
2. Agent 调 `eval`，代码可以读或改 interpreter 变量
3. Agent run 结束，middleware **把更新的 interpreter state 快照到 graph state**
4. 下个 turn 从那个恢复的 interpreter state 开始，而非空 runtime

> 单次 agent run 内的多次 `eval` 调用用 live interpreter context 对象；middleware 不在它们之间快照恢复，只在 run 完成时快照。

⚠️ 跨对话 turn 的 snapshot 只**保留能合理序列化的值**。用它们存数据，**不要存活 runtime 对象**。Function、class、其他不可序列化的值恢复后是不可访问的 artifact。Interpreter 代码恢复后访问它们，eval 工具会抛错：`Value for 'fn' was not restored because it is not serializable (type: function).`

Snapshot **保留 interpreter 内存，不保留外部世界副作用**。如果 interpreter 代码通过 PTC 调了工具，恢复之前的 interpreter snapshot **不会撤销那个 tool call 的副作用**——只恢复记录或处理结果的 interpreter 变量。

```python
from deepagents import create_deep_agent
from langchain_quickjs import CodeInterpreterMiddleware
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()

agent = create_deep_agent(
    model="openai:gpt-5.4",
    checkpointer=checkpointer,
    middleware=[
        CodeInterpreterMiddleware(
            snapshot_between_turns=True,  # 默认
        )
    ],
)
```

可以用 `snapshot_between_turns=False` 禁用跨 turn 快照。

## 安全与限制

Interpreter 用 QuickJS 跑不可信 JS，默认严格隔离。把它当作**作用域受限的 interpreter runtime**，而非**完整的生产 sandbox backend**。

PTC 暴露的每个工具都是 interpreter 代码可用的**外部能力**。**把 PTC allowlist 当作权限边界**：只暴露 agent 需要的工具，避免桥接可访问敏感系统、花钱、突变数据、调不受限网络的广泛工具——除非那是有意行为。

| 能力 | 默认可用 | 如何暴露 |
|------|---------|---------|
| JavaScript 执行 | ✓ | 加 interpreter middleware |
| 顶层 `await` | ✓ | 在 interpreter 代码用 Promise |
| `console.log` 捕获 | ✓ | `capture_console=False` 禁用 |
| Agent 工具 | ✗ | 加 PTC allowlist |
| Interpreter skill 模块 | ✗ | 加 `module` 入口 + 配 `skills_backend` |
| 文件系统访问 | ✗ | 通过 PTC allowlist 加内置文件系统工具 |
| 网络访问 | ✗ | 通过 PTC 暴露特定网络工具 |
| 时钟/日期 | ✗ | 需要时暴露显式时间工具 |
| Shell / 包安装 / 测试 / OS 执行 | ✗ | 用 [sandbox backend](sandboxes.md) |

> Interpreter 代码在嵌入的 QuickJS context 运行，**不是独立 VM 或进程**。Python 里这个 runtime 由 [`quickjs-rs`](https://github.com/langchain-ai/quickjs-rs) 提供，安全文档同样说明同进程执行边界。
>
> 把 interpreter 当作**能力作用域的执行层**，而非**宿主内存隔离边界**。不可信或半可信代码，要在隔离 worker 进程或容器里跑 agent，**保持 PTC allowlist 窄**。

## Middleware 选项

`CodeInterpreterMiddleware` 接受以下选项：

| 参数 | 默认 | 用途 |
|------|------|------|
| `memory_limit` | `64 * 1024 * 1024`（64 MB）| QuickJS 堆内存上限（字节）|
| `timeout` | `5.0` | 每次 eval 超时（秒）|
| `max_ptc_calls` | `256` | 每次 eval 最大 `tools.*` 调用数；`None` 仅在可信环境用 |
| `tool_name` | `"eval"` | 暴露给模型的 interpreter 工具名 |
| `max_result_chars` | `4000` | 结果和 stdout 块的最大字符数 |
| `capture_console` | `True` | 是否捕获 `console.log/warn/error` |
| `ptc` | `None` | PTC allowlist：工具名列表或 `BaseTool` 实例 |
| `skills_backend` | `None` | 解析 interpreter skill 模块的 backend |
| `snapshot_between_turns` | `True` | 是否跨 agent turn 持久化 interpreter state 快照 |
| `max_snapshot_bytes` | `None` | 最大序列化 snapshot 大小，默认 `memory_limit` |
