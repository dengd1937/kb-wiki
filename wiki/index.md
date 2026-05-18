# Knowledge Base Index

## ai-agents

Foundational concepts and research behind LLM-based agents.

| Article | Summary | Updated |
|---------|---------|---------|
| [ReAct: Synergizing Reasoning and Acting in Language Models](ai-agents/react-reasoning-acting.md) | Princeton & Google (Oct 2022, ICLR 2023) foundational LLM-agent paper: interleaving reasoning traces with tool/environment actions in a Thought–Action–Observation loop; +34% / +10% on ALFWorld / WebShop with 1–2 shots. Pre-ChatGPT origin of modern agent loops. | 2026-05-18 |
| [Toolformer: Language Models Can Teach Themselves to Use Tools](ai-agents/toolformer.md) | Meta AI (Feb 2023, NeurIPS 2023): self-supervised approach where the model learns which API to call/when/with what args and bakes tool use into weights. The "training-time" counterpart to ReAct's prompt-time tool use. | 2026-05-18 |
| [Reflexion: Language Agents with Verbal Reinforcement Learning](ai-agents/reflexion.md) | Shinn et al. (Mar 2023, NeurIPS 2023): reinforce agents via verbal feedback + episodic memory instead of weight updates; Actor/Evaluator/Self-Reflection loop on top of ReAct; 91% pass@1 on HumanEval vs GPT-4's 80%. | 2026-05-18 |
| [LLM Powered Autonomous Agents](ai-agents/llm-powered-autonomous-agents.md) | Lilian Weng's classic Jun 2023 survey blog defining the canonical mental model: Agent = LLM (brain) + Planning + Memory + Tool use. The unifying framework anchor over ReAct / Toolformer / Reflexion. | 2026-05-18 |

## ai-engineering

Building and managing software with AI coding agents.

| Article | Summary | Updated |
|---------|---------|---------|
| [Harness Engineering: Building Software with AI Agents](ai-engineering/harness-engineering.md) | OpenAI built a ~1M LOC product with zero manual code using Codex agents; key lessons on context management, agent legibility, enforced architecture, and garbage collection for agent-generated codebases. | 2026-05-14 |
