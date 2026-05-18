# Toolformer：让语言模型自学使用工具

> Sources: arXiv 2302.04761（Schick et al., Meta AI），2023-02-09；NeurIPS 2023
> Raw: [Toolformer（原文/中文）](../../raw/ai-agents/2023-02-09-toolformer.md)

## 概述

Toolformer 由 Meta AI 于 2023 年 2 月提出。语言模型擅长少样本解题，却在算术、事实查询等基础功能上不如更小的专用模型。Toolformer 证明：模型可以**自监督地自学**通过简单 API 使用外部工具——自己决定调用哪个 API、何时调用、传什么参数、如何融入后续生成。仅需每个 API 少量演示，即可在多种下游任务上大幅提升零样本性能，常能与大得多的模型竞争，且不损失核心语言建模能力。

## 核心机制：自监督的工具能力内化

1. **自生成**：用少量示例提示，让 LM 在文本中插入候选 API 调用。
2. **自过滤**：只保留"调用结果能降低后续 token 预测损失"的有用调用。
3. **微调**：在自生成、自过滤的数据上微调，把工具使用写入权重。

接入工具：计算器、问答系统、两个搜索引擎、翻译系统、日历。

## 与 ReAct 的路线之争

| | ReAct | Toolformer |
|---|---|---|
| 时机 | 推理时（prompting 引导） | 训练时（写入权重） |
| 是否改权重 | 否 | 是（自监督微调） |
| 优势 | 灵活、零训练成本 | 调用自然、无需复杂提示 |
| 代价 | 依赖提示工程 | 接新工具需重构数据微调 |

二者是 Agent 工具使用的两条互补技术路线。

## 对后续 Agent 发展的意义

### 确立的范式（正面遗产）

- **把"工具使用"定义为可学习的模型能力，而非提示技巧。** 在此之前工具调用要么硬编码、要么靠 ReAct 式 prompting 临场引导；Toolformer 首次证明"何时调用 / 调哪个 / 传什么参 / 如何用结果"可被烘焙进权重，直接预示了 2023.06 OpenAI Function Calling 及 Anthropic/Google 跟进的**原生函数调用**——让工具调用从脆弱文本解析变为结构化原生能力，这是 Agent 从 demo 走向生产的分水岭。
- **自监督自举的数据范式。** "模型自生成候选调用 → 用'是否降低后续 token 预测损失'过滤 → 在自产数据上微调"这套用效用信号自动构造训练数据、无需人工标注的配方，成为后续工具学习的通用思路，影响了 Gorilla、ToolLLM/ToolBench、NexusRaven 等函数调用专用模型/数据集。
- **与 ReAct 形成互补双路线。** 今天主流方案是二者融合：原生 function calling 提供可靠调用能力（Toolformer 路线的工程化身），ReAct-style loop 提供多步编排。

### 局限催生的后续方向（反向推动）

| Toolformer 的局限 | 催生的后续方向 |
|---|---|
| 工具集静态，接新工具需重构数据再微调 | 动态工具发现、检索式工具选择；最终走向 **MCP** 标准化接入 |
| 偏单次调用，缺调用间依赖与编排 | ReAct/Reflexion 多步循环、工具链组合（ToolLLM 工具图） |
| 调用决策与高层推理解耦不足、不会从失败恢复 | Reflexion 的语言反馈 + 记忆、自我纠错 |
| 工具规模有限（计算器/搜索/翻译等少数几个） | 大规模工具生态（ToolBench 16k+ API）、协议化连接 |

### 一句话定位

Toolformer 是"工具使用作为模型原生能力"这条线的奠基论文：它把 ReAct 在提示层做的事下沉到权重层，思想上直通后来的 Function Calling；其"静态、单次、弱编排"的局限又分别被 MCP（动态接入）、Reflexion（自我纠错）、ToolLLM（大规模编排）接力补全。在本库演进线中，它是 **ReAct → Function Calling → MCP** 这条"工具可靠性"主轴的关键中间环。

## 相关条目

- [LLM Powered Autonomous Agents](llm-powered-autonomous-agents.md) — Lilian Weng 的统一框架，Toolformer 属其 Tool use 组件的训练时路线。
- [ReAct: Synergizing Reasoning and Acting in Language Models](react-reasoning-acting.md) — 工具使用的"推理时"对照路线。
- [Reflexion: Language Agents with Verbal Reinforcement Learning](reflexion.md) — 同期 Agent 自我改进方向。
