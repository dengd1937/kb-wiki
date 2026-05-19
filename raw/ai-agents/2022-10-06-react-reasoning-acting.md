# ReAct：在语言模型中协同推理与行动（ReAct: Synergizing Reasoning and Acting in Language Models）

> Source: https://arxiv.org/abs/2210.03629 ; 项目页 https://react-lm.github.io/
> Collected: 2026-05-19
> Published: 2022-10-06（arXiv v1；v3 2023-03-10 为 ICLR 2023 camera-ready）
> Full text: https://ar5iv.labs.arxiv.org/html/2210.03629

## 论文信息

- **作者**：Shunyu Yao、Jeffrey Zhao、Dian Yu、Nan Du、Izhak Shafran、Karthik Narasimhan、Yuan Cao
- **机构**：普林斯顿大学、Google Research（Brain team）
- **arXiv 编号**：2210.03629
- **版本历史**：v1 2022-10-06；…；v3 2023-03-10（ICLR 2023 camera-ready）
- **会议**：ICLR 2023

## 摘要

LLM 在语言理解与交互式决策上表现出色，但其"推理"（如 chain-of-thought）与"行动"（如生成动作计划）此前多被分开研究。本文探索让 LLM 以**交替方式**同时生成推理轨迹与任务相关动作：推理轨迹帮助模型归纳、跟踪、更新动作计划并处理异常；动作则让模型与外部知识库或环境交互、获取信息。方法命名为 **ReAct**，应用于多种语言与决策任务，在 SOTA 基线之上提升效果，并改善人类可解释性与可信度。具体：在问答（HotpotQA）与事实核查（Fever）上，ReAct 通过与简单 Wikipedia API 交互，克服了 chain-of-thought 推理中普遍的幻觉与错误传播；在两个交互式决策基准（ALFWorld、WebShop）上，仅用一到两个 in-context 示例，ReAct 就以绝对成功率 **+34%** 和 **+10%** 超过模仿学习与强化学习方法。

## 分章节总结

### 1 引言

- 人类智能的独特之处：把面向任务的行动与言语推理（inner speech）无缝结合，用于自我调节、策略规划和维持工作记忆。
- 两条相关线索各有局限：（a）CoT 推理是**静态黑盒**，模型仅用自身内部表示生成思考、不接地于外部世界，导致事实幻觉与错误传播；（b）用 LLM 在交互环境中规划/行动的工作，多只靠语言先验预测动作，不做高层目标的抽象推理、也不维持工作记忆。
- 此前没有系统研究推理与行动如何**协同**结合用于通用任务求解。
- 贡献：(1) 提出 ReAct 这一基于提示、协同推理与行动的新范式；(2) 在多基准上 few-shot 验证其优于"仅推理"或"仅行动"的方法；(3) 系统消融与分析"行动对推理任务的作用"和"推理对交互任务的作用"；(4) 分析提示设定下 ReAct 的局限，并做初步微调实验。

### 2 ReAct：协同推理 + 行动

- 设定：智能体在时刻 $t$ 收到观察 $o_t$，按策略 $\pi(a_t|c_t)$ 采取动作，上下文 $c_t=(o_1,a_1,\cdots,o_t)$。当 $c_t\mapsto a_t$ 高度隐式时，纯策略学习困难。
- ReAct 的核心想法很简单：把动作空间扩展为 $\hat{\mathcal{A}}=\mathcal{A}\cup\mathcal{L}$，$\mathcal{L}$ 为语言空间。语言空间中的动作 $\hat a_t$ 称为**思考 / 推理轨迹**，不影响外部环境、无观察反馈，而是对当前上下文做推理、产出有用信息并更新上下文以支持后续推理或行动。
- 有用思考的类型：分解任务目标与制定计划、注入相关常识、从观察中抽取关键信息、跟踪进度与转移计划、处理异常并调整计划等。
- 本文主要设定：冻结的 PaLM-540B，用 few-shot in-context 示例提示其生成动作与自由文本思考。推理为主的任务交替生成 thought-action-observation；动作量大的决策任务则让模型自行决定思考稀疏出现的位置。
- ReAct 的独特特性：A) 直观易设计（标注者只需在动作上写下想法）；B) 通用灵活（适配 QA、事实核查、文字游戏、网页导航）；C) 高效稳健（仅 1–6 个示例即强泛化，微调时进一步获益）；D) 与人对齐可控（推理过程可检视，可通过编辑思考在线纠正智能体行为）。

### 3 知识密集型推理任务

#### 3.1 设定

