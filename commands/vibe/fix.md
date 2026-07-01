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
- If a term is a synonym or near-synonym of a glossary term: correct the user before proceeding — "The term X is not in the glossary, did you mean Y?"
- If a term is ambiguous: ask for clarification

Do not proceed with the fix until terminology is aligned.

If the bug description is too vague to reproduce deterministically, ask ONE clarifying question before proceeding.

## Step 1b — Baseline check

Before planning anything, establish the current state of the project.

1. Run the test suite (from manifest): record how many tests pass and how many fail.
   - If tests fail: note the failures as **pre-existing** — do not fix them now (unless one of them IS the reported bug), but list them in Step 2 under the assumptions.
2. If the project has a build step (compiled language, bundler, etc.): run the build command.
   - If the build fails: this is a blocker — report it to the user before proceeding. Do not plan over a broken build.
3. If the project has a run/dev command (server, CLI, script): attempt a quick start to verify it doesn't crash immediately.
   - Run the command, wait 3–5 seconds, then stop it. If it crashes on startup: report the error. If it starts: note that the baseline is healthy.

Record the baseline results — they will be referenced in Step 2 and Step 9.

## Step 2 — Plan (must be validated before proceeding)

Present the fix plan to the user and **wait for explicit approval** before writing any code.

The user is a Product Owner, not a developer: present the plan in plain, non-technical language. **Never mention file names, class/function/method/variable names, or other implementation details.**

The plan must cover, in a few short sentences:
- **Le problème observé** — what's broken, from a user-visible perspective
- **La cause probable** — why it happens, in plain language (e.g. "l'appli ne vérifie pas qu'un champ est rempli avant de continuer" rather than "missing null check")
- **Ce qui va changer** — what will be fixed, described functionally
- **Comment on va vérifier que c'est corrigé** — the scenario that will be tested, phrased as a user action and the expected result, plus a plain description of how the fix will be exercised for real at runtime (e.g. "on va relancer l'appli et rejouer le scénario qui plantait")
- **Hypothèses** — any assumption made because the report was ambiguous, in plain language; mention if pre-existing test failures were found (Step 1b) without technical detail

Keep the technical analysis (exact root cause in code terms, files/modules to touch, technical test strategy) in your own working notes — it guides the fix but is not part of what you show the user.

Do not write a single line of code until the user approves the plan. If the user requests changes, update and re-present.

## Step 2b — Create task list

Once the plan is approved, create the full task list using TaskCreate before writing any code. **Keep subject names short (≤ 30 chars)** — they appear in the status line.

Create these tasks in order, chaining them with `addBlockedBy`:

```
[Fix] Write failing test   ← no dependency
[Fix] Implement            ← blockedBy "[Fix] Write failing test"
[Fix] Runtime smoke        ← blockedBy "[Fix] Implement"
[Fix] Refactor + lint      ← blockedBy "[Fix] Runtime smoke"
Update CHANGELOG.md        ← blockedBy "[Fix] Refactor + lint"
Sync .vibe/                ← blockedBy "Update CHANGELOG.md"
Commit                     ← blockedBy "Sync .vibe/"
```

All tasks are created upfront. Do not start coding until the full list is created.

## Step 3 — Reproduce in a test first (red)

Mark the `[Fix] Write failing test` task `in_progress` with TaskUpdate.

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

Mark the task `completed`, then mark the `[Fix] Implement` task `in_progress`.

## Step 4 — Fix the bug (green)

Write the minimum change to make the failing test pass without breaking existing tests.

Run the full test suite after each meaningful change. All tests must remain green — if existing tests break, the fix has a regression; address it before continuing.

### Self-correction loop

If tests fail after the fix:
- Diagnose the failure
- Fix the code
- Re-run the full test suite
- Repeat up to 3 times before escalating to the user with a precise diagnosis

Mark the task `completed`, then mark the `[Fix] Runtime smoke` task `in_progress`.

## Step 4b — Runtime smoke test

Tests are green — now prove the bug is actually gone at runtime. Do not assume tests = fixed.

### Determine how to run the scenario

From (in order of priority):
1. The verification scenario from the plan in Step 2
2. `CLAUDE.md` — look for run/dev/start scripts
3. The project manifest (package.json scripts, Makefile targets, Cargo.toml, pyproject.toml, etc.)
4. Project files: `docker-compose.yml`, `Dockerfile`, a `main.*` entry point, a CLI binary

Replay the exact scenario that triggered the bug, with realistic inputs.

### Execute

**If it succeeds:** verify the behavior matches the expected (correct) behavior from Step 1. Mark the `[Fix] Runtime smoke` task `completed`, then mark the `[Fix] Refactor + lint` task `in_progress`.

**If it fails due to missing configuration** (missing env var, missing config file, missing secret):
1. Identify exactly what is missing from the error output.
2. Create a minimal stub — a `.env.test` with placeholder values, or a stub config file — sufficient to allow the scenario to run.
3. Re-run with the stub in place.

**Self-correction loop (runtime):**
- If the bug still manifests or a new error appears: diagnose, fix the code (not the stub, unless the stub was wrong), re-run.
- Repeat up to **3 attempts** total. After 3 failures: stop and escalate to the user with the exact command run, the exact error output, and what was tried.

Do not mark the task `completed` until the runtime scenario behaves correctly.

## Step 5 — Clean up

With all tests green:
- Remove any debug artifacts introduced during investigation (console.log, print, dbg!, etc.)
- Run the lint command (from manifest) and fix any issues
- Re-run tests to confirm still green
- Re-run the runtime smoke test from Step 4b to confirm the cleanup did not break runtime behavior. If it fails, treat it as a regression: fix before marking the task completed.

Mark the task `completed`.

## Step 6 — Update CHANGELOG.md

Mark the `Update CHANGELOG.md` task `in_progress`.

Add an entry under `## [Unreleased]` > `### Fixed`:

```markdown
- [one-line human-readable description of what was fixed, from a user perspective]
```

Rules:
- Write for the end user ("Fixed crash when submitting an empty form" not "Fixed null check in handleSubmit()")
- If `CHANGELOG.md` does not exist, create it with the Keep a Changelog header and the entry
- If `## [Unreleased]` does not exist, add it at the top below the header
- If `### Fixed` does not exist under [Unreleased], add it

Mark the task `completed`.

## Step 7 — Sync .vibe/

Mark the `Sync .vibe/` task `in_progress`.

If the `.vibe/` directory exists: **invoke the `vibe:sync` skill** using the Skill tool (`skill: "vibe:sync"`) — it will detect changed files via git and update only the affected modules.

If `.vibe/` does not exist: skip — the user can run `/vibe:sync` to generate it.

Mark the task `completed`.

## Step 8 — Commit

Mark the `Commit` task `in_progress`.

Stage all modified and created files (exclude `.env`, secrets) and commit:
- Message format: `fix: [changelog entry text, written for a developer]`

Mark the task `completed`.

## Step 9 — Report to user

Summarize concisely:
- Root cause of the bug (1 sentence)
- What was changed to fix it (1 sentence)
- The test name that now covers this bug
- Runtime smoke test result: the scenario replayed and its outcome
- Full test suite status: X tests passing (mention pre-existing failures from Step 1b, if any)
- Lint status
- The changelog entry that was added
