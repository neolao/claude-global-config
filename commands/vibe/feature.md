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
- If a term in `$ARGUMENTS` is a synonym or near-synonym of a glossary term: stop and correct the user — "The term X is not in the glossary, did you mean Y?"
- If a term is ambiguous (could map to multiple glossary concepts): ask for clarification before proceeding
- If a new term appears that is not in the glossary and is not obviously technical: flag it — "X is not in the glossary. Is this a new business concept to add?"

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
**Context:** [what was being built]
**Decision:** [what was decided]
**Reason:** [why]
**Rejected alternatives:** [what was considered and rejected]
```

## Step 2b — Create task list

Once the plan is approved, create the full task list using TaskCreate before writing any code. **Keep subject names short (≤ 30 chars)** — they appear in the status line.

**For each development sub-task** identified in the plan, create 3 tasks in this order using `addBlockedBy` to chain them:

```
[SubTask A] Write tests      ← no dependency (or blockedBy last task of previous sub-task)
[SubTask A] Implement        ← blockedBy "[SubTask A] Write tests"
[SubTask A] Refactor + lint  ← blockedBy "[SubTask A] Implement"
[SubTask B] Write tests      ← blockedBy "[SubTask A] Refactor + lint"
[SubTask B] Implement        ← blockedBy "[SubTask B] Write tests"
[SubTask B] Refactor + lint  ← blockedBy "[SubTask B] Implement"
...
```

For a **simple feature** (single coherent unit of work), use `Feature` as the sub-task label:

```
[Feature] Write tests
[Feature] Implement        ← blockedBy "[Feature] Write tests"
[Feature] Refactor + lint  ← blockedBy "[Feature] Implement"
```

Then append 3 final tasks, blocked by the last refactor task:

```
Update CHANGELOG.md   ← blockedBy last "[...] Refactor + lint"
Sync .vibe/           ← blockedBy "Update CHANGELOG.md"
Commit                ← blockedBy "Sync .vibe/"
```

All tasks are created upfront. Do not start coding until the full list is created.

## Step 3 — Write tests first (red)

Pick the first pending "Write tests" task with TaskList and mark it `in_progress` with TaskUpdate.

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

Mark the task `completed`, then mark the corresponding "Implement" task `in_progress`.

## Step 4 — Implement (green)

Write the minimum implementation to make the tests pass. Do not over-engineer.

Run the test command after each meaningful change. Iterate until all tests pass.

### Self-correction loop

If tests fail:
- Diagnose the failure
- Fix the code (not the tests, unless the test itself was wrong)
- Re-run
- Repeat up to 3 times before escalating to the user with a precise diagnosis

Mark the task `completed`, then mark the corresponding "Refactor + lint" task `in_progress`.

## Step 5 — Refactor (clean)

With all tests green:
- Remove dead code and unused imports
- Remove debug artifacts (console.log, print, dbg!, etc.)
- Run the lint command (from manifest) and fix any issues
- Re-run tests to confirm still green after lint fixes

Mark the task `completed`. If there are more pending "Write tests" tasks, return to Step 3 for the next sub-task. Otherwise proceed to Step 6.

## Step 6 — Update CHANGELOG.md

Mark the "Update CHANGELOG.md" task `in_progress`.

Add an entry under `## [Unreleased]` > `### Added`:

```markdown
- [one-line human-readable description of what was added, from a user perspective]
```

Rules:
- Write for the end user, not the developer ("Users can now export reports as CSV" not "Added exportToCsv() function")
- If `CHANGELOG.md` does not exist, create it with the Keep a Changelog header and the entry
- If `## [Unreleased]` does not exist, add it at the top below the header
- If `### Added` does not exist under [Unreleased], add it

Mark the task `completed`.

## Step 7 — Sync .vibe/

Mark the "Sync .vibe/" task `in_progress`.

If the `.vibe/` directory exists: **invoke the `vibe:sync` skill** using the Skill tool (`skill: "vibe:sync"`) — it will detect changed files via git and update only the affected modules.

If `.vibe/` does not exist: skip.

Mark the task `completed`.

## Step 8 — Commit

Mark the "Commit" task `in_progress`.

Stage all modified and created files (exclude `.env`, secrets) and commit:
- Message format: `feat: [changelog entry text, written for a developer]`

Mark the task `completed`.

## Step 9 — Report to user

Summarize concisely:
- What was implemented (1–2 sentences)
- Test results: X tests passing, covering [nominal / edge cases / error paths]
- Lint status
- The changelog entry that was added
- Any assumption made in Step 1
