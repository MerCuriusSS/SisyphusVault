---
title: 生产级 Agent：LangChain、LangGraph 与 LangSmith 的分工
type: insight
permalink: main/insights/生产级-agent-lang-chain-、-lang-graph-与-lang-smith-的分工
tags:
- AI
- agent
- LangChain
- LangGraph
- LangSmith
insight_type: learning
maturity: useful
topic: AI application development
source: AI application development job requirements
---

# 生产级 Agent：LangChain、LangGraph 与 LangSmith 的分工

招聘信息把 Agent 的生产级要求拆成复杂任务规划与执行、记忆与状态、评测、监控、异常处理和成本管理。这个要求暴露出一个实践缺口：只会使用 LangChain，并不等于具备生产级 Agent 的完整能力。

更准确的理解是，三者不是并列的框架选择，而是对应 AI 应用不同阶段的实施方案：

- LangChain：统一接入大模型、组件和单向链，适合快速构建 MVP 和简单固定流程。
- LangGraph：面向复杂任务拆解与编排，支持图式成环、状态持久化、human-in-the-loop 和多 Agent 协同。
- LangSmith：面向落地后的链路追踪、监控、评测和防止能力退化。

这改变了求职表达方式：面试中可以说明三者的定位和适用阶段，而不是笼统地说“使用 LangChain”。它也明确了 [[AI零代码应用生成平台]] 下一步需要补齐的能力。

## Observations

- [insight_type] learning
- [maturity] useful
- [distinction] LangChain handles fast application composition, LangGraph handles complex stateful orchestration, and LangSmith handles production observability and evaluation.
- [application] Use the product stage and complexity to choose the implementation layer instead of treating the three tools as interchangeable frameworks.
- [gap] Production-grade Agent knowledge is still missing in planning, execution, memory, state, workflow, evaluation, monitoring, exception handling, and cost optimization.

## Relations

- recorded_in [[2026-09-07]]
- informs [[AI零代码应用生成平台]]
- relates_to [[补齐生产级 Agent 能力]]