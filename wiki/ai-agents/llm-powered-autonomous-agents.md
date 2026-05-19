# LLM Powered Autonomous Agents：Lilian Weng 的 Agent 经典定义

> Sources: Lilian Weng, Lil'Log 博客，2023-06-23
> Raw: [LLM Powered Autonomous Agents（原文要点/中文）](../../raw/ai-agents/2023-06-23-llm-powered-autonomous-agents.md)

## 概述

Lilian Weng（发布时任 OpenAI Safety/Applied AI 团队负责人）于 2023 年 6 月发表的综述博客，提出被引用最广的 LLM Agent 心智模型：**LLM 作为智能体的"大脑/控制器"**，由 Planning、Memory、Tool use 三大组件补充。该文把当时分散的 CoT、ToT、ReAct、Reflexion、Toolformer、HuggingGPT 等统一进同一张蓝图，是 2023 年承上启下的"定义之作"。

## 核心公式

**Agent = LLM（Brain / Controller） + Planning + Memory + Tool use**

## 三大组件

### 1. Planning（规划）

- **任务分解**：CoT（think step by step）、ToT（每步多分支的树状搜索）、LLM+P（外接经典规划器）。
- **自我反思**：ReAct（推理+行动交错的 Thought/Action/Observation）、Reflexion（动态记忆+自我反思）、Chain of Hindsight（从反馈改进）。

### 2. Memory（记忆）

以人类记忆类比：

| 人类记忆 | 智能体对应 |
|---|---|
| 感觉记忆 | 原始输入的 embedding 表示 |
| 短期记忆 | 上下文内学习，受上下文窗口长度限制 |
| 长期记忆 | 外部向量库，快速检索访问 |

长期记忆靠**最大内积搜索（MIPS）** + 近似最近邻（LSH/ANNOY/HNSW/FAISS/ScaNN）。

### 3. Tool use（工具使用）

调用外部 API 获取权重中没有的能力（实时信息、代码执行、专有数据）。代表：MRKL（路由到专家模块）、TALM/Toolformer（微调学工具）、HuggingGPT（ChatGPT 调度模型）、API-Bank（评测基准）。

## 公开挑战（原文）

有限上下文长度、长期规划与任务分解困难、自然语言接口可靠性。

## 在本库中的定位

这是 `ai-agents` 分类的**框架锚点**条目：它是 ReAct（Planning 的推理-行动）、Reflexion（Planning 自我反思 + Memory）、Toolformer（Tool use 训练时路线）的统一上层框架。演进线：单点技术（ReAct/Toolformer/Reflexion）→ **统一框架（本文）** → 工程化范式（Harness Engineering）。

## 相关条目

Planning → Task Decomposition：
- [Chain-of-Thought](chain-of-thought.md) — 思维链，任务分解的基础。
- [Tree of Thoughts](tree-of-thoughts.md) — CoT 的树状搜索扩展。
- [LLM+P](llm-p.md) — 外接经典规划器的神经-符号路线。

Planning → Self-Reflection：
- [ReAct: Synergizing Reasoning and Acting in Language Models](react-reasoning-acting.md) — 推理-行动交错循环。
- [Reflexion: Language Agents with Verbal Reinforcement Learning](reflexion.md) — 自我反思 + 记忆的代表。
- [Chain of Hindsight](chain-of-hindsight.md) — 用反馈序列学习改进。
- [Algorithm Distillation](algorithm-distillation.md) — 历史轨迹驱动的上下文内自我改进。

Tool use：
- [Toolformer: Language Models Can Teach Themselves to Use Tools](toolformer.md) — Tool use 的训练时路线。
- [OpenAI Function Calling](openai-function-calling.md) — Tool use 的工程落地（原生结构化调用）。

延伸：
- [Harness Engineering: Building Software with AI Agents](../ai-engineering/harness-engineering.md) — 该框架在工程化阶段的当代延伸。
