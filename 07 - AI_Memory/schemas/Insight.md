---
title: Insight
type: schema
permalink: main/schemas/insight
entity: Insight
version: 1
schema:
  insight_type(enum): '[idea, learning, reflection], what kind of durable insight
    this is'
  maturity?(enum): '[raw, useful, promoted], how ready it is for reuse'
  topic?: string, the topic or area the insight belongs to
  source?: string, where the insight came from
settings:
  validation: warn
---

# Insight

| Notes | Modification Date | Approved By |
|:--|:--|:--|
| Initial Document | September 7, 2026 | User |

Schema for ideas and learning that should remain useful after the day they
were captured. The body should explain the reasoning and application, while
observations keep the insight searchable.

## Observations

- [convention] Every insight carries an `[insight_type]` observation.
- [convention] Use `[application]` to explain where the insight changes work.
- [convention] Link an insight to the journal entry, project, or procedure that gave it context.