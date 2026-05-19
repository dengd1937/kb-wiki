# SWE-agent：Agent-Computer Interface 使能自动化软件工程

> Sources: arXiv 2405.15793（Yang et al., 普林斯顿等），2024-05-06；NeurIPS 2024
> Raw: [SWE-agent（要点/中文）](../../raw/ai-engineering/2024-05-06-swe-agent.md)

## 概述

普林斯顿团队（与 SWE-bench/ReAct 同组）2024 年 5 月提出。SWE-agent 让 LM agent **自主使用计算机**解决软件工程任务，核心是定制的**智能体-计算机接口（ACI）**：增强 agent 创建/编辑文件、浏览仓库、执行测试的能力。SWE-bench 上达 **12.5% pass@1**、HumanEvalFix **87.7% pass@1**（当时 SOTA）。

## 核心思想：ACI

关键洞见：**为人设计的界面（shell、编辑器）不是 agent 的最优界面**。SWE-agent 为 agent 专门设计简洁、强反馈的命令接口（带 lint 反馈的编辑、有界滚动查看、专用搜索），大幅提升真实仓库成功率——"接口设计"本身是 Agent 能力的关键杠杆。

## 在工程化线中的位置

**能力跃迁**阶段的方法代表。ACI 思想（为 agent 而非人优化接口）是 Harness Engineering「为智能体设计环境」的直接前驱，也是 MCP「标准化 agent 接口」的思想近邻。

## 相关条目

- [2024–25 能力跃迁](capability-leap.md) — 本阶段综述。
- [SWE-bench](swe-bench.md) — 本方法所用标尺。
- [Harness Engineering](harness-engineering.md) — ACI「为 agent 设计环境」思想的工程化延伸。
- [ReAct](../ai-agents/react-reasoning-acting.md) — 同一普林斯顿研究组谱系。
