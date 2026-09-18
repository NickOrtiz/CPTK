---
description: Integrates already-reviewed ticket branches into the current integration branch, validates the combined result, and identifies conflicts or cross-ticket regressions.
---

Integrate the branch names supplied in `$ARGUMENTS` into the current branch.

Before changing anything:

1. Show the current branch and working-tree state.
2. Require a clean working tree.
3. Confirm the supplied branches exist.
4. Summarize the proposed merges.

Do not push, force-push, delete branches, or rewrite published history.

Merge one branch at a time. If a conflict occurs:

- stop automatic integration
- inspect the conflict
- explain the competing changes
- resolve only when the correct result is supported by ticket/project context
- otherwise ask for human direction

After integration:

1. Run `/validate`.
2. Review for cross-ticket regressions, duplicated implementation, CSS/state collisions, inconsistent APIs, and unexpected combined behavior.
3. Report the integrated commit/branch state.

Return:

STATUS: INTEGRATED | CONFLICT | VALIDATION FAILED

## Integrated
- ...

## Validation
- ...

## Cross-ticket findings
- ...

## Human review requested
- ...
