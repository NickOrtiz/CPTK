# CPTK — Claude Parallel Ticket Kit

A small, reusable Claude Code toolkit for running engineering tickets with isolated worktrees, repeatable agent roles, and lightweight workflows.

The goal is deliberately simple:

- **Worktree = one ticket**
- **Claude session = one worker**
- **Agent = reusable specialist**
- **Skill = reusable procedure**
- **You = reviewer/integrator**

Start small. Use the scripts manually before adding more automation.

## Install the personal Claude defaults

From this repository:

```bash
mkdir -p ~/.claude/agents ~/.claude/skills

cp global/agents/*.md ~/.claude/agents/
cp -R global/skills/* ~/.claude/skills/
```

Review `global/settings.json` before copying it. It intentionally grants only read-only git commands plus a baseline of common, local, non-destructive validation commands (lint/typecheck/test/build across a few ecosystems) — nothing that pushes, installs packages, or touches git history.

```bash
cp global/settings.json ~/.claude/settings.json
```

If you already have `~/.claude/settings.json`, merge it instead of overwriting it.

## Initialize a project

From a project repository:

```bash
/path/to/cptk/scripts/cptk-init
```

This creates:

```text
CLAUDE.md
.cptk/
  project-config.md
  tickets/
.claude/
  settings.json
```

and adds a `.gitignore` entry for `.cptk/tickets/*.status` (see [Check active worktrees](#check-active-worktrees) below).

It will not overwrite existing files.

Edit `CLAUDE.md` and `.cptk/project-config.md` with the real project commands, architecture, and constraints.

### Allow-list your validation commands

`.claude/settings.json` ships with a few example `npm` entries. **Replace them with the project's actual validation commands** (the same ones you documented in `.cptk/project-config.md`) — different package manager, different script names, a Makefile target, `pytest`, `cargo test`, whatever the project really runs.

This matters because of how Claude Code permissions work: a subagent's `permissionMode: acceptEdits` (used by `implementer` and `debugger`) auto-approves file edits, but it does **not** auto-approve arbitrary Bash commands. Any validation command not covered by an `allow` rule — in this file or in `global/settings.json` — stops the session and waits for interactive approval, every time it runs. For one ticket that's a minor nuisance; across several parallel worktrees it means every session stalls waiting on you, which defeats the point of running them unattended.

`global/settings.json` (copied to `~/.claude/settings.json`) ships with a baseline of common cross-ecosystem validation commands (`npm run lint`, `pytest`, `go test ./...`, `cargo test`, etc.) already allow-listed. Treat that as a starting point, not a substitute for the project-specific list in `.claude/settings.json`.

## Create a ticket worktree

```bash
/path/to/cptk/scripts/cptk-ticket ABC-123 hero
```

By default this creates a sibling directory similar to:

```text
../project-ABC-123-hero
```

and a branch:

```text
feature/ABC-123-hero
```

Inside the new worktree it creates:

```text
.cptk/tickets/ABC-123.md
```

Fill in the ticket context, then:

```bash
cd ../project-ABC-123-hero
claude
```

Inside Claude:

```text
/start-ticket ABC-123
```

Repeat for other tickets to run several in parallel, each in its own terminal/session — that's the whole point of CPTK.

## Check active worktrees

```bash
/path/to/cptk/scripts/cptk-status
```

Beyond `git worktree list` and the current worktree's git state, this also aggregates every ticket's `STATUS:` result across **all** worktrees for the repo — no need to check each parallel Claude session individually:

```text
Ticket status (all worktrees)
------------------------------
ABC-123        READY FOR HUMAN REVIEW       2026-09-18T14:32:00Z     ../project-ABC-123-hero
               Implemented hero carousel, all validation passing.
ABC-124        CHANGES REQUIRED             2026-09-18T14:40:00Z     ../project-ABC-124-footer
               Reviewer flagged missing focus trap in footer modal.
```

This comes from `.cptk/tickets/<TICKET_ID>.status`, a short `STATUS:`/`UPDATED:`/`SUMMARY:` file that `/start-ticket` and `/review-ticket` write when they finish. Nothing to configure — it's part of those skills' return contract. `cptk-init` gitignores these automatically since they're regenerated scratch state, not something worth preserving in history.

## Clean up a finished worktree

```bash
/path/to/cptk/scripts/cptk-clean ../project-ABC-123-hero
```

The cleanup script refuses to remove a worktree with uncommitted changes unless you explicitly use `--force`.

## Included agents

### researcher
Read-only codebase investigation. Use it to find patterns, dependencies, architectural context, and likely implementation surfaces.

### implementer
Focused implementation role. Makes the smallest correct change and validates it.

### reviewer
Read-only independent review against the ticket, project conventions, and diff.

### debugger
Reproduce → isolate root cause → fix → validate.

### accessibility-reviewer
Read-oriented accessibility review with emphasis on semantics, keyboard behavior, focus, names/roles/states, and responsive interactions.

## Included skills

### /start-ticket
Runs the normal ticket workflow: inspect context, research, implement, validate, independent review, fix findings, hand off. For UI/frontend changes it also drives the app in a real browser directly in the main session before handoff — subagents have no browser access, so passing tests/build is never treated as proof the UI works — and delegates to `accessibility-reviewer` when interaction or markup changed. Writes `.cptk/tickets/<id>.status` on completion (see `cptk-status` above).

### /review-ticket
Performs an independent review against the ticket and current diff. Writes `.cptk/tickets/<id>.status` on completion.

### /validate
Runs the project's documented validation commands and reports failures clearly.

### /integrate
Helps integrate already-reviewed ticket branches. It does not push, force-push, or delete branches.

## Safety defaults

These files encourage Claude to take local, reversible actions while keeping these actions behind human approval:

- pushing code
- force pushing
- destructive Git operations
- deleting branches/worktrees
- changing shared infrastructure
- modifying secrets or environment files
- broad refactors outside ticket scope

## Recommended first-day workflow

Do not automate more yet.

1. Initialize one real project.
2. Create two ticket worktrees.
3. Run one Claude session in each.
4. Let one agent implement while you review the other.
5. Send defects back to the ticket's Claude session instead of fixing them yourself.
6. Adjust these files based on the friction you actually encounter.

That is enough to prove whether the workflow helps.
