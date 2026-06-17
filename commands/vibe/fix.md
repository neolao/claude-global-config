---
description: Fix a bug using TDD (reproduce first), then update CHANGELOG.md under [Unreleased] > Fixed
argument-hint: <bug description in natural language>
---

# /vibe:fix — TDD Bug Fix

Fix the bug described in `$ARGUMENTS` following the vibe coding workflow: the user is the Product Owner only and never tests code manually.

## Step 1 — Understand the bug

Read `$ARGUMENTS` carefully. Identify:
- What is the observed (broken) behavior?
- What is the expected (correct) behavior?
- Where in the codebase is the issue likely located?

Read, in order:
- `.vibe/index.md` if it exists — to identify which module the bug likely belongs to
- The relevant `.vibe/modules/[name].md` for that area
- `.vibe/glossary.md` if it exists — to check terminology
- The actual source files to locate the root cause

### Terminology check

Compare the terms used in `$ARGUMENTS` against `.vibe/glossary.md`:
- If a term is a synonym or near-synonym of a glossary term: correct the user before proceeding — "Le terme X n'est pas dans le glossaire, voulez-vous dire Y ?"
- If a term is ambiguous: ask for clarification

Do not proceed with the fix until terminology is aligned.

If the bug description is too vague to reproduce deterministically, ask ONE clarifying question before proceeding.

## Step 2 — Plan (must be validated before proceeding)

Present the fix plan to the user and **wait for explicit approval** before writing any code.

The plan must include:
- **Root cause hypothesis** — what you believe is causing the bug, based on reading the code
- **Module(s) touched** — which files will be modified
- **Test strategy** — how the failing test will reproduce the bug
- **Fix approach** — the minimal change that will make the test pass

Do not write a single line of code until the user approves the plan. If the user requests changes, update and re-present.

## Step 3 — Reproduce in a test first (red)

Before touching any implementation:

1. Write a failing test that captures exactly the bug:
   - The test must **fail** with the current code (proving the bug exists)
   - The test must describe the expected correct behavior
   - Place it in the appropriate existing test file, or create one if needed
2. Run the test command (from manifest) and confirm the new test **fails**
   - If it passes without any fix, the test does not reproduce the bug — revise it

Test name must describe the bug scenario:
- ✅ `"does not crash when input list is empty"`
- ❌ `"bug fix test"`

## Step 4 — Fix the bug (green)

Write the minimum change to make the failing test pass without breaking existing tests.

Run the full test suite after each meaningful change. All tests must remain green — if existing tests break, the fix has a regression; address it before continuing.

### Self-correction loop

If tests fail after the fix:
- Diagnose the failure
- Fix the code
- Re-run the full test suite
- Repeat up to 3 times before escalating to the user with a precise diagnosis

## Step 5 — Clean up

With all tests green:
- Remove any debug artifacts introduced during investigation (console.log, print, dbg!, etc.)
- Run the lint command (from manifest) and fix any issues
- Re-run tests to confirm still green

## Step 6 — Update CHANGELOG.md

Add an entry under `## [Unreleased]` > `### Fixed`:

```markdown
- [one-line human-readable description of what was fixed, from a user perspective]
```

Rules:
- Write for the end user ("Fixed crash when submitting an empty form" not "Fixed null check in handleSubmit()")
- If `CHANGELOG.md` does not exist, create it with the Keep a Changelog header and the entry
- If `## [Unreleased]` does not exist, add it at the top below the header
- If `### Fixed` does not exist under [Unreleased], add it

## Step 7 — Sync .vibe/

If the `.vibe/` directory exists: run `/vibe:sync` — it will detect changed files via git and update only the affected modules.

If `.vibe/` does not exist: skip — the user can run `/vibe:sync` to generate it.

## Step 8 — Report to user

Summarize concisely:
- Root cause of the bug (1 sentence)
- What was changed to fix it (1 sentence)
- The test name that now covers this bug
- Full test suite status: X tests passing
- Lint status
- The changelog entry that was added
