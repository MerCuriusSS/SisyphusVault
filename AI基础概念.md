#Resources/AI 

### LLM(Large Language Model)
- 定义：大语言模型，基于「Transformer」，决策大脑，通过词语接龙方式输出内容

### Prompt：输入到LLM的指令
- #### User Prompt：
- #### System Prompt：


### Context

### AI-Agent
- 定义：[AI-Agent](03%20-%20Areas/AI-Agent.md)

###  RAG（检索增强生成）
- 定义：一种搜索本地文档/数据库时快速将「语义相近」的片段获取出来，加入上下文以增强生成内容可靠性的方法

### Function Calling
- 定义：Agent与LLM之间关于「工具调用」所约定的对话格式

### MCP（Model Context Protocol）
- 定义：面向LLM、方便Agent执行远端工具发现与调用的格式规范
- Agent调用流程
	- 建立所需 MCP Server连接
	- 动态发现获取可用工具：`tools/list`
	- 携带 `userPrompt`与 function call格式的`tool_result`，交由LLM决策
	- 返回不使用 或 使用 MCP工具的function call
	- 调用MCP工具执行
	- 将 LLM之前的回复+工具结果 返回给LLM，生成最终回答

### 