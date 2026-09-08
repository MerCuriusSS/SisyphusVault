---
title: AI零代码应用生成平台
type: project
permalink: main/projects/ai-零代码应用生成平台/ai-零代码应用生成平台
tags:
- project
- AI
- agent
- zero-code
status: active
goal: Build an AI zero-code application generation platform with complex task execution,
  long-term memory, and observable production behavior.
next_step: 补齐生产级 Agent 相关知识并完成技术实践
---

# AI零代码应用生成平台

## Goal

完成一个 AI 零代码应用生成平台，使它能够规划和执行复杂的应用生成任务，拥有长期记忆，并且让执行过程可观测、可评测、可控制。

## Current State

已经完成 Prompt / 上下文工程、RAG 知识检索和 Tool Calling 工具调用。

目前缺少 Agent 任务规划与执行、记忆、状态和工作流编排、评测、监控、异常处理以及成本优化相关的知识和技术实践。

生产级 Agent 的能力边界已经通过 [[生产级 Agent：LangChain、LangGraph 与 LangSmith 的分工]] 初步理清，Java 后端性能排查方法可以作为后续稳定性和性能实践的通用方法参考。

## Next Step

先补齐生产级 Agent 的知识，再用小型技术实践验证规划执行、记忆状态、工作流、评测、监控、异常和成本控制能力。

## Observations

- [status] active
- [goal] 让平台能够生成复杂应用并具备生产级 Agent 能力
- [completed] Prompt / 上下文工程、RAG 知识检索、Tool Calling
- [gap] 规划执行、记忆状态、工作流编排、评测、监控、异常处理、成本优化
- [next_step] 补齐相关知识并完成技术实践

## Relations

- contains [[补齐生产级 Agent 能力]]
- informed_by [[生产级 Agent：LangChain、LangGraph 与 LangSmith 的分工]]
- supported_by [[系统性定位和解决 Java 后端性能问题]]