# Project Engineering Guide

## Working principles

Work from repository evidence rather than assumptions.

Prefer the smallest correct implementation that satisfies the ticket and follows existing project patterns.

Do not perform unrelated cleanup, speculative refactors, or architecture changes unless the ticket requires them.

Local and reversible actions are encouraged. Ask before actions that are destructive, hard to reverse, externally visible, or affect shared systems.

Do not:

- push or force-push without explicit approval
- use `git reset --hard` to discard work
- delete branches or worktrees without explicit approval
- bypass validation with `--no-verify`
- modify secrets or `.env` files
- change shared infrastructure unless the ticket requires it and approval is explicit

## Project context

Read `.cptk/project-config.md` for project-specific commands, architecture notes, and validation expectations.

Ticket context lives under `.cptk/tickets/`.

## Implementation behavior

Before editing:

- read the ticket/context
- inspect relevant existing code
- identify existing patterns to follow

During implementation:

- keep scope tight
- preserve established APIs unless change is required
- avoid one-off abstractions
- consider accessibility and responsive behavior for UI work
- add/update tests when the project normally tests the behavior being changed

Before handoff:

- inspect the diff
- run documented validation
- resolve failures caused by the ticket
- perform an independent review
- call out assumptions and manual checks

A changed file set is not the definition of done. The change should be validated and ready for human review.
