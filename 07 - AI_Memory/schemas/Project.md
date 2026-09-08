---
title: Project
type: schema
permalink: main/schemas/project
entity: Project
version: 1
schema:
  status(enum): '[planned, active, paused, done], current project state'
  goal: string, the result the project is intended to achieve
  next_step?: string, the next concrete action
  target_date?: string, the target completion date
settings:
  validation: warn
---

# Project

| Notes | Modification Date | Approved By |
|:--|:--|:--|
| Initial Document | September 7, 2026 | User |

Schema for multi-step projects. A project note is the durable context for its
tasks: goal, boundaries, current state, decisions, capability gaps, and the
next action.

## Observations

- [convention] Every project carries `[status]` and `[goal]` observations.
- [convention] Projects link to their tasks in both directions.
- [convention] Keep the project note focused on outcome and context; put checkable actions in Task notes.