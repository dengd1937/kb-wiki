# Karpathy：让 LLM 输出 HTML 与人机 I/O 模态演进

> Sources: Andrej Karpathy（@karpathy）X 长推文，2026-05-11
> Raw: [原文 verbatim / 中文](../../raw/ai-engineering/2026-05-11-karpathy-ask-for-html.md)

## 概述

Karpathy 2026-05-11 的长推文（引用 Thariq 关于《Using Claude Code: The Unreasonable Effectiveness of HTML》一文）。核心实用贴士：在提问末尾让 LLM「structure your response as HTML」，再用浏览器查看。并由此引申出对人机输入/输出模态演进的判断。

## 核心论断

- **音频是人 → AI 的偏好输入；视觉（图像/动画/视频）是 AI → 人的偏好输出。** 依据：人脑约 1/3 是专门处理视觉的大规模并行处理器，是信息进入大脑的「十车道高速公路」。

## 输出格式演进阶梯

1. raw text（费力）
2. **markdown** — 当前默认
3. **HTML** — 早期但正形成新的良好默认（图形/布局/交互更灵活）
4. … n) 交互式神经视频/模拟（终局：扩散网络直接生成的交互式视频）

开放问题：过程式「Software 1.0」工件（如交互式模拟）与神经（扩散）工件如何编织。

## 输入侧短板

纯音频/文本/视频都不够，缺少「指点/手势」式自然交互；在 neuralink 式 BCI 之前仍有大量工作空间。

## 实践要点

提问末尾加 `structure your response as HTML` → 浏览器打开；也可要求幻灯片等形式。低成本提升 AI 输出可读性。

## 相关条目

- [HTML 的「不合理有效性」（Thariq）](thariq-html-effectiveness.md) — 本推文所引用并引申的原文。
- [Harness Engineering: Building Software with AI Agents](harness-engineering.md) — 同属 AI 辅助工程实践语境（Claude Code 使用）。
