---
tags:
  - AI/agent实现
category: 技术或思维
status: 加工
project:
application:
source:
---
## 核心概念

大模型应用（Agent）的可观测性平台

## 价值

- 调用过程可见，不再是黑盒——看得见
- 花销、用量、耗时有数——算得清
- 执行效果可评估——评的准
- 多次实现版本有对比——可回归

## 核心概念

- Trace：一次调用的完整链路（足迹）
- Observation：链路内的每个检查项
	- span：执行工作单元（如：检索、写prompt、解析结果）
	- generation：模型调用
	- event：时间点
- Score/Eval：对以上结果评分（人工判、程序判、LLM判）
- prompt：