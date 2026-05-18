# ReAct: Synergizing Reasoning and Acting in Language Models

> Source: https://arxiv.org/abs/2210.03629 ; 项目页 https://react-lm.github.io/
> Collected: 2026-05-18
> Published: 2022-10-06（arXiv v1；v3 2023-03-10 为 ICLR 2023 camera-ready）

## 元数据

- **标题**：ReAct: Synergizing Reasoning and Acting in Language Models
- **作者**：Shunyu Yao、Jeffrey Zhao、Dian Yu、Nan Du、Izhak Shafran、Karthik Narasimhan、Yuan Cao
- **机构**：普林斯顿大学（Shunyu Yao、Karthik Narasimhan）与 Google Research / Brain 团队（其余作者）
- **arXiv 编号**：2210.03629
- **版本历史**：v1 2022-10-06；v2 2022-11-27；v3 2023-03-10
- **会议**：ICLR 2023（v3 为 camera-ready）

## 摘要（原文）

While large language models (LLMs) have demonstrated impressive capabilities across tasks in language understanding and interactive decision making, their abilities for reasoning (e.g. chain-of-thought prompting) and acting (e.g. action plan generation) have primarily been studied as separate topics. In this paper, we explore the use of LLMs to generate both reasoning traces and task-specific actions in an interleaved manner, allowing for greater synergy between the two: reasoning traces help the model induce, track, and update action plans as well as handle exceptions, while actions allow it to interface with external sources, such as knowledge bases or environments, to gather additional information. We apply our approach, named ReAct, to a diverse set of language and decision making tasks and demonstrate its effectiveness over state-of-the-art baselines, as well as improved human interpretability and trustworthiness over methods without reasoning or acting components. Concretely, on question answering (HotpotQA) and fact verification (Fever), ReAct overcomes issues of hallucination and error propagation prevalent in chain-of-thought reasoning by interacting with a simple Wikipedia API, and generates human-like task-solving trajectories that are more interpretable than baselines without reasoning traces. On two interactive decision making benchmarks (ALFWorld and WebShop), ReAct outperforms imitation and reinforcement learning methods by an absolute success rate of 34% and 10% respectively, while being prompted with only one or two in-context examples.

## 摘要（中文翻译）

大语言模型（LLM）在语言理解和交互式决策任务上已展现出令人瞩目的能力，但其"推理"能力（如思维链提示）与"行动"能力（如行动计划生成）一直被作为彼此独立的课题分别研究。本文探索让 LLM 以**交错（interleaved）**的方式同时生成推理轨迹与任务相关的具体行动，从而让二者产生更强的协同：推理轨迹帮助模型归纳、跟踪并更新行动计划，并处理异常；行动则使其能够与外部知识库或环境交互，获取额外信息。我们将该方法命名为 **ReAct**，应用于一系列语言与决策任务，证明其相对当时 SOTA 基线的有效性，同时在人类可解释性和可信度上优于缺少推理或行动组件的方法。具体而言：在问答（HotpotQA）和事实核查（Fever）任务上，ReAct 通过调用一个简单的 Wikipedia API，克服了思维链推理中普遍存在的幻觉与错误传播问题，并生成比无推理轨迹基线更可解释的、类人的解题轨迹；在两个交互式决策基准（ALFWorld 与 WebShop）上，ReAct 仅用一到两个上下文示例提示，就以 34% 和 10% 的绝对成功率优势超越了模仿学习与强化学习方法。

## 核心思想

ReAct = **Reason + Act**：在同一个 LLM 推理循环中，交错产生三类 token——

- **Thought（思考）**：自然语言推理，用于分解任务、制定/修订计划、处理异常。
- **Action（行动）**：调用外部工具/环境的结构化动作（如 `search[entity]`、`lookup[string]`、`finish[answer]`）。
- **Observation（观察）**：环境返回的结果，反馈进上下文供下一步推理。

循环形式：`Thought → Action → Observation → Thought → …`，直到产出最终答案。

## 解决的关键问题

- **纯思维链（CoT）**：仅靠内部推理，缺乏外部信息接地，易产生幻觉与错误沿推理链传播，且事实无法核验。
- **纯行动（Act-only）**：能与环境交互但缺少高层推理，难以分解复杂目标、难以从失败中恢复。
- **ReAct 的协同**：推理为行动提供计划与纠错；行动为推理提供真实世界的接地（grounding），二者互补。

## 实验与结果

- **知识密集型推理**：HotpotQA（多跳问答）、FEVER（事实核查），通过简单 Wikipedia API（search / lookup / finish）接地，显著缓解幻觉。
- **交互式决策**：
  - ALFWorld（文本具身环境）：绝对成功率比模仿/强化学习基线高 **+34%**。
  - WebShop（网购环境）：绝对成功率高 **+10%**。
- 仅需 **1–2 个上下文示例**（few-shot），无需微调即可超越当时 SOTA。
- 额外收益：生成的轨迹**类人、可解释、可信**，便于人类诊断与干预。

## 历史意义

ReAct 早于 ChatGPT 发布（2022 年 10 月），是公认的"现代 LLM Agent"奠基性工作之一：它首次系统性地把"推理 + 工具调用 + 环境反馈"统一进一个提示驱动的循环，奠定了后续 Agent 框架（工具使用、ReAct-style agent loop、LangChain Agent、AutoGPT 等）的范式基础。当下（2026）热门的 Agent Harness 等工程范式，其智能体内核仍可追溯至这一 Thought–Action–Observation 循环。
