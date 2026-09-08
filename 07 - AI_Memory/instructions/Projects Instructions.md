---
title: Projects Instructions
type: instruction
permalink: main/instructions/projects-instructions
tags:
- basic-memory
- projects
- tasks
---

# Projects Instructions

| Notes | Modification Date | Approved By |
|:--|:--|:--|
| Initial Document | September 7, 2026 | User |

## Scope

Use this domain for multi-step outcomes and the concrete actions needed to
move them forward. A Project holds durable context; a Task holds one checkable
action.

## Folder Map

- Projects: `Projects/{Project Name}/{Project Name}.md`
- Tasks: `Tasks/{short imperative title}.md`
- Active task index: `Tasks/Task Board.md`

## Project Workflow

1. Search before creating a Project or Task.
2. Keep the Project note focused on goal, scope, current state, decisions, capability gaps, and next step.
3. Break the next concrete action into a Task with a checkable step list.
4. Use `planned`, `active`, `blocked`, `done`, and `abandoned` for Tasks.
5. Use `planned`, `active`, `paused`, and `done` for Projects.
6. Link Tasks to Projects with `- project [[Project Name]]`; list the same Tasks in the Project Relations section.
7. Keep `Task Board` limited to planned, active, and blocked Tasks. Completed Tasks remain searchable but leave the board.

## Current Project

The first project is `[[AI零代码应用生成平台]]`. Its current gap is production-grade
Agent capability: planning and execution, memory, state, workflow orchestration,
evaluation, monitoring, exception handling, and cost optimization.

## Observations

- [convention] A Project without a next step becomes a knowledge island.
- [convention] A Task without a Project relation loses the reason it exists.

## Relations

- routes_from [[Startup Router]]
- uses_schema [[Project]]
- uses_schema [[Task]]
- maintains [[Task Board]]