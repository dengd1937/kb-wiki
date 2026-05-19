# OpenAI Function Calling（Function calling and other API updates）

> Source（官方，403/需校验）: https://openai.com/index/function-calling-and-other-api-updates/
> Source（交叉印证）: https://www.infoq.com/news/2023/06/openai-api-function-chatgpt/
> Collected: 2026-05-18
> Published: 2023-06-13

## 元数据

- **发布方**：OpenAI
- **标题**：Function calling and other API updates
- **发布日期**：2023-06-13
- **载体**：OpenAI 官方博客 / API 更新公告
- **采集来源说明**：官网直链返回 403；以下据 OpenAI 官方公告文字（经检索还原）并以 InfoQ 报道交叉印证，verbatim 句子已标注

## 核心内容

随 `gpt-4-0613`、`gpt-3.5-turbo-0613` 等模型发布，引入 **function calling（函数调用）** 能力。

> "Developers can now describe functions to gpt-4-0613 and gpt-3.5-turbo-0613, and have the model intelligently choose to output a JSON object containing arguments to call those functions."

> "These models have been fine-tuned to both detect when a function needs to be called (depending on the user's input) and to respond with JSON that adheres to the function signature. This is a new way to more reliably connect GPT's capabilities with external tools and APIs."

中文：开发者向模型描述一组函数（名称、参数 JSON schema、用途），模型据用户输入**自行判断是否需要调用、调用哪个、并产出符合函数签名的 JSON 参数**。模型经微调以「检测何时该调用」并「返回符合签名的 JSON」，从而更可靠地把 GPT 与外部工具/API 连接起来。

## 模型更新

- `gpt-4-0613` / `gpt-4-32k-0613`：含函数调用的改进版（后者扩展上下文长度）。
- `gpt-3.5-turbo-0613`：同样的函数调用能力，并提升通过 system message 的可操纵性（steerability）。

## 官方给出的用例

- 调用外部工具回答问题的聊天机器人（类似 ChatGPT Plugins）。
- 自然语言 → API 调用：如 "Email Anya to see if she wants to get coffee next Friday" → `send_email(to: string, body: string)`。
- 结构化抽取：如 "What's the weather like in Boston?" → `get_current_weather(location: string, unit: 'celsius'|'fahrenheit')`。

## 在 ReAct 条线中的意义

这是 ReAct 条线的**工程可靠性分水岭**：

- **此前**：ReAct 式工具调用靠提示工程引导，模型以自由文本「描述」动作，需脆弱的文本解析，易格式漂移、易失败。
- **此后**：工具调用成为**模型原生、结构化、经微调保证 JSON 合规**的能力——把 Toolformer「让工具使用进入模型能力」的思想产品化。Anthropic、Google 随后跟进。这是 Agent 从 demo 走向生产的关键拐点，也是后续 MCP（工具接入标准化）、Agent Harness 的基础设施前提。

## 与本库关联

- 承接：ReAct（工具调用的提示时路线）、Toolformer（工具能力进权重的思想）。
- 归位：Lilian Weng 框架中的 **Tool use** 组件的工程落地。
- 下游：MCP、Harness Engineering。
