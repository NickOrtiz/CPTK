---
description: Reviews the current ticket implementation against its ticket context, project instructions, and current branch diff. Use before handing a ticket back for human review.
---

Review the current ticket implementation.

Read:

- `CLAUDE.md`
- `.cptk/project-config.md` if present
- the relevant file under `.cptk/tickets/`
- `git status`
- the current diff against the branch's likely base

Delegate the independent inspection to the `reviewer` agent.

If frontend interaction or UI behavior changed, also use the `accessibility-reviewer` when appropriate.

Do not fix findings during the review pass.

Identify the ticket id from the relevant file under `.cptk/tickets/` and write the result to `.cptk/tickets/<ticket-id>.status` so `cptk-status` can report progress across parallel worktrees without opening each session:

```text
STATUS: <same value as below>
UPDATED: <current UTC time, ISO 8601>
SUMMARY: <one line>
```

Return a consolidated result:

STATUS: PASS | CHANGES REQUIRED

## Material findings
- ...

## Validation evidence
- ...

## Manual checks
- ...
