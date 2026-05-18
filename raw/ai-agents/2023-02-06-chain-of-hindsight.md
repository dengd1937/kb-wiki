# Chain of Hindsight Aligns Language Models with Feedback

> Source: https://arxiv.org/abs/2302.02676
> Collected: 2026-05-18
> Published: 2023-02-06（arXiv v1；共 8 个版本，末版 2023-10-18）

## 元数据

- **标题**：Chain of Hindsight Aligns Language Models with Feedback
- **作者**：Hao Liu、Carmelo Sferrazza、Pieter Abbeel
- **机构**：加州大学伯克利分校（业界公认；arXiv 页面未直接列出）
- **arXiv 编号**：2302.02676
- **版本历史**：v1 2023-02-06 …… 末版 2023-10-18（共 8 版）
- **会议**：ICLR 2024（业界公认；arXiv 页面未直接注明）

## 摘要（据 arXiv 整理）

提出 **Chain of Hindsight (CoH)** 用于让语言模型对齐人类偏好/反馈。方法把**所有类型的反馈都转化为句子序列**，再用其微调模型。不同于依赖强化学习或人工挑选生成结果，CoH 让模型以"成对的输出 + 反馈"为条件，学会**根据反馈生成输出**，并识别与纠正其中的负面属性或错误。在摘要和对话基准上显著优于已有对齐方法。

## 核心思想

把"做得好/做得差 + 对应反馈"拼成一个后见之明（hindsight）序列让模型一起读，使模型学会"在反馈条件下生成更好的输出、并主动纠正坏属性"。是一种用语言序列承载反馈、绕开 RLHF 复杂性的对齐/自我改进方法。

## 在 Lilian Weng 框架中的位置

属 **Planning → Self-Reflection**：作为"从反馈中学习改进"的代表方法之一，与 ReAct/Reflexion 并列。

## 意义

提供了"用语言序列直接编码反馈来改进模型"的轻量路线，影响后续从反馈中学习、自我改进与对齐的研究。
