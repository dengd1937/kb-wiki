# 长时间运行 Agent 的有效 Harness 设计（Effective Harnesses for Long-Running Agents）

> Source: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
> Collected: 2026-05-21
> Published: Unknown
> Full text: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents

## 文章信息

- **作者**：Justin Young（Anthropic）
- **载体**：Anthropic Engineering Blog
- **发布日期**：Unknown
- **性质**：工程实践

---

随着 AI agent 变得越来越强大，开发者越来越多地要求它们承担需要数小时甚至数天才能完成的复杂任务。然而，让 agent 在多个 context window 之间保持一致的进展仍然是一个开放性问题。

长时间运行 agent 的核心挑战在于，它们必须在离散的 session 中工作，而每个新 session 开始时都没有之前工作的记忆。想象一下，一个软件项目由轮班工作的工程师组成，而每位新工程师到岗时都不记得上一班发生了什么。由于 context window 的大小是有限的，而且大多数复杂项目无法在单个 window 内完成，agent 需要一种方式来跨越编码 session 之间的鸿沟。

我们开发了一个两部分的解决方案，使 Claude Agent SDK 能够在多个 context window 之间有效地工作：一个 **initializer agent**，在首次运行时设置环境；以及一个 **coding agent**，在每个 session 中负责推进增量进展，同时为下一个 session 留下清晰的产物（artifacts）。你可以在附带的 quickstart 中找到代码示例。

## 长时间运行 Agent 的问题

Claude Agent SDK 是一个强大的通用 agent harness，擅长编码以及需要模型使用工具来收集上下文、规划和执行的其他任务。它具有 context 管理能力，例如 compaction，使 agent 能够在不耗尽 context window 的情况下持续工作。理论上，在这种设置下，agent 应该可以在任意长的时间内持续进行有价值的工作。

然而，compaction 并不足够。开箱即用的情况下，即使像 Opus 4.5 这样的前沿编码模型，在 Claude Agent SDK 上以循环方式跨多个 context window 运行，如果只给它一个高层 prompt（比如"构建一个 claude.ai 的克隆"），也无法构建出生产质量的 Web 应用。

Claude 的失败表现为两种模式。首先，agent 倾向于一次尝试做太多事情——本质上试图一次性完成整个应用。这通常导致模型在实现过程中耗尽 context，使得下一个 session 面对一个功能只实现了一半且没有文档记录的状态。然后 agent 不得不猜测之前发生了什么，并花费大量时间试图让基本应用重新运行。即使有 compaction 这种情况也会发生，因为 compaction 并不总是能向下一个 agent 传递完全清晰的指令。

第二种失败模式通常在项目后期出现。在已经构建了一些功能之后，后续的 agent 实例会环顾四周，看到已经取得的进展，然后宣布任务完成。

这将问题分解为两个部分。首先，我们需要设置一个初始环境，为给定 prompt 所需的 _所有_ 功能奠定基础，这使 agent 能够一步步、一个功能一个功能地工作。其次，我们应该提示每个 agent 向目标推进增量进展，同时在 session 结束时将环境保持在干净的状态。所谓"干净状态"，我们指的是那种适合合并到 main branch 的代码：没有重大 bug，代码整洁且有良好的文档，总体而言，开发者可以轻松开始一个新功能的工作，而不必先清理不相关的混乱。

在内部实验中，我们使用两部分的方案来解决这些问题：

1. **Initializer agent**：第一个 agent session 使用专门的 prompt，要求模型设置初始环境：一个 `init.sh` 脚本、一个记录 agent 所做工作的 `claude-progress.txt` 文件，以及一个显示添加了哪些文件的初始 git commit。
2. **Coding agent**：每个后续的 session 要求模型推进增量进展，然后留下结构化的更新。^1

这里的关键洞察是找到一种方法，让 agent 在以一个全新的 context window 开始时能够快速理解工作状态，这是通过 `claude-progress.txt` 文件和 git 历史共同实现的。这些实践方法的灵感来自了解优秀的软件工程师每天做什么。

