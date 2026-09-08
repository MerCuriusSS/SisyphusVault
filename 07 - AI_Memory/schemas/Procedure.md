---
title: Procedure
type: schema
permalink: main/schemas/procedure
entity: Procedure
version: 1
schema:
  procedure_type(enum): '[how-to, troubleshooting, checklist, decision-guide, experience],
    the procedure form'
  status?(enum): '[draft, verified, stale], whether the procedure is trusted for reuse'
  applies_to?: string, the system or situation where it applies
  last_verified?: string, the last date the procedure was checked in practice
settings:
  validation: warn
---

# Procedure

| Notes | Modification Date | Approved By |
|:--|:--|:--|
| Initial Document | September 7, 2026 | User |

Schema for reusable work methods, operating procedures, troubleshooting
runbooks, and engineering experience. Procedures should make prerequisites,
steps, verification, risks, and boundaries explicit.

## Observations

- [convention] Every procedure carries a `[procedure_type]` observation.
- [convention] Troubleshooting procedures separate symptom definition, baseline, diagnosis, optimization, verification, and risk review.
- [convention] Update `[last_verified]` when a procedure is used and confirmed or corrected.