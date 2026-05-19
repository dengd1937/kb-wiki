# 2024–25 能力跃迁：Computer Use / Coding Agent（综述）

> 性质: 综合综述条目（无单一来源，下列为各节点权威出处）
> Collected: 2026-05-18
> 时间跨度: 2023-10 ～ 2025（锚点 2024-03-12 Devin 发布，能力跃迁标志）

## 来源（各节点）

- SWE-bench — arXiv 2310.06770（见单独条目）
- SWE-agent — arXiv 2405.15793（见单独条目）
- Devin — VentureBeat「Cognition emerges from stealth to launch AI software engineer Devin」；devin.ai
- Anthropic Computer Use — https://www.anthropic.com/news/3-5-models-and-computer-use ；simonwillison.net/2024/Oct/22/computer-use/
- OpenAI Operator — https://openai.com/index/introducing-operator/ ；techcrunch 2025-01-23

## 背景

框架浪潮（2023 H2–2024）解决了"如何编排"后，2024 年 Agent 发生**能力质变**：不再只是"调 API + 编排循环"，而是获得两类新本事——**端到端写代码**与**直接操作计算机（GUI/浏览器）**。这是 Function Calling 铰链之后、MCP 标准化之前的关键阶段。

## 两条子线与时间线（已交叉核对）

### A. Coding Agent

| 时间 | 节点 | 主体 | 意义 |
|---|---|---|---|
| 2023-10-10 | **SWE-bench** | 普林斯顿（Jimenez 等） | 真实 GitHub issue 评测基准，定义标尺；初始 SOTA ≈ 1.96% |
| 2024-03-12 | **Devin** | Cognition Labs | 宣称"首个全自主 AI 软件工程师"，引爆产品认知 |
| 2024-05-06 | **SWE-agent** | 普林斯顿（同组） | 提出 ACI，SWE-bench 升至 12.5% pass@1 |
| 2025+ | Claude Code / OpenAI Codex 等 | Anthropic / OpenAI | 生产级 Coding Agent，衔接 Harness |

### B. Computer Use（操作计算机）

| 时间 | 节点 | 主体 | 意义 |
|---|---|---|---|
| 2024-10-22 | **Anthropic Computer Use** | Anthropic（Claude 3.5 Sonnet） | 首个通用计算机操作前沿模型，公测：看屏幕、移光标、点击、输入 |
| 2025-01-23 | **OpenAI Operator** | OpenAI（CUA 模型） | 浏览器自主操作 agent，GPT-4o 视觉 + RL |

## 主线脉络

1. **标尺先行**（SWE-bench, 2023.10）：用真实 bug 修复定义"能不能干活"。
2. **产品引爆**（Devin, 2024.03）：把 Coding Agent 推入大众视野。
3. **方法突破**（SWE-agent, 2024.05）：ACI——为 agent 而非人设计接口，成功率数量级提升。
4. **跨出代码**（Computer Use 2024.10 / Operator 2025.01）：从"写代码"扩展到"操作任意 GUI"。

## 意义与下游

能力跃迁让 Agent 真正"能自主交付"，但也使**工具/上下文接入的碎片化**变得不可忍受（每个 agent 一套接口胶水）——直接催生 **MCP（2024-11）** 的标准化，再到 **《Building Effective Agents》（2024-12）** 的设计模式收敛，最终汇入 **Harness Engineering（2026）**。SWE-agent 的 ACI 思想（为智能体设计环境）正是 Harness 的直接前驱。
