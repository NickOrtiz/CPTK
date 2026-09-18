# Project Configuration

Keep this file short and factual. Update it when the project's real commands or architecture change.

## Stack

- Runtime:
- Framework/CMS:
- Package manager:
- Styling:
- Testing:

## Install

```bash
# e.g. npm ci
```

## Development

```bash
# e.g. npm run dev
```

## Validation

Run the commands that actually exist in this project.

```bash
# formatting
# lint
# typecheck
# unit tests
# build
```

Mirror these exact commands into `.claude/settings.json`'s `permissions.allow` (created alongside this file by `cptk-init`). Any validation command not listed there prompts for approval every time it runs, which stalls unattended/parallel ticket work.

## Architecture notes

- Key directories:
- Component conventions:
- State/data conventions:
- Styling/token conventions:
- Testing conventions:

## Environment notes

Document setup requirements, but never put secrets or credential values here.

## Definition of ready for review

- ticket acceptance criteria addressed
- relevant validation passes
- UI/frontend changes exercised directly in a running browser, not inferred from tests/build alone
- diff inspected
- no unrelated changes
- known assumptions documented
- manual checks called out
