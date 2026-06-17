---
description: Implement a new feature using TDD, then update CHANGELOG.md under [Unreleased] > Added
argument-hint: <feature description in natural language>
---

# /vibe:feature — TDD Feature Implementation

Implement the feature described in `$ARGUMENTS` following the vibe coding workflow: the user is the Product Owner only and never tests code manually.

## Step 1 — Understand the requirement

Read `$ARGUMENTS` carefully. Then read:
- `CLAUDE.md` for project conventions, definition of done, and test location
- `.vibe/index.md` if it exists — for a quick overview of modules and patterns
- The relevant `.vibe/modules/[name].md` files for the areas likely involved
- `.vibe/glossary.md` if it exists — to check terminology
- Relevant existing source files to understand the current architecture

### Terminology check

Compare the terms used in `$ARGUMENTS` against `.vibe/glossary.md`:
- If a term in `$ARGUMENTS` is a synonym or near-synonym of a glossary term: stop and correct the user — "Le terme X n'est pas dans le glossaire, voulez-vous dire Y ?"
- If a term is ambiguous (could map to multiple glossary concepts): ask for clarification before proceeding
- If a new term appears that is not in the glossary and is not obviously technical: flag it — "X ne figure pas dans le glossaire. Est-ce un nouveau concept métier à ajouter ?"

Do not proceed with implementation until terminology is aligned.

### Design challenge

Compare the requested feature against the existing architecture (`.vibe/modules/`, `.vibe/index.md`):
- If the feature duplicates something that already exists: flag it and ask if it's intentional
- If the feature contradicts an established pattern (e.g. a new module that should belong to an existing one): challenge the approach before implementing

If the requirement is ambiguous on a point that would lead to fundamentally different implementations: ask ONE clarifying question.

## Step 2 — Plan (must be validated before proceeding)

Present the implementation plan to the user and **wait for explicit approval** before writing any code.

The plan must include:
- **Approach** — what will be implemented, in 2–3 sentences
- **Modules touched** — which existing files/modules will be modified, and why
- **New modules** — if any new file or module needs to be created, justify it
- **Test strategy** — nominal path, edge cases, error paths planned
- **Assumptions** — any assumption made due to ambiguity in the requirement

Do not write a single line of code until the user approves the plan. If the user requests changes to the plan, update it and present it again.

Once the plan is approved: if it includes a non-obvious design decision (a choice between multiple valid approaches, or a deliberate deviation from existing patterns), record it in `.vibe/decisions.md`:

```markdown
## [YYYY-MM-DD] [Short title]
**Contexte :** [what was being built]
**Décision :** [what was decided]
**Raison :** [why]
**Alternatives rejetées :** [what was considered and rejected]
```

## Step 2b — Task decomposition (complex features only)

**A feature is complex if the approved plan contains 2 or more distinct sub-tasks** that can be implemented and tested independently (e.g. "add the data model", "expose the API endpoint", "handle edge case X").

If the feature is complex:
- Create one task per sub-task using the TaskCreate tool
- Task title: short imperative describing the sub-task (e.g. "Add User model", "Expose POST /users endpoint")
- Steps 3–5 below must be executed once per task, in order, before moving to the next

If the feature is simple (single coherent unit of work): skip this step and proceed directly to Step 3.

## Step 3 — Write tests first (red)

_If tasks were created in Step 2b: pick the first pending task with TaskList, mark it `in_progress` with TaskUpdate, then execute Steps 3–5 for that task before moving to the next._

Before writing any implementation:

1. Create or update the appropriate test file for this feature
2. Write tests covering:
   - **Nominal path** — the happy path with valid input
   - **Edge cases** — at least 2 boundary or unusual inputs
   - **Error path** — invalid input, missing data, failure scenarios
3. Run the test command (from manifest) and confirm the new tests **fail** — if they pass without implementation, the tests are wrong; fix them first

Test names must describe behavior, not implementation:
- ✅ `"returns 404 when user does not exist"`
- ❌ `"test getUserById error case"`

## Step 4 — Implement (green)

Write the minimum implementation to make the tests pass. Do not over-engineer.

Run the test command after each meaningful change. Iterate until all tests pass.

### Self-correction loop

If tests fail:
- Diagnose the failure
- Fix the code (not the tests, unless the test itself was wrong)
- Re-run
- Repeat up to 3 times before escalating to the user with a precise diagnosis

## Step 5 — Refactor (clean)

With all tests green:
- Remove dead code and unused imports
- Remove debug artifacts (console.log, print, dbg!, etc.)
- Run the lint command (from manifest) and fix any issues
- Re-run tests to confirm still green after lint fixes

_If tasks were created in Step 2b: mark the current task `completed` with TaskUpdate, then return to Step 3 for the next pending task. Continue until all tasks are completed._

## Step 6 — Update CHANGELOG.md

Add an entry under `## [Unreleased]` > `### Added`:

```markdown
- [one-line human-readable description of what was added, from a user perspective]
```

Rules:
- Write for the end user, not the developer ("Users can now export reports as CSV" not "Added exportToCsv() function")
- If `CHANGELOG.md` does not exist, create it with the Keep a Changelog header and the entry
- If `## [Unreleased]` does not exist, add it at the top below the header
- If `### Added` does not exist under [Unreleased], add it

## Step 7 — Sync .vibe/

If the `.vibe/` directory exists: **invoke the `vibe:sync` skill** using the Skill tool (`skill: "vibe:sync"`) — it will detect changed files via git and update only the affected modules.

If `.vibe/` does not exist: skip — the user can run `/vibe:sync` to generate it.

## Step 8 — Commit

Stage all modified and created files (exclude `.env`, secrets) and commit:
- Message format: `feat: [changelog entry text, written for a developer]`

## Step 9 — Report to user

Summarize concisely:
- What was implemented (1–2 sentences)
- Test results: X tests passing, covering [nominal / edge cases / error paths]
- Lint status
- The changelog entry that was added
- Any assumption made in Step 1
