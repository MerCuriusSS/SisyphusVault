---
tags:
  - Resources/AI
category: 思维
status: 加工
project:
application:
source:
---
### 🔴概念

- LLM（Large-Language-Model）表示**大脑**，本质上是「**文字进，文字出**」
- Agent表示**有四肢的大脑**（会调用工具），本质上是「**文字进->使用工具改变外部环境->反馈结果**」


### 🔴构建模式
#### ReAct模式（推理+行动）

- 基本原理：
	- 通过系统提示词让LLM扮演一个「**能使用工具**」、「**分任务执行**」的角色
	- 每个任务执行按照「思考(**thought**)->行动(**action**)->中间结果(**observation**)」模式进行，循环往复。
	- 多个中间结果叠加形成最终结果(**finalAnswer**)时完成整个过程，流程结束。
- ReAct模式提示词：[ReAct模式 系统提示词](../04%20-%20Resources/ReAct模式%20系统提示词.md)
- 时序图：[Agent-ReAct模式时序图](../excalidraw/Agent-ReAct模式时序图.md)

#### Plan&Execute模式
- 基本原理：
	- 通过模型将用户任务生成为结构化的执行计划
	- Agent按顺序执行计划中的每一步并调用可用的工具
	- 模型结合已完成的步骤、中间结果，修改或重建剩余计划，循环往复。
	- 多个中间结果叠加形成最终结果时完成整个过程，流程结束
- 时序图：[Agent-Plan&Execute时序图](../excalidraw/Agent-Plan&Execute时序图.md)

#### Plan&Execute模式与ReAct模式对比：
- ReAct模式：任务的“分步”只是一段写进上下文的推理文本，只有**软性约束**，后续的新信息很容易将它冲淡或带偏。
- Plan&Execute模式：任务的规划被指定为像系统提示词一样的**强制约束**，执行器不会擅自偏离，遗忘或干扰的风险极低。