# Chain-of-Thought Prompting Elicits Reasoning in Large Language Models

> Source: https://arxiv.org/abs/2201.11903
> Collected: 2026-05-18
> Published: 2022-01-28（arXiv v1；v6 2023-01-10）

## 元数据

- **标题**：Chain-of-Thought Prompting Elicits Reasoning in Large Language Models
- **作者**：Jason Wei、Xuezhi Wang、Dale Schuurmans、Maarten Bosma、Brian Ichter、Fei Xia、Ed Chi、Quoc Le、Denny Zhou
- **机构**：Google Research, Brain Team（业界公认；arXiv 页面未直接列出）
- **arXiv 编号**：2201.11903
- **版本历史**：v1 2022-01-28；…；v6 2023-01-10
- **会议**：NeurIPS 2022（业界公认；arXiv 页面未直接注明）

## 摘要（据 arXiv 整理）

研究表明：生成一条"思维链"（chain of thought，一系列中间推理步骤）能显著提升大语言模型执行复杂推理的能力。论文提出 **chain-of-thought prompting**——只需在少样本示例中提供"带推理过程的演示"即可触发该能力，无需微调。在三个模型上的实验显示，它在算术、常识和符号推理任务上均带来提升；一个 540B 参数模型借此在 GSM8K 数学基准上取得 SOTA。

## 核心思想

在 few-shot 提示的示例中，把"问题 → 答案"替换为"问题 → 中间推理步骤 → 答案"，引导模型在输出答案前显式生成推理过程。这是一种**纯推理时**、零训练成本的能力激发方法，且推理能力随模型规模涌现。

## 在 Lilian Weng 框架中的位置

属 **Planning → Task Decomposition** 的基础方法：用"think step by step"把难任务拆成可处理的中间步骤。是 ToT、ReAct 等后续工作的共同起点。

## 意义

CoT 是 LLM 推理与 Agent 规划能力的奠基性提示范式，催生了 Zero-shot-CoT、Self-Consistency、ToT 等一系列后续工作，是 Agent "会思考再行动"的源头。
