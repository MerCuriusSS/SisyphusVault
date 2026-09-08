---
title: Basic Memory Cheat Sheet
type: instruction
permalink: main/instructions/basic-memory-cheat-sheet
tags:
- basic-memory
- cheat-sheet
- how-to
---

# Basic Memory Cheat Sheet

你不需要记 Basic Memory 的语法，直接用自然语言告诉 Codex 你要记录、查找、更新还是建立关联。

## Capture：随手记录

可以说：

> 记录：今天我把 LangChain、LangGraph、LangSmith 的定位理清了。

Codex 会先读取 `Startup Router` 和对应领域 instruction，搜索当天的 `Journal/YYYY-MM-DD.md`，然后追加到日记；如果内容值得长期复用，再单独形成 Insight，并链接回当天日记。

## Retrieve：找回内容

可以说：

> 查找：我对生产级 Agent 的理解。

或：

> AI 零代码应用生成平台现在缺什么？

Codex 会先按标题和关键词搜索，再读取命中的完整 note；需要理解上下文时，会沿着项目、任务、Insight 和 Procedure 的关系继续查找。

## Update：更新现有内容

可以说：

> 更新任务“补齐生产级 Agent 能力”：我已经完成任务规划与执行学习。

Codex 会先完整读取原 Task，做有针对性的更新，再同步 Project 和 `Task Board`，避免状态和关系过期。不要直接创建同名的新任务。

## Connect：建立关联

可以说：

> 把“生产级 Agent：LangChain、LangGraph 与 LangSmith 的分工”关联到“AI零代码应用生成平台”，说明它是项目下一阶段的依据。

Codex 会在相关 note 的 `Relations` 中建立带类型的 wiki-link，并检查关联能从项目和任务两端走通。

## 已建立的入口

- 日记与想法：`Journal/`、`Insights/`
- 工作方法与排障流程：`Procedures/`
- 项目与任务：`Projects/`、`Tasks/`
- 规则入口：`memory://main/instructions/startup-router`

## 关键规则

1. 每次先搜索，再决定新建还是更新。
2. Basic Memory 始终使用项目 `main` 和本地模式。
3. 目录大小写保持一致。
4. 覆盖前先完整读取原 note。
5. 每条内容 note 保留 `Observations` 和 `Relations`。
6. 不确定归属时先问，不自行猜测。

## 当前真实示例

- 日记：`Journal/2026-09-07.md`
- Insight：`Insights/生产级 Agent：LangChain、LangGraph 与 LangSmith 的分工.md`
- Procedure：`Procedures/系统性定位和解决 Java 后端性能问题.md`
- Project：`Projects/AI零代码应用生成平台/AI零代码应用生成平台.md`
- Task：`Tasks/补齐生产级 Agent 能力.md`

## Relations

- routes_from [[Startup Router]]
- demonstrates [[2026-09-07]]
- demonstrates [[生产级 Agent：LangChain、LangGraph 与 LangSmith 的分工]]
- demonstrates [[AI零代码应用生成平台]]
- demonstrates [[补齐生产级 Agent 能力]]