- **数据集**：HotpotQA（多跳问答，需跨 ≥2 个 Wikipedia 段落推理）；FEVER（事实核查，claim 标为 SUPPORTS/REFUTES/NOT ENOUGH INFO）。均采用 question-only 设定，模型不给支撑段落，须靠内部知识或检索。
- **动作空间（简单 Wikipedia API，3 种）**：`search[entity]`（返回实体页前 5 句或建议相似实体）、`lookup[string]`（返回页内含该串的下一句，模拟 Ctrl+F）、`finish[answer]`。该动作空间故意弱于 SOTA 检索器，以模拟人类与 Wikipedia 的交互、迫使模型靠显式语言推理检索。

#### 3.2 方法

- **ReAct 提示**：HotpotQA/Fever 分别手工写 6/3 个 ReAct 格式轨迹作 few-shot 示例（密集思考）。
- **基线（系统消融 ReAct 轨迹得到）**：Standard（去掉全部 thought/action/observation）；CoT（去掉 action/observation，仅推理；另设 CoT-SC：采样 21 条 temperature 0.7 取多数答案）；Act（去掉 thought）。
- **结合内外知识**：ReAct↔CoT-SC 互为回退——ReAct 在限定步数内未给答案则回退 CoT-SC（HotpotQA/FEVER 各设 7/5 步）；CoT-SC 的多数答案出现次数 < n/2 时回退 ReAct。
- **微调**：用 ReAct 生成的 3000 条正确轨迹自举，微调较小模型（PaLM-8/62B）。

#### 3.3 结果与观察

- **ReAct 一致优于 Act**：推理对引导行动（尤其综合最终答案）有价值；微调结果同样确认。
- **ReAct vs CoT**：ReAct 在 Fever 上胜 CoT（60.9 vs 56.3），在 HotpotQA 上略逊（27.4 vs 29.4）。人工标注 200 条轨迹（成功/失败模式，见表2）的关键观察：(A) 幻觉是 CoT 的严重问题——成功模式假阳性 14% vs ReAct 6%，且占其失败的 56%；ReAct 更接地、事实驱动、可信。(B) 交替结构虽提升接地性，但也降低推理灵活性，使 ReAct 推理错误率高于 CoT，且有"重复生成既往 thought/action"的特有错误。(C) ReAct 成功强依赖检索到有信息的内容，无信息检索占错误的 23%。
- **ReAct + CoT-SC 最佳**：HotpotQA 上 ReAct→CoT-SC、Fever 上 CoT-SC→ReAct 最佳；仅用 3–5 个样本即达 CoT-SC 用 21 个样本的水平。
- **微调时 ReAct 最佳**：提示设定下 PaLM-8/62B 的 ReAct 最差（同时学推理与行动难）；但仅用 3000 例微调后 ReAct 成为四法最佳——微调 PaLM-8B 的 ReAct 超过所有 PaLM-62B 提示法，微调 62B 的 ReAct 超过所有 540B 提示法。

### 4 决策任务

- **ALFWorld**：合成文字家务游戏，6 类任务，需在 >50 个位置、>50 步内达成高层目标；内置"常见物品可能位置"的常识挑战。ReAct 每类随机标 3 条带稀疏思考（分解目标 / 跟踪子目标 / 决定下一子目标 / 常识推断物品位置）的轨迹；评测 134 个未见游戏。基线 BUTLER（对每类任务用 $10^5$ 专家轨迹做模仿学习）。
- **WebShop**：1.18M 真实商品、12k 人类指令的在线购物环境，含大量结构化/非结构化文本，须按指令购买商品；以平均得分与成功率（500 测试指令）评测。基线为 IL（1012 条人类轨迹）与 IL+RL（额外 10587 条指令）。
- **结果**：ALFWorld 上最佳 ReAct 试验平均成功率 71%，远超最佳 Act（45%）与 BUTLER（37%），且最差 ReAct 试验（48%）也胜两者最佳；6 次受控试验中 ReAct 对 Act 相对增益 33%–90%、均值 62%。WebShop 上单样本 Act 已与 IL/IL+RL 持平，ReAct 再绝对提升 10% 成功率；但仍远低于专家人类。
- **内部推理 vs 外部反馈**：与 Inner Monologue（IM，思考仅限环境状态观察）对比，ReAct 的推理灵活且稀疏；消融 ReAct-IM 后 ReAct 总成功率 71 vs 53 显著更优——IM 式提示缺乏高层目标分解与常识推理。

### 5 相关工作

- **LLM 推理**：CoT 及其后续（least-to-most、zero-shot-CoT、self-consistency、Selection-Inference、STaR、Faithful reasoning、Scratchpad）。ReAct 不止做孤立固定推理，而是把动作与对应观察整合进连贯输入流。
- **LLM 决策**：WebGPT、BlenderBot、Sparrow、SimpleTOD（多不显式建模推理、依赖昂贵人类反馈）；SayCan、Inner Monologue（机器人规划，IM 是首个闭环系统，ReAct 在其上构建但论证 IM 非真正"内部思考"）。

