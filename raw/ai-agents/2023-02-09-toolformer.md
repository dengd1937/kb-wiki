# Toolformer: Language Models Can Teach Themselves to Use Tools

> Source: https://arxiv.org/abs/2302.04761
> Collected: 2026-05-18
> Published: 2023-02-09（arXiv v1）；NeurIPS 2023 接收

## 元数据

- **标题**：Toolformer: Language Models Can Teach Themselves to Use Tools
- **作者**：Timo Schick、Jane Dwivedi-Yu、Roberto Dessì、Roberta Raileanu、Maria Lomeli、Luke Zettlemoyer、Nicola Cancedda、Thomas Scialom
- **机构**：Meta AI（FAIR）
- **arXiv 编号**：2302.04761
- **发布**：2023-02-09（v1）
- **会议**：NeurIPS 2023

## 摘要（原文）

Language models (LMs) exhibit remarkable abilities to solve new tasks from just a few examples or textual instructions, especially at scale. They also, paradoxically, struggle with basic functionality, such as arithmetic or factual lookup, where much simpler and smaller models excel. In this paper, we show that LMs can teach themselves to use external tools via simple APIs and achieve the best of both worlds. We introduce Toolformer, a model trained to decide which APIs to call, when to call them, what arguments to pass, and how to best incorporate the results into future token prediction. This is done in a self-supervised way, requiring nothing more than a handful of demonstrations for each API. We incorporate a range of tools, including a calculator, a Q&A system, two different search engines, a translation system, and a calendar. Toolformer achieves substantially improved zero-shot performance across a variety of downstream tasks, often competitive with much larger models, without sacrificing its core language modeling abilities.

## 摘要（中文翻译）

语言模型（LM）展现出仅凭少量示例或文本指令就能解决新任务的卓越能力，尤其在大规模时。但矛盾的是，它们在一些基础功能上表现糟糕——如算术或事实查询，而更简单更小的模型却能胜任。本文表明：LM 可以**自学**通过简单 API 使用外部工具，从而兼得两者之长。我们提出 **Toolformer**——一个被训练来决定"调用哪个 API、何时调用、传什么参数、如何把结果最好地融入后续 token 预测"的模型。这一切以**自监督**方式完成，每个 API 只需少量演示。我们接入了一系列工具：计算器、问答系统、两个搜索引擎、翻译系统和日历。Toolformer 在多种下游任务上显著提升零样本性能，常可与大得多的模型竞争，且不牺牲其核心语言建模能力。

## 核心思想

与 ReAct"在推理时用提示引导工具调用"不同，Toolformer 把工具使用能力**烘焙进模型权重**：

1. **自监督数据构造**：用少量示例提示让 LM 自己在文本中插入候选 API 调用。
2. **过滤**：保留那些"调用结果能降低后续 token 预测损失"的 API 调用——即只有真正有用的调用才被采纳。
3. **微调**：在这份自生成、自过滤的数据上微调模型，使其内化"何时/如何调用工具"。

## 与 ReAct 的关系（路线之争）

- **ReAct**：推理时（inference-time）通过 prompting 引导，不改权重，灵活但依赖提示工程。
- **Toolformer**：训练时（training-time）把能力写入权重，调用更自然、无需复杂提示，但接入新工具需重新构造数据微调。

二者代表 Agent 工具使用的两条互补技术路线。

## 接入的工具

计算器、问答系统、两个搜索引擎、翻译系统、日历。

## 意义

证明了"模型可自学工具使用"，是 LLM 工具调用从纯提示走向**原生能力**的关键一步，思想上预示了后续 OpenAI Function Calling 等原生工具调用机制的方向。
