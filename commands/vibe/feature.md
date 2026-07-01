---
description: Implement a new feature using TDD, then update CHANGELOG.md under [Unreleased] > Added
argument-hint: <feature description in natural language>
---

# /vibe:feature — TDD Feature Implementation

Implement the feature described in `$ARGUMENTS` following the vibe coding workflow: the user is the Product Owner only and never tests code manually.

## Step 1 — Understand the requirement

### Backlog resolution

Before treating `$ARGUMENTS` as a free-form description, check whether it is a reference to a backlog item.

A backlog reference matches one of these patterns (the **entire** `$ARGUMENTS` must match — not just a prefix):
- A pure number: `3`, `003`, `42`
- A number followed by an optional slug: `003-oauth`, `3-oauth-integration`

Detection rule: `$ARGUMENTS` matches `^\d+(-[\w-]+)?$`.

**If `$ARGUMENTS` is a backlog reference:**

1. Extract the numeric part and normalize it to 3 digits with zero-padding (e.g. `3` → `003`).
2. Search `.vibe/backlog/` for a file whose name starts with that 3-digit prefix: `003-*.md`.
   - If the directory does not exist or no matching file is found: stop and report "No backlog item `NNN` found in `.vibe/backlog/`. Run `/vibe:backlog` to list existing items."
3. Read the file:
   - Extract the title (first `# ` heading).
   - Extract the full `## Description` section.
   - Extract the `## Acceptance Criteria` section.
   - Extract the optional `depends_on` list from the frontmatter.
4. **Dependency check:** if `depends_on` is non-empty, for each dependency number find the file `NNN-*.md` in `.vibe/backlog/` (top level or `done/`) and read its `status`.
   - If ALL dependencies have `status: done`: continue normally.
   - If ANY dependency is NOT done: display a warning listing each blocking item (number, title, status). Then ask the user: "Certaines dépendances ne sont pas encore terminées. Voulez-vous continuer quand même ?" — do not proceed until the user explicitly confirms.
5. Update the frontmatter in the file: replace `status: todo` with `status: in_progress`.
6. Store the resolved backlog file path (e.g. `.vibe/backlog/003-oauth.md`) — it will be needed at Step 8.

Use the extracted title + description + acceptance criteria as the feature brief for all subsequent steps, in place of the raw `$ARGUMENTS` string.

**If `$ARGUMENTS` does not match a backlog reference:** proceed normally — treat `$ARGUMENTS` as the free-form feature description.

---

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

### Duplicate check

Before designing anything, verify the feature does not already exist — even under a different name or phrasing.

Check each of the following sources:

1. **Backlog (all statuses)** — read every `.md` file under `.vibe/backlog/` and `.vibe/backlog/done/` (if they exist). Compare the title and description of each item against the requested feature. Look for semantic overlap, not just identical wording — e.g. "export to CSV" and "download data as spreadsheet" are the same feature.

2. **CHANGELOG.md** — if it exists, scan every bullet under `### Added` across all versions and `[Unreleased]`. Flag any entry that describes the same capability.

3. **Existing modules** — scan `.vibe/modules/` and `.vibe/index.md` for any module or function description that already covers the requested behaviour.

**If a likely duplicate is found in any source:** stop and report it to the user:
- Quote the matching item (its title, status, and source file or changelog version).
- Ask: "Cette fonctionnalité semble déjà exister (`<match>`). S'agit-il vraiment d'une nouvelle feature distincte ?"
- Do not proceed until the user explicitly confirms it is a distinct, new feature.

**If no duplicate is found:** note "No duplicate detected" and continue.

### Design challenge

Compare the requested feature against the existing architecture (`.vibe/modules/`, `.vibe/index.md`):
- If the feature duplicates something that already exists: flag it and ask if it's intentional
- If the feature contradicts an established pattern (e.g. a new module that should belong to an existing one): challenge the approach before implementing

If the requirement is ambiguous on a point that would lead to fundamentally different implementations: ask ONE clarifying question.

## Step 1b — Baseline check