## 环境管理

在更新的 Claude 4 prompting guide 中，我们分享了 multi-context window 工作流的一些最佳实践，包括一种使用"第一个 context window 使用不同的 prompt"的 harness 结构。这个"不同的 prompt"要求 initializer agent 用未来 coding agent 有效工作所需的所有必要上下文来设置环境。在这里，我们深入探讨这种环境的一些关键组成部分。

### Feature List

为了解决 agent 一次性尝试构建整个应用或过早认为项目完成的问题，我们提示 initializer agent 编写一个全面的功能需求文件，扩展用户的初始 prompt。在 claude.ai 克隆的例子中，这意味着超过 200 个功能，例如"用户可以打开一个新聊天，输入查询，按回车，并看到 AI 的响应"。这些功能最初都被标记为"failing"，这样后续的 coding agent 就能清楚了解完整功能应该是什么样子。

```json
{
    "category": "functional",
    "description": "New chat button creates a fresh conversation",
    "steps": [
      "Navigate to main interface",
      "Click the 'New Chat' button",
      "Verify a new conversation is created",
      "Check that chat area shows welcome state",
      "Verify conversation appears in sidebar"
    ],
    "passes": false
  }
```

我们提示 coding agent 只能通过更改 `passes` 字段的状态来编辑此文件，并使用措辞强烈的指令，例如"It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality."。经过一些实验，我们最终选择使用 JSON 格式，因为与 Markdown 文件相比，模型不太可能不适当地更改或覆盖 JSON 文件。

### 增量进展

有了这种初始环境脚手架，coding agent 的下一次迭代被要求一次只处理一个功能。这种增量方法被证明是解决 agent 一次做太多事情的倾向的关键。

一旦采用增量工作方式，模型在进行代码更改后将环境保持干净状态仍然至关重要。在我们的实验中，我们发现引发这种行为的最佳方式是要求模型用描述性的 commit message 将进度提交到 git，并在进度文件中写入其进度的摘要。这使得模型可以使用 git 来回滚不良的代码更改，恢复到代码库的可工作状态。

这些方法也提高了效率，因为它们消除了 agent 必须猜测之前发生了什么并花费时间试图让基本应用重新运行的需要。

### 测试

我们观察到的最后一个主要失败模式是 Claude 倾向于在没有适当测试的情况下将功能标记为完成。在没有明确提示的情况下，Claude 倾向于进行代码更改，甚至用单元测试或对开发服务器的 `curl` 命令进行测试，但未能认识到该功能并没有端到端地工作。

在构建 Web 应用的案例中，一旦明确提示 Claude 使用浏览器自动化工具并像人类用户一样进行所有测试，它在端到端验证功能方面表现得相当不错。

![Screenshots taken by Claude through the Puppeteer MCP server as it tested the claude.ai clone.](assets/effective-harnesses-long-running/puppeteer-screenshots.gif)

*Claude 通过 Puppeteer MCP server 在测试 claude.ai 克隆时截取的截图。*

为 Claude 提供这些测试工具显著提高了性能，因为 agent 能够识别和修复仅从代码中不明显的 bug。

一些问题仍然存在，比如 Claude 的视觉能力和浏览器自动化工具的局限性，使得难以识别每种 bug。例如，Claude 无法通过 Puppeteer MCP 看到浏览器原生的 alert modal，因此依赖这些 modal 的功能往往因此更加 buggy。

## 快速上手

有了以上所有内容，每个 coding agent 都会被提示运行一系列步骤来了解当前状况，其中一些非常基本但仍然有帮助：

1. _运行 `pwd` 查看你正在工作的目录。你只能编辑此目录中的文件。_
2. _阅读 git 日志和进度文件，了解最近做了什么工作。_
3. _阅读 feature list 文件，选择尚未完成的最高优先级功能来工作。_

这种方法在每个 session 中为 Claude 节省了一些 token，因为它不必弄清楚如何测试代码。让 initializer agent 编写一个可以运行开发服务器的 `init.sh` 脚本，然后在实现新功能之前进行基本的端到端测试，也是有帮助的。

