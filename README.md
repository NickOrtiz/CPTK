# CPTK — Claude Parallel Ticket Kit

A small, reusable Claude Code toolkit for running engineering tickets with isolated worktrees, repeatable agent roles, and lightweight workflows.

The goal is deliberately simple:

- **Worktree = one ticket**
- **Claude session = one worker**
- **Agent = reusable specialist**
- **Skill = reusable procedure**
- **You = reviewer/integrator**

## Install the personal Claude defaults

From this repository:

```bash
mkdir -p ~/.claude/agents ~/.claude/skills

cp global/agents/*.md ~/.claude/agents/
cp -R global/skills/* ~/.claude/skills/
```

Review `global/settings.json` before copying it. It intentionally grants only read-only git commands plus a baseline of common, local, non-destructive validation commands (lint/typecheck/test/build across a few ecosystems) — nothing that pushes, installs packages, or touches git history.

`git push` (and `git push --force`) are explicitly **denied**, not just left off the allow list. That's a real difference: an unlisted command still prompts for your approval and you can say yes in the moment; a `deny` rule is a hard block that Claude Code refuses outright, and no in-session approval can override it. If you ever want a session to push, do it yourself in a separate terminal, or deliberately remove that rule first — don't expect an approval prompt to offer it as a choice.

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

It refuses to run from inside another ticket's worktree (preventing accidental nesting), and after creating the worktree it auto-detects the project's package manager/build tool (npm/yarn/pnpm, Cargo, Go modules, Poetry/pip) and runs install plus a baseline test pass, so a broken starting state is caught immediately instead of being mistaken for something the ticket broke. A failed baseline step is reported as a warning — the worktree is still created either way.

### Tickets tracked in Teamwork (or another tracker)

If tickets already live in Teamwork, you don't need to duplicate them into markdown. The ticket-context template has a "Teamwork task" field at the top — fill in the task id or URL and leave the rest blank. `/start-ticket` and `/review-ticket` fetch that task's current title, description, and comments via the Teamwork MCP tools and treat them as authoritative; the file's other sections are only for CPTK-specific notes Teamwork doesn't capture (out-of-scope boundaries, validation expectations) or for tickets with no Teamwork task at all. If Teamwork MCP tools aren't available in a given session, it falls back to asking for details instead of guessing.

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

### Grouping several tickets into one branch

A worktree doesn't have to map to exactly one ticket. When a group of tickets are tightly related and likely to touch the same files — several small tickets under one larger issue, a set of changes to a shared tokens/config file — putting them on separate branches just creates merge conflicts to untangle later for no benefit. Use `--multi` to scaffold one worktree/branch for the whole group instead:

```bash
/path/to/cptk/scripts/cptk-ticket --multi ISSUE-B typography TYPO-01 TYPO-02 TYPO-03 --base develop
```

This creates a single branch (`feature/ISSUE-B-typography`) and worktree, with one ticket-context file per id: `.cptk/tickets/ISSUE-B.md` (fill this in as the issue-level overview) plus `.cptk/tickets/TYPO-01.md`, `TYPO-02.md`, `TYPO-03.md`. Work through them one at a time in the same session:

```text
/start-ticket TYPO-01
/start-ticket TYPO-02
/start-ticket TYPO-03
```

Each still gets the full `/start-ticket` treatment — research, implementation, validation, browser check, independent review — and writes its own `.status` file, so `cptk-status` shows every sub-ticket's progress under that one worktree. You lose intra-group parallelism (they run sequentially, not concurrently) but gain a single merge at the end instead of one per ticket — worth it whenever the tickets would otherwise collide.

**Only run one `claude` session per worktree at a time.** Two sessions editing the same worktree concurrently will collide on uncommitted changes and git state — grouping tickets trades parallelism *within* the group for safety, it doesn't let you run them all at once in the same worktree.

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
