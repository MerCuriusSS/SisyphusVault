---
title: Startup Router
type: instruction
permalink: main/instructions/startup-router
tags:
- basic-memory
- instructions
- router
---

# Startup Router

| Notes | Modification Date | Approved By |
|:--|:--|:--|
| Initial Document | September 7, 2026 | User |

> Read this note before any knowledge-base work in Basic Memory project `main`.

## Non-negotiable Rules

1. Search before creating a note; update an existing note when it is the same entity.
2. Pass project `main` explicitly on every read, search, list, and write.
3. Use exact folder casing from this router and the domain instruction notes.
4. Read an existing note in full before overwriting it.
5. Keep observations and Relations sections on every content note.
6. Ask when a note could belong to more than one domain instead of guessing.
7. Update both sides of a Project ↔ Task relationship.

## Dispatch Table

| Work type | Load in order |
|:--|:--|
| Journal, daily record, or raw idea | `memory://main/instructions/journal-instructions` |
| Reusable work method or troubleshooting flow | `memory://main/instructions/procedures-instructions` |
| Project planning or task update | `memory://main/instructions/projects-instructions` → `memory://main/tasks/task-board` |
| Ambiguous knowledge-base work | Load all three domain instruction notes |

## Known Failure Modes

- Duplicate notes: search by full title and keywords before creating.
- Parallel folder trees: preserve exact casing, especially existing `schemas/`.
- Stale Task Board: update it whenever an active task changes.
- Lost context: record the next step and blockers in the Project or Task note.
- Knowledge islands: add typed Relations and links to the surrounding Project, Procedure, Insight, or Journal entry.

## Observations

- [purpose] This router is the first entry point for Basic Memory project `main`.
- [convention] The user speaks naturally; Codex handles note types, observations, relations, and validation.

## Relations

- dispatches_to [[Journal Instructions]]
- dispatches_to [[Procedures Instructions]]
- dispatches_to [[Projects Instructions]]
- dispatches_to [[Task Board]]

- teaches [[Basic Memory Cheat Sheet]]