在 claude.ai 克隆的案例中，这意味着 agent 总是先启动本地开发服务器，并使用 Puppeteer MCP 开始一个新聊天、发送消息并接收响应。这确保了 Claude 能够快速识别应用是否处于损坏状态，并立即修复任何现有的 bug。如果 agent 反而开始实现新功能，它可能会使问题变得更糟。

考虑到以上所有内容，一个典型的 session 以以下 assistant 消息开始：

```
[Assistant] I'll start by getting my bearings and understanding the current state of the project.
[Tool Use] <bash - pwd>
[Tool Use] <read - claude-progress.txt>
[Tool Use] <read - feature_list.json>
[Assistant] Let me check the git log to see recent work.
[Tool Use] <bash - git log --oneline -20>
[Assistant] Now let me check if there's an init.sh script to restart the servers.
<Starts the development server>
[Assistant] Excellent! Now let me navigate to the application and verify that some fundamental features are still working.
<Tests basic functionality>
[Assistant] Based on my verification testing, I can see that the fundamental functionality is working well. The core chat features, theme switching, conversation loading, and error handling are all functioning correctly. Now let me review the tests.json file more comprehensively to understand what needs to be implemented next.
<Starts work on a new feature>
```

## Agent 失败模式与解决方案

| **问题** | **Initializer Agent 行为** | **Coding Agent 行为** |
| --- | --- | --- |
| Claude 过早宣布整个项目完成。 | 设置 feature list 文件：根据输入规范，设置一个结构化的 JSON 文件，包含端到端的功能描述列表。 | 在 session 开始时阅读 feature list 文件。选择一个单独的功能开始工作。 |
| Claude 将环境留在有 bug 或进度未记录的状态。 | 编写初始的 git 仓库和进度笔记文件。 | Session 开始时阅读进度笔记文件和 git commit 日志，并在开发服务器上运行基本测试以捕获未记录的 bug。Session 结束时写入 git commit 和进度更新。 |
| Claude 过早将功能标记为完成。 | 设置 feature list 文件。 | 自行验证所有功能。仅在仔细测试后才将功能标记为"passing"。 |
| Claude 需要花时间弄清楚如何运行应用。 | 编写一个可以运行开发服务器的 `init.sh` 脚本。 | Session 开始时阅读 `init.sh`。 |

*总结长时间运行 AI agent 中四种常见失败模式及解决方案。*

## 未来工作

这项研究展示了长时间运行 agent harness 中一组可能的解决方案，使模型能够在多个 context window 之间推进增量进展。然而，仍然存在开放性问题。

最值得注意的是，目前仍不清楚单个通用 coding agent 是否在跨 context 中表现最佳，或者是否可以通过 multi-agent 架构实现更好的性能。看起来合理的是，专门的 agent——例如测试 agent、质量保证 agent 或代码清理 agent——可以在软件开发生命周期中的子任务上做得更好。

此外，这个演示是为全栈 Web 应用开发优化的。未来的一个方向是将这些发现推广到其他领域。这些经验教训中的部分或全部可能适用于其他类型的长时间运行 agentic 任务，例如科学研究和金融建模。

### 致谢

由 Justin Young 撰写。特别感谢 David Hershey、Prithvi Rajasakeran、Jeremy Hadfield、Naia Bouscal、Michael Tingley、Jesse Mu、Jake Eaton、Marius Buleandara、Maggie Vo、Pedram Navid、Nadine Yasser 和 Alex Notov 的贡献。

这项工作反映了 Anthropic 多个团队的共同努力，他们使 Claude 能够安全地进行长时间跨度的自主软件工程，特别是 code RL 和 Claude Code 团队。有兴趣贡献的候选人欢迎在 anthropic.com/careers 申请。

### 脚注

1. 我们在此上下文中将它们称为独立的 agent，仅因为它们有不同的初始用户 prompt。system prompt、工具集和整体 agent harness 在其他方面完全相同。
