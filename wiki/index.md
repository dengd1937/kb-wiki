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
| [LLM Powered Autonomous Agents](ai-agents/llm-powered-autonomous-agents.md) | Lilian Weng's classic Jun 2023 survey blog defining the canonical mental model: Agent = LLM (brain) + Planning + Memory + Tool use. The unifying framework anchor over the entries above. | 2026-05-18 |

## ai-engineering

Building and managing software with AI coding agents.

| Article | Summary | Updated |
|---------|---------|---------|
| [Harness Engineering: Building Software with AI Agents](ai-engineering/harness-engineering.md) | OpenAI built a ~1M LOC product with zero manual code using Codex agents; key lessons on context management, agent legibility, enforced architecture, and garbage collection for agent-generated codebases. | 2026-05-14 |
| [Thariq: the Unreasonable Effectiveness of HTML (Claude Code)](ai-engineering/thariq-html-effectiveness.md) | Thariq, Anthropic Claude Code team (May 2026): ask Claude for HTML, not Markdown — once docs are read & fed to later sessions rather than hand-edited, Markdown's editing advantage vanishes while its expressiveness limit remains; HTML is Anthropic's internal default. | 2026-05-18 |
| [Karpathy: ask LLMs for HTML & human–AI I/O modalities](ai-engineering/karpathy-ask-for-html.md) | Karpathy (May 2026): practical tip to ask LLMs to "structure your response as HTML"; argues audio = preferred human→AI input, vision = preferred AI→human output; output ladder text → markdown → HTML → interactive neural video. | 2026-05-18 |
