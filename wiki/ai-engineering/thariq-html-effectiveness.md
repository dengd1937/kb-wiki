# HTML 的「不合理有效性」：向 Claude 索要 HTML 而非 Markdown

> Sources: Thariq Shihipar（@trq212, Anthropic Claude Code 团队）X 长文，2026-05-08；经 Simon Willison 转述存档与多方报道交叉印证
> Raw: [原文要点 / 来源说明](../../raw/ai-engineering/2026-05-08-thariq-html-effectiveness.md)

## 概述

Thariq（Anthropic Claude Code 团队）2026-05-08 发表的反直觉观点：日常使用 Claude Code 时，**应索要 HTML 输出而非 Markdown**。原文为 X 长文，经一条仅含链接的推文发布，获约 800 万浏览，并被 Karpathy 转引。

## 核心论点

当 Claude 生成的文档主要被**阅读、并传入后续 Claude 会话**（作为规格、参考、头脑风暴产物），而非逐行人工编辑时——Markdown 失去主要优势（便于人改），却保留主要局限（表达力弱），权衡不再成立。HTML 已是 Anthropic 内部计划/代码评审/设计系统/报告的默认。

## HTML 优势

- 嵌入 SVG 图表与交互式组件
- 页内导航
- 更强视觉排版
- 更佳信息浏览体验

## 代表用法（原文示例提示）

> "Help me review this PR by creating an HTML artifact that describes it... Render the actual diff with inline margin annotations, color-code findings by severity..."

配套站点收录约 20 个 Claude Code 生成的独立 HTML 用例：https://thariqs.github.io/html-effectiveness/

## 来源透明性

原 X 长文需登录；本条依据 Simon Willison 转述存档 + 多家报道交叉印证整理，verbatim 引用仅限上方示例提示（经 Simon 转述）。

## 相关条目

- [Karpathy: ask LLMs for HTML & human–AI I/O modalities](karpathy-ask-for-html.md) — Karpathy 对本文的引申（人机 I/O 模态、输出格式演进阶梯）。
- [Harness Engineering: Building Software with AI Agents](harness-engineering.md) — 同属 AI 辅助工程实践语境。
