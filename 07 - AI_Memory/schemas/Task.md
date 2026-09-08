---
title: Task
type: schema
entity: Task
version: 2
schema:
  description: string, what needs to be done
  status(enum): '[planned, active, blocked, done, abandoned], current task state'
  assigned_to?: string, who is working on this
  priority?(enum): '[low, medium, high], urgency'
  due?: string, target date in YYYY-MM-DD format
  project?: Project, project this task belongs to
  steps?(array): string, ordered steps to complete
  current_step?: integer, which step number is current
  context?: string, key context needed to resume
  started?: string, when work began
  completed?: string, when work finished
  blockers?(array): string, what prevents progress
  parent_task?: Task, parent task if this is a subtask
settings:
  validation: warn
permalink: main/schemas/task
---

# Task

| Notes | Modification Date | Approved By |
|:--|:--|:--|
| Initial Document | September 7, 2026 | Basic Memory plugin |
| Added project workflow, priority, due date, and planned state | September 7, 2026 | User |

Schema for one checkable unit of work. Tasks live in `Tasks/`, use short
imperative titles, and link to a parent project when one exists.

## Observations

- [convention] Every task carries `[description]` and `[status]` observations.
- [convention] Use `[current_step]` and concrete checkboxes for multi-step work.
- [convention] A blocked task records the reason in `[blockers]` instead of silently remaining active.