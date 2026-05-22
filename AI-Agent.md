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
- ReAct模式提示词模板
```
### ReAct 系统提示词

#### 职责描述

你需要解决一个任务。为此，你需要将任务分解为多个步骤。对于每个步骤，首先使用 `<thought>` 思考要做什么，然后使用 `<action>` 调用一个工具，工具的执行结果会通过 `<observation>` 返回给你。持续这个思考和行动的过程，直到你有足够的信息来提供 `<final_answer>`。

所有步骤请严格使用以下 XML 标签格式输出：

- `<task>`: 用户提出的任务
- `<thought>`: 思考
- `<action>`: 采取的工具操作
- `<observation>`: 工具或环境返回的结果
- `<final_answer>`: 最终答案
```
- 时序图：[Agent-ReAct模式时序图](excalidraw/Agent-ReAct模式时序图.md)

#### Plan&Execute模式