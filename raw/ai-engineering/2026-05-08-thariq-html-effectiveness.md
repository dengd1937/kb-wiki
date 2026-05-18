# Using Claude Code: The Unreasonable Effectiveness of HTML（Thariq）

> Source（推文）: https://x.com/trq212/status/2052809885763747935
> Source（原文，X 长文，需登录）: https://x.com/i/article/2052796100608974848
> Source（高保真转述存档）: https://simonwillison.net/2026/May/8/unreasonable-effectiveness-of-html/
> 配套示例站: https://thariqs.github.io/html-effectiveness/
> Collected: 2026-05-18
> Published: 2026-05-08

## 元数据

- **作者**：Thariq Shihipar（@trq212），Anthropic Claude Code 团队工程师
- **载体**：X 长文（long-form Article）；通过一条仅含链接的推文发布
- **发布日期**：2026-05-08
- **互动**：该推文约 16,612 likes / 约 2,149 retweets（采集时快照，约数）；原文据报道在 X 上获约 800 万次浏览
- **采集来源说明**：原 X 长文需登录访问；以下内容依据 Simon Willison 的转述存档及多家报道交叉印证整理，并标注 verbatim 与转述部分

## 核心论点

为日常 Claude Code 用户提出反直觉建议：**向 Claude 索要 HTML 输出，而非 Markdown**。论证：当 Claude 生成的文档越来越多地被当作规格说明、参考文件、头脑风暴产物——被阅读、并喂给后续 Claude 会话，而非逐行人工编辑时，**Markdown 失去了它的主要优势（便于人类编辑），却保留了主要局限（表达力受限）**，这一权衡不再成立。HTML 已成为 Anthropic 内部用于计划、代码评审、设计系统、报告的默认格式。

## HTML 相对 Markdown 的关键优势

- 可嵌入 SVG 图表与交互式小组件
- 支持页内导航
- 更强的视觉排版能力
- 更愉悦的信息浏览体验

## 原文示例提示（verbatim 引用，经 Simon Willison 转述）

> "Help me review this PR by creating an HTML artifact that describes it. I'm not very familiar with the streaming/backpressure logic so focus on that. Render the actual diff with inline margin annotations, color-code findings by severity and whatever else might be needed to convey the concept well."

## 论证支撑

Thariq 发布了配套站点，收录约 20 个由 Claude Code 生成的、各自独立的 HTML 文件，每个对应一个真实用例（PR 评审、设计系统、报告等），公开于 https://thariqs.github.io/html-effectiveness/ 。

## Simon Willison 的评注

Simon 表示自己此前因 token 效率（GPT-4 时代 8,192 token 限制）默认使用 Markdown；Thariq 的文章促使他重新考虑——尤其是输出格式层面。他用 GPT-5.5 生成了一份解释某 Linux 安全漏洞的交互式 HTML，借助 CSS/JavaScript 富样式来澄清复杂技术内容，验证了该思路。

## 与本库关联

本文即 Karpathy 2026-05-11 推文所引用的对象；Karpathy 在其基础上引申出「音频=人→AI 偏好输入、视觉=AI→人偏好输出」与输出格式演进阶梯的判断。
