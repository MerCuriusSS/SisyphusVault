---
title: Procedures Instructions
type: instruction
permalink: main/instructions/procedures-instructions
tags:
- basic-memory
- procedures
- work-methods
---

# Procedures Instructions

| Notes | Modification Date | Approved By |
|:--|:--|:--|
| Initial Document | September 7, 2026 | User |

## Scope

Use this domain for reusable work methods, troubleshooting runbooks,
checklists, decision guides, and lessons from repeated practice.

## Folder Map

- Procedures: `Procedures/{descriptive title}.md`
- Procedure rules: this note
- Related Projects: `Projects/`

## Procedure Workflow

1. Search for the problem, tool names, and likely existing title before creating a procedure.
2. State the situation and the observable symptoms before listing solutions.
3. Record the baseline and the tools used to locate the cause.
4. Separate cause and optimization by layer: code, concurrency, remote calls, database, cache, MQ, JVM, and architecture.
5. Add an effect-verification method using the same traffic or test model where possible.
6. Record risks, error rate, new problems, and possible bottleneck transfer.
7. Set `status` to `verified` only after the method has been used or checked; otherwise use `draft`.

## Naming

Use the phrase the user would search for, such as `系统性定位和解决 Java 后端性能问题`.

## Observations

- [convention] A procedure is not complete without a verification method and boundary conditions.
- [convention] Update `last_verified` whenever practice confirms or corrects the procedure.

## Relations

- routes_from [[Startup Router]]
- uses_schema [[Procedure]]
- links_to [[Projects Instructions]]