Before planning anything, establish the current state of the project.

1. Run the test suite (from manifest): record how many tests pass and how many fail.
   - If tests fail: note the failures as **pre-existing** — do not fix them now, but list them in Step 2 under Assumptions.
2. If the project has a build step (compiled language, bundler, etc.): run the build command.
   - If the build fails: this is a blocker — report it to the user before proceeding. Do not plan over a broken build.
3. If the project has a run/dev command (server, CLI, script): attempt a quick start to verify it doesn't crash immediately.
   - Run the command, wait 3–5 seconds, then stop it. If it crashes on startup: report the error. If it starts: note that the baseline is healthy.

Record the baseline results — they will be referenced in Step 2 and Step 9.

## Step 2 — Plan (must be validated before proceeding)

Present the implementation plan to the user and **wait for explicit approval** before writing any code.

The user is a Product Owner, not a developer: present the plan in plain, non-technical language. **Never mention file names, class/function/method/variable names, module names, or other implementation details.**

The plan must cover, in a few short sentences:
- **Ce qui va être fait** — what will be built, described functionally
- **Ce que ça touche** (if relevant) — which existing behavior or area of the app is affected, in plain terms, without naming files or modules
- **Ce qui va être testé** — the scenarios that will be verified, phrased as user actions and expected results (the normal case, a couple of edge cases, and what happens when something goes wrong)
- **Comment on va vérifier que ça marche vraiment** — a plain description of how the feature will be exercised for real once built (e.g. "on va lancer l'appli et essayer d'exporter un rapport")
- **Hypothèses** — any assumption made because the request was ambiguous, in plain language; mention if pre-existing test failures were found (Step 1b) without technical detail

Keep the technical plan (exact modules/files touched, new files to create, technical test strategy, runtime verification command and arguments) as your own working notes — it guides the implementation but is not part of what you show the user.

Do not write a single line of code until the user approves the plan. If the user requests changes to the plan, update it and present it again.

Once the plan is approved: if it includes a non-obvious design decision (a choice between multiple valid approaches, or a deliberate deviation from existing patterns), record it as an ADR file `.vibe/decisions/NNN-slug.md`:

- **Number**: same rule as the backlog — scan `.vibe/decisions/*.md` for the highest `NNN` prefix, increment, zero-pad to 3 digits; `001` if the directory is empty or absent (create it).
- **Slug**: kebab-case of the decision's short title (lowercase, `-` separators, no punctuation).
- **Content**:

```markdown
---
date: YYYY-MM-DD
status: accepted
---
# [Short title]
**Context:** [what was being built]
**Decision:** [what was decided]
**Reason:** [why]
**Rejected alternatives:** [what was considered and rejected]
```

ADR files are append-only: NEVER modify or delete an existing file in `.vibe/decisions/`. If a new decision replaces an old one, create the new file and change the old file's frontmatter to `status: superseded by NNN` — that frontmatter line is the only permitted mutation.

## Step 2b — Create task list

Once the plan is approved, create the full task list using TaskCreate before writing any code. **Keep subject names short (≤ 30 chars)** — they appear in the status line.

**For each development sub-task** identified in the plan, create 4 tasks in this order using `addBlockedBy` to chain them:

```
[SubTask A] Write tests      ← no dependency (or blockedBy last task of previous sub-task)
[SubTask A] Implement        ← blockedBy "[SubTask A] Write tests"
[SubTask A] Runtime smoke    ← blockedBy "[SubTask A] Implement"
[SubTask A] Refactor + lint  ← blockedBy "[SubTask A] Runtime smoke"
[SubTask B] Write tests      ← blockedBy "[SubTask A] Refactor + lint"
[SubTask B] Implement        ← blockedBy "[SubTask B] Write tests"
[SubTask B] Runtime smoke    ← blockedBy "[SubTask B] Implement"
[SubTask B] Refactor + lint  ← blockedBy "[SubTask B] Runtime smoke"
...
```

For a **simple feature** (single coherent unit of work), use `Feature` as the sub-task label:

