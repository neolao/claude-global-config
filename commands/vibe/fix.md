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

Read relevant source files to locate the root cause before writing any code.

If the bug description is too vague to reproduce deterministically, ask ONE clarifying question before proceeding.

## Step 2 — Reproduce in a test first (red)

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

## Step 3 — Fix the bug (green)

Write the minimum change to make the failing test pass without breaking existing tests.

Run the full test suite after each meaningful change. All tests must remain green — if existing tests break, the fix has a regression; address it before continuing.

### Self-correction loop

If tests fail after the fix:
- Diagnose the failure
- Fix the code
- Re-run the full test suite
- Repeat up to 3 times before escalating to the user with a precise diagnosis

## Step 4 — Clean up

With all tests green:
- Remove any debug artifacts introduced during investigation (console.log, print, dbg!, etc.)
- Run the lint command (from manifest) and fix any issues
- Re-run tests to confirm still green

## Step 5 — Update CHANGELOG.md

Add an entry under `## [Unreleased]` > `### Fixed`:

```markdown
- [one-line human-readable description of what was fixed, from a user perspective]
```

Rules:
- Write for the end user ("Fixed crash when submitting an empty form" not "Fixed null check in handleSubmit()")
- If `CHANGELOG.md` does not exist, create it with the Keep a Changelog header and the entry
- If `## [Unreleased]` does not exist, add it at the top below the header
- If `### Fixed` does not exist under [Unreleased], add it

## Step 6 — Sync .vibe/

If the `.vibe/` directory exists: run `/vibe:sync` — it will detect changed files via git and update only the affected modules.

If `.vibe/` does not exist: skip — the user can run `/vibe:sync` to generate it.

## Step 7 — Report to user

Summarize concisely:
- Root cause of the bug (1 sentence)
- What was changed to fix it (1 sentence)
- The test name that now covers this bug
- Full test suite status: X tests passing
- Lint status
- The changelog entry that was added
