---
description: Runs the project's documented validation commands for the current change and reports failures without hiding or working around them.
---

Validate the current change.

Read validation commands from:

1. `.cptk/project-config.md`
2. `CLAUDE.md`
3. existing package scripts / Makefile / project tooling when the commands are not documented

Prefer targeted checks first, then broader checks when appropriate.

Typical categories:

- formatting
- lint
- type checking
- unit tests
- integration tests
- build
- project-specific static analysis

Rules:

- Do not alter tests merely to make them pass.
- Do not use `--no-verify`, skip flags, or destructive cleanup as shortcuts.
- If an unrelated pre-existing failure prevents a clean run, distinguish it from failures introduced by this ticket.
- Avoid running unusually expensive suites unless the project instructions or change scope warrant them.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "This failure looks pre-existing, I'll skip it" | Confirm it against the base branch before dismissing it — don't assume. |
| "The suite is slow, I'll skip the broader checks" | Run what the ticket's scope and project instructions call for; speed is not a reason to skip documented validation. |
| "`--no-verify` will get this past the hook faster" | Never use skip flags as a shortcut. Fix the underlying failure. |

Return:

STATUS: PASS | FAIL | PARTIAL

## Commands
- `command` — result

## Failures
- concise explanation

## Follow-up
- what should be fixed or manually checked
