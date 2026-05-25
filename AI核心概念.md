#Resources/AI 

### LLM(Large Language Model)
- 定义：大语言模型，基于「Transformer」，决策大脑，通过词语接龙方式输出内容

### Prompt：
- 定义：输入到LLM的指令
	- #### User Prompt：对话开始前由开发者/系统设置的**指令**，用于设定 LLM 的行为、风格、角色、规则等。【具有强制性约束】
	- #### System Prompt：用户输入的内容。【具备软性约束，容易被淹没】


### Context
- 定义：与大模型的对话历史（大模型记忆本质就是每次对话都读取整个对话历史）

### AI-Agent
- 定义：[AI-Agent](03%20-%20Areas/AI-Agent.md)

###  RAG（检索增强生成）
- 定义：一种搜索本地文档/数据库时快速将「语义相近」的片段获取出来，加入上下文以增强生成内容可靠性的方法

### Function Calling
- 定义：**Agent与LLM**之间关于「工具调用」的约定规范
- 目的：让LLM回答符合一定格式，方便程序解析

### MCP（Model Context Protocol）
- 定义：**Agent与远端工具**调用的约定规范
- 目的：面向LLM、方便Agent执行
- Agent调用流程
	- 建立所需 MCP Server连接
	- 动态发现获取可用工具：`tools/list`
	- 携带 `userPrompt`与 function call格式的`tool_result`，交由LLM决策
	- 返回不使用 或 使用 MCP工具的function call
	- 调用MCP工具执行
	- 将 LLM之前的回复+工具结果 返回给LLM，生成最终回答

### LangChain
- 定义：将多个相互独立的操作步骤串联并抽象出任务链，以编程方式实现可复用、输出可控的工作流。

### Skill
- 定义：提示词按需加载器，通过文字描述在什么场景最适合用什么方式做什么事。