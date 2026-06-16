---
description: Implement a new feature using TDD, then update CHANGELOG.md under [Unreleased] > Added
argument-hint: <feature description in natural language>
---

# /vibe:feature — TDD Feature Implementation

Implement the feature described in `$ARGUMENTS` following the vibe coding workflow: the user is the Product Owner only and never tests code manually.

## Step 1 — Understand the requirement

Read `$ARGUMENTS` carefully. If the requirement is ambiguous on a point that would lead to fundamentally different implementations, ask ONE clarifying question before proceeding. Otherwise, make a reasonable assumption, state it upfront, and continue.

Also read:
- `CLAUDE.md` for project conventions, definition of done, and test location
- Relevant existing source files to understand the current architecture

## Step 2 — Write tests first (red)

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

## Step 3 — Implement (green)

Write the minimum implementation to make the tests pass. Do not over-engineer.

Run the test command after each meaningful change. Iterate until all tests pass.

### Self-correction loop

If tests fail:
- Diagnose the failure
- Fix the code (not the tests, unless the test itself was wrong)
- Re-run
- Repeat up to 3 times before escalating to the user with a precise diagnosis

## Step 4 — Refactor (clean)

With all tests green:
- Remove dead code and unused imports
- Remove debug artifacts (console.log, print, dbg!, etc.)
- Run the lint command (from manifest) and fix any issues
- Re-run tests to confirm still green after lint fixes

## Step 5 — Update CHANGELOG.md

Add an entry under `## [Unreleased]` > `### Added`:

```markdown
- [one-line human-readable description of what was added, from a user perspective]
```

Rules:
- Write for the end user, not the developer ("Users can now export reports as CSV" not "Added exportToCsv() function")
- If `CHANGELOG.md` does not exist, create it with the Keep a Changelog header and the entry
- If `## [Unreleased]` does not exist, add it at the top below the header
- If `### Added` does not exist under [Unreleased], add it

## Step 6 — Report to user

Summarize concisely:
- What was implemented (1–2 sentences)
- Test results: X tests passing, covering [nominal / edge cases / error paths]
- Lint status
- The changelog entry that was added
- Any assumption made in Step 1
