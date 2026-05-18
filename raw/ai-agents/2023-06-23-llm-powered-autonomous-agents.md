# LLM Powered Autonomous Agents

> Source: https://lilianweng.github.io/posts/2023-06-23-agent/
> Collected: 2026-05-18
> Published: 2023-06-23

## 元数据

- **标题**：LLM Powered Autonomous Agents
- **作者**：Lilian Weng（翁丽莲），发布时任 OpenAI Safety / Applied AI 研究团队负责人
- **载体**：个人博客 Lil'Log
- **发布日期**：2023-06-23
- **性质**：综述性博客文章，被广泛引用为 LLM Agent 的经典心智模型

## 核心论点（原文框架）

> "In a LLM-powered autonomous agent system, LLM functions as the agent's brain, complemented by several key components."

LLM 作为智能体的"大脑/控制器"，由三大关键组件补充，构成统一蓝图：

**Agent = LLM（Brain / Controller） + Planning + Memory + Tool use**

## 组件一：Planning（规划）

### 任务分解（Task Decomposition）

复杂任务通常涉及很多步骤，智能体需要知道步骤并提前规划。

- **Chain of Thought (CoT)**：指示模型"think step by step"，用更多推理时计算把难任务拆成更小步骤。
- **Tree of Thoughts (ToT)**：在 CoT 基础上，每一步探索多个推理可能，形成树状搜索（可用 BFS/DFS，结合自评估）。
- 任务分解可由：LLM 简单提示、任务专属指令、或人类输入完成。
- 另提及 LLM+P：借助外部经典规划器，用 PDDL 描述问题。

### 自我反思（Self-Reflection）

智能体对过去行动做自我批评与反思，从错误中学习并改进后续步骤。

- **ReAct**：将动作空间扩展为"任务专属离散动作 + 语言"，在 LLM 内融合推理（reasoning）与行动（acting），交错产生 Thought / Action / Observation。
- **Reflexion**：为智能体配备动态记忆与自我反思能力以提升推理；通过启发式判断低效/幻觉轨迹并将反思写入记忆。
- **Chain of Hindsight (CoH)**：用带反馈标注的历史序列让模型从反馈中改进。
- 也讨论了 Algorithm Distillation（AD）将强化学习历史蒸馏进模型。

## 组件二：Memory（记忆）

用人类记忆类型类比映射到智能体：

- **感觉记忆（Sensory Memory）** → 对原始输入（文本、图像等）学习 embedding 表示。
- **短期记忆（Short-Term Memory）** → 上下文内学习（in-context learning），受限于有限上下文窗口长度。
- **长期记忆（Long-Term Memory）** → 外部向量库，通过快速检索访问，提供近乎无限的可保留信息。

长期记忆访问依赖**最大内积搜索（MIPS, Maximum Inner Product Search）**，采用近似最近邻（ANN）算法以保证检索速度：LSH、ANNOY、HNSW、FAISS、ScaNN。

## 组件三：Tool Use（工具使用）

通过调用外部 API 获取模型权重中缺失的能力——实时信息、代码执行能力、访问专有信息源等，显著扩展模型能力。

- **MRKL（Modular Reasoning, Knowledge and Language）**：LLM 作为路由器，把查询分发给最合适的专家模块（神经或符号）。
- **TALM / Toolformer**：微调 LM 使其学会使用外部工具 API。
- **HuggingGPT**：以 ChatGPT 作为任务规划器，根据模型描述选择 HuggingFace 上的模型执行子任务，再汇总结果。
- **API-Bank**：评测工具增强 LLM 的基准。

## 案例研究与挑战（原文要点）

- 案例：科学发现智能体（ChemCrow）、生成式智能体模拟（Generative Agents Simulation）、概念验证示例（AutoGPT、GPT-Engineer）。
- 公开挑战：有限上下文长度、长期规划与任务分解困难、自然语言接口的可靠性。

## 意义

该文把当时分散的 CoT、ToT、ReAct、Reflexion、Toolformer、HuggingGPT 等工作统一进同一张"LLM as brain + Planning + Memory + Tool use"蓝图，成为 2023 年承上启下、被引用最广的 Agent 定义与心智模型。
