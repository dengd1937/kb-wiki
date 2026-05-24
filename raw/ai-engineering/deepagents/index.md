# DeepAgents 官方文档（中文摄取）

> Source: https://docs.langchain.com/oss/python/deepagents/
> Collected: 2026-05-24
> Maintained by: LangChain
> License: MIT

LangChain 开源的 agent harness——把 Claude Code 的 harness 模式提取出来做成**模型无关、生产就绪**的开源整车。本目录为官方文档全套中文摄取，按概念层 / Get Started / API Reference 三段组织。

---

## 一、概念层

| Title | Summary | 页面 |
|-------|---------|-----|
| [Overview](overview.md) | DeepAgents 全景：harness 定位 + 11 项核心能力 + 何时使用 | overview |
| [vs Claude Agent SDK](comparison.md) | 与 Anthropic 闭源 harness 的 5 维对比与选型建议 | comparison |
| Frameworks/Runtimes/Harnesses（已存档）| 三层定义概念辨析 | 已在 [../2025-10-25-agent-frameworks-runtimes-harnesses.md](../2025-10-25-agent-frameworks-runtimes-harnesses.md) |

## 二、Get Started

| Title | Summary | 页面 |
|-------|---------|-----|
| [Quickstart](quickstart.md) | 5 分钟跑通第一个 deep agent | quickstart |
| [Customization](customization.md) | `create_deep_agent` 全参数定制手册 | customization |
| [Models](models.md) | 模型 provider 配置 + 推荐模型 + evals | models |
| [Backends](backends.md) | 6 种可插拔文件系统后端 | backends |
| [Sandboxes](sandboxes.md) | Modal / Daytona / Runloop / LangSmith 隔离执行 | sandboxes |
| [Interpreters](interpreters.md) | QuickJS 解释器：工具组合 + 数据变换 | interpreters |
| [Permissions](permissions.md) | 声明式文件系统访问控制 | permissions |
| [Human-in-the-loop](human-in-the-loop.md) | 敏感操作的人工审批 | human-in-the-loop |
| [Code](code.md) | Deep Agents Code 终端编码 agent | code |
| [ACP](acp.md) | 在 Zed 等代码编辑器中用 ACP 连接 deep agent | acp |

## 三、API Reference

| Title | Summary | 页面 |
|-------|---------|-----|
| [API Reference](api-reference.md) | `deepagents` 包完整 API 签名速查 | reference.langchain.com |

---

## 摄取说明

- 所有文件均为 2026-05-24 从官方文档摄取的中文化版本
- 代码示例**完整保留**英文原文（未翻译变量名 / 注释关键部分）
- 重要术语保持**中英对照**（如 backend / middleware / subagent）
- 原文 URL 保留在每个文件的 `> Source` 行，方便回到英文官方文档对照
