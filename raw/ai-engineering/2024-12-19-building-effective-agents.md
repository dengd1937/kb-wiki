# Building Effective Agents（Anthropic）

> Source: https://www.anthropic.com/engineering/building-effective-agents
> Collected: 2026-05-18
> Published: 2024-12-19

## 元数据

- **发布方**：Anthropic（Engineering 博客）
- **标题**：Building Effective Agents
- **发布日期**：2024-12-19
- **性质**：工程实践指南 / 设计模式总结

## 核心论点

最成功的 LLM agent 实现使用**简单、可组合的模式**，而非复杂框架。开发者应**从最简单的方案开始**，仅当可度量的收益证明值得时才增加复杂度。

> "Success in the LLM space isn't about building the most sophisticated system. It's about building the _right_ system for your needs."

## 关键区分：Workflows vs Agents

- **Workflows（工作流）**：LLM 与工具通过**预定义代码路径**编排。
- **Agents（智能体）**：LLM **动态自主决定**自己的流程与工具使用，自行掌控如何完成任务。

选择原则：明确、可预测、需一致性的任务用 **workflow**；步数不可预测、需模型自主决策的开放问题才用 **agent**。

## 基础构件：Augmented LLM

基础 LLM 增强检索、工具、记忆——能自行生成检索查询、选择合适工具、决定保留哪些信息。

## 五种工作流模式

1. **Prompt Chaining**：顺序步骤，每次 LLM 调用处理上一步输出；适合可分解任务。
2. **Routing**：对输入分类并分流到专门处理；适合需分别处理的不同类别。
3. **Parallelization**：并行——sectioning（独立子任务）或 voting（多次尝试取共识）。
4. **Orchestrator-Workers**：中心 LLM 动态拆解任务并委派给 worker；适合子任务不可预测。
5. **Evaluator-Optimizer**：生成器与评估器之间的迭代反馈回路；适合有明确评判标准。

## 三条核心原则

1. 保持 agent 设计的**简单性**。
2. 通过**显式规划步骤**优先保证**透明性**。
3. 精心打造**智能体-计算机接口（ACI）**——充分的工具文档与测试。

## 在工程化线中的位置

属**设计模式收敛**节点：框架浪潮带来过度复杂、协议标准化（MCP/A2A）解决接入碎片化之后，本文**回归克制**——用 workflow/agent 二分与五种朴素模式遏制过度工程。其第 3 原则「精心设计 ACI」直接呼应 SWE-agent 的 ACI，并与后续 Harness Engineering「为智能体设计环境、强约束、反馈回路」的生产范式一脉相承——是研究/框架阶段通往 Harness 工程范式的最后一块拼图。
