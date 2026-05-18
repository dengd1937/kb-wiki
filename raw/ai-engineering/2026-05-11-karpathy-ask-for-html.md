# Karpathy：让 LLM「以 HTML 结构化输出」与人机 I/O 模态

> Source: https://x.com/karpathy/status/2053872850101285137
> Collected: 2026-05-18
> Published: 2026-05-11

## 元数据

- **作者**：Andrej Karpathy（@karpathy）
- **载体**：X（Twitter）长推文
- **发布日期**：2026-05-11
- **互动**：约 17,490 likes / 约 1,827 retweets（采集时快照，约数）
- **引用对象**：引用 Thariq（@trq212）推文 https://x.com/trq212/status/2052809885763747935，该推文指向文章《Using Claude Code: The Unreasonable Effectiveness of HTML》

## 原文（英文 verbatim）

This works really well btw, at the end of your query ask your LLM to "structure your response as HTML", then view the generated file in your browser. I've also had some success asking the LLM to present its output as slideshows, etc.

More generally, imo audio is the human-preferred input to AIs but vision (images/animations/video) is the preferred output from them. Around a ~third of our brains are a massively parallel processor dedicated to vision, it is the 10-lane superhighway of information into brain. As AI improves, I think we'll see a progression that takes advantage:

1) raw text (hard/effortful to read)
2) markdown (bold, italic, headings, tables, a bit easier on the eyes) <-- current default
3) HTML (still procedural with underlying code, but a lot more flexibility on the graphics, layout, even interactivity) <-- early but forming new good default
...4,5,6,...
n) interactive neural videos/simulations

Imo the extrapolation (though the technology doesn't exist just yet) ends in some kind of interactive videos generated directly by a diffusion neural net. Many open questions as to how exact/procedural "Software 1.0" artifacts (e.g. interactive simulations) may be woven together with neural artifacts (diffusion grids), but generally something in the direction of the recently viral https://x.com/zan2434/status/2046982383430496444

There are also improvements necessary and pending at the input. Audio nor text nor video alone are not enough, e.g. I feel a need to point/gesture to things on the screen, similar to all the things you would do with a person physically next to you and your computer screen.

TLDR The input/output mind meld between humans and AIs is ongoing and there is a lot of work to do and significant progress to be made, way before jumping all the way into neuralink-esque BCIs and all that. For what's worth exploring at the current stage, hot tip try ask for HTML.

## 中文翻译

顺便说，这招其实很管用：在你的提问末尾让 LLM「把回复结构化为 HTML」，然后在浏览器里打开生成的文件查看。我也成功地让 LLM 把输出做成幻灯片之类的形式。

更一般地，在我看来：**音频是人类向 AI 输入的偏好方式，而视觉（图像/动画/视频）是 AI 向人类输出的偏好方式**。我们大脑约三分之一是专门处理视觉的大规模并行处理器，它是信息进入大脑的「十车道高速公路」。随着 AI 进步，我认为会出现一个利用这一点的演进阶梯：

1) 纯文本（阅读费力）
2) markdown（粗体、斜体、标题、表格，稍微好读些）← 当前默认
3) HTML（底层仍是过程式代码，但在图形、布局乃至交互性上灵活得多）← 早期但正形成新的良好默认
…4、5、6…
n) 交互式神经视频/模拟

我认为这个外推（尽管相关技术尚不存在）最终会走向某种由扩散神经网络直接生成的交互式视频。关于精确/过程式的「Software 1.0」工件（如交互式模拟）如何与神经工件（扩散网格）编织在一起，还有许多开放问题，但大方向类似于近期走红的 https://x.com/zan2434/status/2046982383430496444 。

输入侧也有待改进。仅靠音频、文本或视频都不够——比如我常觉得需要在屏幕上指点/比划，就像有人实际坐在你和电脑屏幕旁时你会做的那些动作。

**TLDR**：人与 AI 之间的输入/输出「意识融合」仍在进行中，在跳到 neuralink 式脑机接口之前，还有大量工作要做、有显著进展空间。就现阶段值得探索的而言，一个实用小贴士：试试让它输出 HTML。

## 要点提炼

- **实用贴士**：提问末尾加「structure your response as HTML」，浏览器打开查看；也可要求做成幻灯片等。
- **核心论断**：音频 = 人 → AI 的偏好输入；视觉 = AI → 人的偏好输出（视觉是大脑信息高速公路）。
- **输出演进阶梯**：raw text → markdown（当前默认）→ HTML（正形成新默认）→ … → 交互式神经视频/模拟（扩散网络直接生成）。
- **开放问题**：过程式「Software 1.0」工件与神经（扩散）工件如何编织。
- **输入侧短板**：纯音频/文本/视频不够，缺少「指点/手势」这类自然交互；BCI 之前仍有大量空间。
- **语境**：引用 Thariq 推文 →《Using Claude Code: The Unreasonable Effectiveness of HTML》。
