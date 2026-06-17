---
description: Refactor a module or file with plan validation and TDD safety net
argument-hint: <file or module to refactor, and what to improve>
---

# /vibe:refactor — Targeted Refactoring

Refactor the target described in `$ARGUMENTS`. The user is the Product Owner — they do not write or manually test code.

## Step 1 — Understand the target

Read:
- `CLAUDE.md` for project conventions
- `.vibe/index.md` and the relevant `.vibe/modules/[name].md`
- `.vibe/glossary.md` — terminology check
- `.vibe/decisions.md` — existing architectural decisions to respect
- The target file(s)

Identify what makes this code worth refactoring (complexity, naming, duplication, SOLID violations, etc.).

### Terminology check

If any term in `$ARGUMENTS` is a synonym or ambiguous relative to `.vibe/glossary.md`: correct the user before proceeding.

## Step 2 — Confirm safety net

Run the test command (from manifest). All existing tests must pass before starting.

If the target has no tests: **stop**. Refactoring without a safety net is not allowed in vibe coding. Suggest running `/vibe:feature` or `/vibe:fix` to add tests first.

## Step 3 — Plan (must be validated before proceeding)

Present the refactoring plan and **wait for explicit approval**:
- **What will change** — specific improvements (rename, extract, simplify, restructure, etc.)
- **What will NOT change** — behavior must be identical; no feature additions
- **Files touched**
- **Risk** — any part of the refactor that could introduce regressions

Do not write a single line of code until the user approves the plan.

## Step 4 — Refactor

Apply changes incrementally. After each meaningful change:
1. Run the test command — all tests must remain green
2. Run the lint command

### Self-correction loop

If tests break: revert that specific change, diagnose, try an alternative approach — repeat up to 3 times.
If still failing after 3 attempts: skip that change and flag it in the report.

## Step 5 — Sync .vibe/

Run `/vibe:sync` to update the affected module documentation.

## Step 6 — Commit

Stage all modified files and commit:
- Message format: `refactor: [one-line description of what was improved]`

## Step 7 — Report

- What was refactored (1–2 sentences)
- Specific changes applied
- Changes NOT applied (reverted or skipped) with diagnosis
- Test suite status: X tests passing
- Lint status
