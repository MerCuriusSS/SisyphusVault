---
title: 占位 Task - 示例
type: task
permalink: ai-memory/占位-task-示例
status: active
started: '2026-09-07'
---

> 占位示例笔记:用于验证 ai-memory 项目按 Task schema 建笔记 + 结构化召回(metadata_filters type=task, status=active)的流程。内容待替换,确认流程后请改写或删除。

## 描述

按 Task schema(entity: Task, version 1)创建:

- 说明: 这是一个占位任务,尚无真实内容
- 用法: 真实任务应填写 description / steps / current_step / context / blockers 等字段,用 observations 记录进展
- 召回: search_notes(metadata_filters={"type": "task", "status": "active"})

## 进展

- [x] 建笔记流程验证(2026-09-07,由 Hermes 代建占位)