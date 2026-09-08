---
title: Journal Entry
type: schema
permalink: main/schemas/journal-entry
entity: JournalEntry
version: 1
schema:
  date: string, the journal date in YYYY-MM-DD format
  mood?(enum): '[low, neutral, good, energized, mixed], optional mood signal'
  highlight?: string, the most important event or realization of the day
settings:
  validation: warn
---

# Journal Entry

| Notes | Modification Date | Approved By |
|:--|:--|:--|
| Initial Document | September 7, 2026 | User |

Schema for low-friction daily notes. A journal entry records what happened,
what the user noticed, and what changed. It should stay easy to write rather
than becoming a form.

## Observations

- [convention] Every journal entry carries one `[date]` observation.
- [convention] Use `[event]`, `[idea]`, `[learning]`, and `[change]` when they apply.
- [convention] Promote reusable ideas to an Insight note instead of copying the full journal entry.