### 6 结论

ReAct 是协同 LLM 推理与行动的简单有效方法，在多跳问答、事实核查、交互决策上取得更优性能与可解释决策轨迹。局限：大动作空间的复杂任务需更多示例、易超 in-context 长度上限；微调初步有前景，但需更多高质量人类标注。多任务训练 + 与强化学习结合可进一步释放潜力。

## 关键图表

### 图1：四种提示方法对比（核心示意）

![图1 Standard / CoT / Act / ReAct 对比](assets/react-reasoning-acting/fig1-four-methods.png)

(1) 在 HotpotQA 上对比 (a) Standard、(b) CoT（仅推理）、(c) Act（仅行动）、(d) ReAct（推理+行动）；(2) 在 AlfWorld 上对比 (a) Act 与 (b) ReAct。展示模型生成的 (Act, Thought) 与环境的 (Obs)，省略 in-context 示例。

### 图3：HotpotQA 上提示 vs 微调的缩放结果

![图3 提示与微调的缩放对比](assets/react-reasoning-acting/fig3-scaling-prompt-vs-finetune.png)

左 learning=prompt、右 learning=finetune，横轴模型规模 8b/62b/540b，纵轴 HotpotQA EM，对比 Standard/CoT/Act/ReAct。微调下 ReAct 反超其余方法。

### 表1：PaLM-540B 在 HotpotQA / Fever 上的提示结果

| Prompt Method | HotpotQA (EM) | Fever (Acc) |
|---|---|---|
| Standard | 28.7 | 57.1 |
| CoT (Wei et al., 2022) | 29.4 | 56.3 |
| CoT-SC (Wang et al., 2022a) | 33.4 | 60.4 |
| Act | 25.7 | 58.9 |
| ReAct | 27.4 | 60.9 |
| CoT-SC → ReAct | 34.2 | 64.6 |
| ReAct → CoT-SC | 35.1 | 62.0 |
| Supervised SoTA | 67.5 | 89.5 |

### 表2：ReAct 与 CoT 在 HotpotQA 上的成功/失败模式（人工抽样占比）

| 类型 | 定义 | ReAct | CoT |
|---|---|---|---|
| 成功-真阳性 | 正确推理轨迹与事实 | 94% | 86% |
| 成功-假阳性 | 幻觉推理轨迹或事实 | 6% | 14% |
| 失败-推理错误 | 错误推理轨迹（含无法跳出重复步骤） | 47% | 16% |
| 失败-检索结果错误 | 检索为空或无有用信息 | 23% | - |
| 失败-幻觉 | 幻觉推理轨迹或事实 | 0% | 56% |
| 失败-标签歧义 | 预测正确但未精确匹配标签 | 29% | 28% |

### 表3：ALFWorld 各任务成功率（%）

| Method | Pick | Clean | Heat | Cool | Look | Pick 2 | All |
|---|---|---|---|---|---|---|---|
| Act (best of 6) | 88 | 42 | 74 | 67 | 72 | 41 | 45 |
| ReAct (avg) | 65 | 39 | 83 | 76 | 55 | 24 | 57 |
| ReAct (best of 6) | 92 | 58 | 96 | 86 | 78 | 41 | 71 |
| ReAct-IM (avg) | 55 | 59 | 60 | 55 | 23 | 24 | 48 |
| ReAct-IM (best of 6) | 62 | 68 | 87 | 57 | 39 | 33 | 53 |
| BUTLERg (best of 8) | 33 | 26 | 70 | 76 | 17 | 12 | 22 |
| BUTLER (best of 8) | 46 | 39 | 74 | 100 | 22 | 24 | 37 |

### 表4：WebShop 上的得分与成功率

| Method | Score | SR |
|---|---|---|
| Act | 62.3 | 30.1 |
| ReAct | 66.6 | 40.0 |
| IL | 59.9 | 29.1 |
| IL+RL | 62.4 | 28.7 |
| Human (Expert) | 82.1 | 59.6 |

## 参考文献

完整参考文献见 Full text 链接。正文重点引用：Wei et al. 2022（Chain-of-Thought）、Wang et al. 2022a/b（self-consistency / CoT-SC）、Yang et al. 2018（HotpotQA）、Thorne et al. 2018（FEVER）、Shridhar et al. 2020b（ALFWorld / BUTLER）、Yao et al. 2022（WebShop）、Ahn et al. 2022（SayCan）、Huang et al. 2022b（Inner Monologue）、Chowdhery et al. 2022（PaLM）、Zelikman et al. 2022（STaR 自举）。
