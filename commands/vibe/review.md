---
description: Run code quality review using specialized sub-agents defined in CLAUDE.md
argument-hint: [optional: path or file to review — defaults to full codebase]
---

# /vibe:review — Code Quality Review Orchestrator

Run a structured code quality review by invoking the specialized agents declared in `CLAUDE.md`. Each agent focuses on one dimension only.

## Step 1 — Read configuration

Read `CLAUDE.md` and find the `## Review agents` section.
It lists which agents are active for this project and their scope.

If the section is absent: run all available agents except `review-ddd` (which requires explicit opt-in).

## Step 2 — Determine scope

If `$ARGUMENTS` is provided: review only the specified path or file.
If no argument: review the full codebase (excluding `node_modules`, `dist`, `build`, `.git`, migration files, generated files).

## Step 3 — Collect files

Gather all source files in scope. Exclude:
- Dependency directories (`node_modules/`, `vendor/`, `.venv/`)
- Build output (`dist/`, `build/`, `out/`, `target/`)
- Generated files (marked with `// generated`, `# auto-generated`, etc.)
- Configuration files (`*.config.*`, `*.json` unless it contains logic)
- Migration files

## Step 4 — Run active agents in order

Invoke each active agent in this order, passing the collected files:

1. `review-coverage` — identifies gaps before reviewing the code itself
2. `review-naming` — naming issues
3. `review-complexity` — complexity hotspots
4. `review-solid` — SOLID violations (if active)
5. `review-ddd` — DDD alignment (if active)

Collect all findings from each agent.

## Step 5 — Deduplicate and prioritize

Before reporting:
- Merge findings that point to the same issue from different angles
- Assign priority to each finding:
  - **High** — affects correctness, testability, or maintainability significantly
  - **Medium** — reduces readability or violates a clear principle
  - **Low** — style preference or minor improvement

## Step 6 — Apply fixes (if findings are High priority)

For High priority findings only:
1. Apply the fix directly
2. Run the test command (from manifest) to confirm nothing broke
3. Run the lint command (from manifest)
4. If tests break after a refactor: revert that specific fix and flag it for manual review instead

Do NOT auto-fix Medium or Low findings — report them only.

## Step 7 — Report to user

Structure the report as follows:

```
## Review summary

Agents run: [list]
Files reviewed: N
Total findings: N (High: N, Medium: N, Low: N)

## Applied fixes (High priority)
[list of what was changed, with file + one-line description]
[or "None" if nothing was auto-fixed]

## Remaining findings

### High
[list — these were not auto-fixable and need attention]

### Medium
[list]

### Low
[list — address when convenient]

## Test status after fixes
[result of test run]
```

Keep the report scannable. Do not dump raw agent output — synthesize it.
