# Knowledge Base Index

## ai-agents

Foundational concepts and research behind LLM-based agents.

| Article | Summary | Updated |
|---------|---------|---------|
| [Chain-of-Thought Prompting Elicits Reasoning in LLMs](ai-agents/chain-of-thought.md) | Wei et al., Google Brain (Jan 2022, NeurIPS 2022): few-shot "think step by step" exemplars elicit multi-step reasoning; foundational for agent planning / task decomposition. | 2026-05-18 |
| [ReAct: Synergizing Reasoning and Acting in Language Models](ai-agents/react-reasoning-acting.md) | Princeton & Google (Oct 2022, ICLR 2023) foundational LLM-agent paper: interleaving reasoning traces with tool/environment actions in a Thought–Action–Observation loop; +34% / +10% on ALFWorld / WebShop with 1–2 shots. Pre-ChatGPT origin of modern agent loops. | 2026-05-18 |
| [In-context RL with Algorithm Distillation](ai-agents/algorithm-distillation.md) | DeepMind (Oct 2022, ICLR 2023): distill an RL learning algorithm into a sequence model from training histories; in-context self-improvement without weight updates. Self-Reflection-route counterpart. | 2026-05-18 |
| [Chain of Hindsight Aligns LMs with Feedback](ai-agents/chain-of-hindsight.md) | Liu et al., UC Berkeley (Feb 2023, ICLR 2024): turn all feedback into sentence sequences and fine-tune on them; lightweight alternative to RLHF for learning from feedback. | 2026-05-18 |
| [Toolformer: Language Models Can Teach Themselves to Use Tools](ai-agents/toolformer.md) | Meta AI (Feb 2023, NeurIPS 2023): self-supervised approach where the model learns which API to call/when/with what args and bakes tool use into weights. The "training-time" counterpart to ReAct's prompt-time tool use. | 2026-05-18 |
| [Reflexion: Language Agents with Verbal Reinforcement Learning](ai-agents/reflexion.md) | Shinn et al. (Mar 2023, NeurIPS 2023): reinforce agents via verbal feedback + episodic memory instead of weight updates; Actor/Evaluator/Self-Reflection loop on top of ReAct; 91% pass@1 on HumanEval vs GPT-4's 80%. | 2026-05-18 |
| [LLM+P: Empowering LLMs with Optimal Planning Proficiency](ai-agents/llm-p.md) | Liu et al., UT Austin (Apr 2023): translate NL problems to PDDL, solve with a classical planner, translate back — neuro-symbolic fix for LLMs' unreliable long-horizon planning. | 2026-05-18 |
| [Tree of Thoughts: Deliberate Problem Solving with LLMs](ai-agents/tree-of-thoughts.md) | Yao et al., Princeton & Google DeepMind (May 2023, NeurIPS 2023): generalize CoT into a searchable tree of thoughts with self-evaluation, lookahead and backtracking; 74% vs 4% on Game of 24. | 2026-05-18 |
| [OpenAI Function Calling (API update)](ai-agents/openai-function-calling.md) | OpenAI (Jun 13 2023): native function calling on gpt-4-0613/gpt-3.5-turbo-0613 — describe functions via JSON schema, model returns schema-compliant JSON args. The engineering watershed turning ReAct-style tool use from fragile prompting into production-grade native capability. | 2026-05-19 |
| [LLM Powered Autonomous Agents](ai-agents/llm-powered-autonomous-agents.md) | Lilian Weng's classic Jun 2023 survey blog defining the canonical mental model: Agent = LLM (brain) + Planning + Memory + Tool use. The unifying framework anchor over the entries above. | 2026-05-18 |

## ai-engineering

Building and managing software with AI coding agents.

