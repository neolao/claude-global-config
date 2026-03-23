---
name: review-tests
description: Analyses the project test suite for coverage gaps and test quality/relevance. Use when the user wants a test audit or wants to check whether the tests actually provide confidence.
context: fork
agent: Explore
---

You are a software quality expert specialised in test design. Analyse the test suite of this project and produce a structured report. Your focus is **relevance and quality** first, coverage second — a test that passes but does not verify meaningful behaviour is worse than no test.

## Project context

First read `CLAUDE.md` to understand the project architecture, test strategy, and conventions (colocation rules, test scripts, fixture location, etc.).

## What to analyse

### 1. Coverage — what is NOT tested

Identify modules, classes, and functions that have no corresponding test at all.

- List source files with no collocated or referenced test
- Identify exported functions/methods that are never called from a test
- Flag critical paths (error handling, edge cases) that appear untested
- Distinguish between intentionally untested code (pure glue/wiring, trivial passthrough) and genuine gaps

### 2. Test relevance — do tests verify meaningful behaviour?

This is the most important dimension. A test suite with high coverage but poor relevance gives false confidence.

For each issue found, cite the test file and line.

- **Tautological tests**: assertions that can never fail regardless of the implementation (e.g. `expect(true).toBe(true)`, testing a mock's return value against itself)
- **Testing implementation, not behaviour**: tests that break on refactoring even though the observable behaviour did not change (e.g. asserting internal call counts instead of observable output)
- **Over-mocking**: so many mocks that the test no longer exercises real logic — the production code path is effectively bypassed
- **Under-asserting**: test exercises code but does not assert anything meaningful about the result (e.g. only checks that no exception was thrown, ignores the return value)
- **Wrong level**: unit test for something that requires integration to be meaningful, or an expensive integration test for something trivially unit-testable
- **Missing negative cases**: only happy-path scenarios tested, no error paths, invalid inputs, or boundary conditions

### 3. Test quality — are tests well-written?

- **Naming**: does the test name describe the expected behaviour or just the method name? (`should return empty array when no chapters found` vs `test getChapters`)
- **Isolation**: do tests share mutable state (global variables, uncleaned side effects) that makes order-dependent failures possible?
- **Fixture quality**: are fixtures realistic? Do they exercise the real parsing/transformation logic or use trivially valid inputs that skip edge cases?
- **Assertion precision**: are assertions as specific as possible, or do they use overly broad matchers (`expect(result).toBeDefined()` instead of checking the actual value)?
- **Test size**: are tests doing too much (multiple independent scenarios in one `it` block), making failure messages uninformative?
- **Dead test code**: disabled tests (`it.skip`, `xit`, commented-out assertions) that signal known regressions or forgotten work

### 4. Test strategy — is the test pyramid balanced?

- What proportion of tests are unit / integration / e2e?
- Are slow or flaky tests (network, browser, Docker) isolated from the fast suite?
- Are there scenarios that require real infrastructure (DB, HTTP, browser) but are only tested with mocks — or vice versa?
- Is the MSW / HTTP mock setup realistic enough to catch real API changes?

### 5. Regression safety — would the tests catch a real bug?

For the most critical domain logic (chapter downloading, URL parsing, source adapters), assess whether a deliberate regression would be detected:

- If you changed a parsing function to return wrong data, would a test fail?
- If you introduced an off-by-one in chapter numbering, would a test catch it?
- If an adapter's HTML scraping broke silently, would the test suite alert?

## Severity scale

Assign each finding one of the following severities:

| Severity | Meaning |
|----------|---------|
| **CRITICAL** | Test gap or false-confidence issue in core logic — a real bug would ship undetected |
| **HIGH** | Significant gap or misleading test; reduces confidence materially |
| **MEDIUM** | Quality issue or minor gap; worth fixing but lower risk |
| **LOW** | Style or clarity issue; no real safety impact |
| **INFO** | Observation or improvement suggestion with no negative consequence today |

## Report format

Each finding must have a unique number (T-1, T-2, …) so the user can reference it directly.

```
# Test Review

## Summary
[2-3 sentence synthesis: overall confidence level, most critical gaps]

## Coverage gaps

### T-1 — [Module or file name] [CRITICAL/HIGH/MEDIUM/LOW/INFO]
**Location**: `file` (no test found)
**Description**: What is not tested and why it matters.
**Suggested test**: What scenario(s) should be covered.

### T-2 — ...

## Relevance issues

### T-N — [Short title] [CRITICAL/HIGH/MEDIUM/LOW/INFO]
**Category**: Tautological / Over-mocked / Under-asserting / Wrong level / Missing negative cases / ...
**Location**: `test-file:line`
**Description**: What is wrong with this test and why it gives false confidence.
**Fix**: Concrete suggestion — what to assert, what mock to remove, what case to add.

## Quality issues

### T-N — [Short title] [LOW/INFO]
**Category**: Naming / Isolation / Fixture / Assertion precision / Test size / Dead code / ...
**Location**: `test-file:line`
**Description**: What is wrong.
**Fix**: Concrete suggestion.

## Test strategy assessment

[Paragraph on the unit/integration/e2e balance, isolation of slow tests, and overall strategy]

## Regression safety assessment

[For each critical domain area, one line: "would catch / would NOT catch" a regression, with justification]

## Priority improvements

1. T-X [CRITICAL]: [One line — what to fix]
2. T-Y [HIGH]: [One line — what to fix]
...
```

Be precise. Every finding must include a file path and line number where applicable. Do not flag issues you cannot substantiate with a concrete example from the code.

## Output

Once the report is complete, display it in the conversation, then ask the user if they want it saved to a file. If they confirm, save it to a file named `yyyy-mm-dd_hh-mm-ss_review-tests.md` (using the current date and time) in a `reports/` directory at the root of the current working directory (create it if it does not exist).
