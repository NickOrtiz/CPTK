---
description: Runs a scoped engineering ticket from context gathering through implementation, validation, independent review, and handoff. Use when starting work on a prepared ticket in its own branch or worktree.
---

Work on the ticket identified by `$ARGUMENTS`.

## Locate context

First look for:

```text
.cptk/tickets/$ARGUMENTS.md
```

If the exact filename does not exist, inspect `.cptk/tickets/` and identify the intended ticket from the argument.

Also read:

- `CLAUDE.md`
- `.cptk/project-config.md` if present

## Workflow

1. Confirm the repository/branch/worktree state with `git status --short --branch`.
2. Read the ticket completely.
3. Delegate focused codebase investigation to the `researcher` agent when repository discovery is needed.
4. Resolve ordinary implementation questions from existing code and documented project conventions.
5. Escalate only genuinely product-, architecture-, scope-, security-, or destructive-operation decisions.
6. Delegate implementation to the `implementer` agent or perform it in the main session when delegation would add unnecessary overhead.
7. Run `/validate`.
8. If the ticket touches UI, frontend behavior, or visual output, verify it directly yourself in a real browser before continuing: start the project's dev server (use the project's own launch skill, or Claude Code's built-in `run` skill, when available) and exercise the changed behavior. `implementer` and `reviewer` subagents cannot do this — passing type-checks/tests/build is not evidence the UI actually works. Perform this in the main session, not a subagent.
9. Delegate an independent review to the `reviewer` agent.
10. If frontend interaction or UI behavior changed, also delegate to the `accessibility-reviewer` agent.
11. If either reviewer finds material issues, fix them and repeat the relevant validation/review/browser-check cycle.
12. Stop only when the implementation is ready for human inspection.

Do not push code, force-reset, delete branches, or modify shared infrastructure unless explicitly instructed.

Before returning, write the same status to `.cptk/tickets/$ARGUMENTS.status` so `cptk-status` can report progress across parallel worktrees without opening each session:

```text
STATUS: <same value as below>
UPDATED: <current UTC time, ISO 8601>
SUMMARY: <one line>
```

Return:

STATUS: READY FOR HUMAN REVIEW

## Summary
- ...

## Validation
- ...

## Browser verification
- what you exercised directly in a running browser, or why it did not apply

## Review result
- ...

## Human review requested
- ...