| Article | Summary | Updated |
|---------|---------|---------|
| [2023–24 Agent Framework Wave](ai-engineering/agent-framework-wave.md) | Survey of the post–Function-Calling framework explosion: LangChain (2022.10), AutoGPT/BabyAGI (2023.03–04, hype & disillusionment), AutoGen (2023.08), CrewAI (2023.11), LangGraph (2024.01) — first engineering-era phase before MCP. | 2026-05-19 |
| [AutoGen: Multi-Agent Conversation](ai-engineering/autogen.md) | Wu et al., Microsoft Research (Aug 2023): open framework for building LLM apps from multiple customizable, conversable agents; declarative conversation patterns. Representative of the multi-agent collaboration phase of the framework wave. | 2026-05-19 |
| [2024–25 Capability Leap: Computer Use / Coding Agent](ai-engineering/capability-leap.md) | Survey of the 2024 capability jump: SWE-bench (2023.10) → Devin (2024.03) → SWE-agent/ACI (2024.05) for coding agents; Anthropic Computer Use (2024.10) → OpenAI Operator (2025.01) for GUI control. The phase between the framework wave and MCP. | 2026-05-19 |
| [SWE-bench: Can LMs Resolve Real-World GitHub Issues?](ai-engineering/swe-bench.md) | Jimenez et al., Princeton (Oct 2023, ICLR 2024): 2,294 real GitHub issue→PR tasks across 12 Python repos; defined the coding-agent yardstick (Claude 2 solved just 1.96%). Same group as ReAct. | 2026-05-19 |
| [SWE-agent: Agent-Computer Interfaces](ai-engineering/swe-agent.md) | Yang et al., Princeton (May 2024, NeurIPS 2024): a custom Agent-Computer Interface (ACI) — interfaces designed for agents not humans — lifts SWE-bench to 12.5% pass@1. Direct precursor to Harness's "design the environment for the agent". | 2026-05-19 |
| [MCP: Model Context Protocol](ai-engineering/model-context-protocol.md) | Anthropic (Nov 25 2024): open standard ("USB-C for AI apps") connecting AI apps to tools/data/workflows via JSON-RPC client-server; collapses M×N integration to M+N. By 2026 the de-facto agent interconnect standard (donated to Linux Foundation). | 2026-05-19 |
| [Building Effective Agents (Anthropic)](ai-engineering/building-effective-agents.md) | Anthropic (Dec 19 2024): the "less is more" design canon — distinguish workflows vs agents, prefer simple composable patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer) over frameworks; simplicity, transparency, well-crafted ACI. The design-pattern-consolidation node into Harness. | 2026-05-19 |
| [A2A: Agent2Agent Protocol](ai-engineering/a2a-protocol.md) | Google (Apr 9 2025; to Linux Foundation Jun 2025): open protocol for agent↔agent discovery, communication and task delegation across vendors/frameworks. Complements MCP — MCP connects agents to tools, A2A connects agents to each other. 50+ launch partners. | 2026-05-19 |
| [Harness Engineering: Building Software with AI Agents](ai-engineering/harness-engineering.md) | OpenAI built a ~1M LOC product with zero manual code using Codex agents; key lessons on context management, agent legibility, enforced architecture, and garbage collection for agent-generated codebases. | 2026-05-14 |
| [Thariq: the Unreasonable Effectiveness of HTML (Claude Code)](ai-engineering/thariq-html-effectiveness.md) | Thariq, Anthropic Claude Code team (May 2026): ask Claude for HTML, not Markdown — once docs are read & fed to later sessions rather than hand-edited, Markdown's editing advantage vanishes while its expressiveness limit remains; HTML is Anthropic's internal default. | 2026-05-18 |
| [Karpathy: ask LLMs for HTML & human–AI I/O modalities](ai-engineering/karpathy-ask-for-html.md) | Karpathy (May 2026): practical tip to ask LLMs to "structure your response as HTML"; argues audio = preferred human→AI input, vision = preferred AI→human output; output ladder text → markdown → HTML → interactive neural video. | 2026-05-18 |