```
[Feature] Write tests
[Feature] Implement        ← blockedBy "[Feature] Write tests"
[Feature] Runtime smoke    ← blockedBy "[Feature] Implement"
[Feature] Refactor + lint  ← blockedBy "[Feature] Runtime smoke"
```

Then append 3 final tasks, blocked by the last refactor task:

```
Update CHANGELOG.md   ← blockedBy last "[...] Refactor + lint"
Update docs           ← blockedBy "Update CHANGELOG.md"
Sync .vibe/           ← blockedBy "Update docs"
Commit                ← blockedBy "Sync .vibe/"
```

If the feature was loaded from a backlog file (detected in Step 1), append one additional task, blocked by "Commit":

```
Update backlog status  ← blockedBy "Commit"
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

Mark the task `completed`, then mark the corresponding "Runtime smoke" task `in_progress`.

## Step 4b — Runtime smoke test

Tests are green — now prove the feature actually works at runtime. Do not assume tests = working.

### Determine how to run the feature

From (in order of priority):
1. The "Runtime verification" plan from Step 2
2. `CLAUDE.md` — look for run/dev/start scripts
3. The project manifest (package.json scripts, Makefile targets, Cargo.toml, pyproject.toml, etc.)
4. Project files: `docker-compose.yml`, `Dockerfile`, a `main.*` entry point, a CLI binary

Derive realistic inputs from the acceptance criteria.

### Execute

Run the feature with realistic inputs.

**If it succeeds:** verify that the output or behavior matches what the acceptance criteria describe. Mark the "Runtime smoke" task `completed`, then mark the "Refactor + lint" task `in_progress`.

**If it fails due to missing configuration** (missing env var, missing config file, missing secret):
1. Identify exactly what is missing from the error output.
2. Create a minimal stub — a `.env.test` with placeholder values, or a stub config file — sufficient to allow the feature to start and exercise the happy path.
3. Re-run with the stub in place.

**Self-correction loop (runtime):**
- If it still fails: diagnose the error, fix the code (not the stub, unless the stub was wrong), re-run.
- Repeat up to **3 attempts** total. After 3 failures: stop and escalate to the user with the exact command run, the exact error output, and what was tried.

Do not mark the task `completed` until runtime execution produces the expected behavior.

## Step 5 — Refactor (clean)

With all tests green:
- Remove dead code and unused imports
- Remove debug artifacts (console.log, print, dbg!, etc.)
- Run the lint command (from manifest) and fix any issues
- Re-run tests to confirm still green after lint fixes
- Re-run the runtime smoke test from Step 4b to confirm the refactor did not break runtime behavior. If the smoke test fails, treat it as a regression: fix before marking the task completed.

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

## Step 6b — Update docs

Mark the "Update docs" task `in_progress`.

If the feature is visible to the end user AND `README.md` contains `vibe:` managed section markers: **invoke the `vibe:docs` skill** using the Skill tool (`skill: "vibe:docs"`) — it refreshes the impacted managed sections (features, usage) and the applicable `docs/` files. The modified files will be part of the commit in Step 8.

Skip (with a one-line explanation in the report) if:
- the change is purely internal (no user-visible behavior), or
- `README.md` has no managed sections — the user can run `/vibe:docs` once to set them up.

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

If the feature was loaded from a backlog file:
- Mark the "Update backlog status" task `in_progress`.
- Create `.vibe/backlog/done/` if it does not exist.
- Move the backlog file: `git mv .vibe/backlog/NNN-slug.md .vibe/backlog/done/NNN-slug.md`
- In the moved file, replace `status: in_progress` with `status: done` in the frontmatter.
- Stage all changes and create a second commit: `chore: close backlog item NNN`
- Mark the "Update backlog status" task `completed`.

## Step 9 — Report to user

Summarize concisely:
- What was implemented (1–2 sentences)
- Test results: X tests passing, covering [nominal / edge cases / error paths]
- Lint status
- The changelog entry that was added
- Any assumption made in Step 1
