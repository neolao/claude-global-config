---
description: Run code quality review using specialized sub-agents defined in CLAUDE.md
argument-hint: [optional: path or file to review — defaults to full codebase]
---

# /vibe:review — Code Quality Review Orchestrator

Run a structured code quality review by invoking the specialized agents declared in `CLAUDE.md`. Each agent focuses on one dimension only.

## Step 1 — Read configuration

Read `CLAUDE.md` and find the `## Review agents` section.
It lists which agents are active for this project and their scope.

If the section is absent: run `review-coverage`, `review-naming`, `review-complexity`; add `review-solid` only if the codebase contains classes or interfaces; add `review-architecture` only if `.vibe/` exists; skip `review-ddd` (explicit opt-in required).

## Step 2 — Determine scope and collect files

- If `$ARGUMENTS` is provided: review only the specified path or file.
- If no argument: review the full codebase.

Exclude: `node_modules/`, `vendor/`, `.venv/`, `dist/`, `build/`, `out/`, `target/`, generated files (`// generated`, `# auto-generated`), `*.config.*`, `*.json` (unless it contains logic), migration files.

## Step 3 — Run active agents in order

Invoke each active agent in this order, passing the collected files:

1. `review-coverage` — identifies gaps before reviewing the code itself
2. `review-naming` — naming issues
3. `review-complexity` — complexity hotspots
4. `review-solid` — SOLID violations (if active)
5. `review-ddd` — DDD alignment (if active)
6. `review-architecture` — architectural drift: module boundaries, circular deps, layer violations, decisions violated (if active — requires `.vibe/`)

Collect all findings from each agent.

## Step 4 — Deduplicate and prioritize

- Merge findings that point to the same issue from different angles
- Assign priority:
  - **High** — affects correctness, testability, or maintainability significantly
  - **Medium** — reduces readability or violates a clear principle
  - **Low** — style preference or minor improvement

## Step 5 — Apply fixes

Apply High and Medium priority fixes directly — this is vibe coding: the user does not fix code manually.

For each fix:
1. Apply it
2. Run the test command (from manifest) to confirm nothing broke
3. Run the lint command (from manifest)
4. If tests break: revert, diagnose, try an alternative approach — repeat up to 3 times
5. If still failing after 3 attempts: skip that fix and escalate to the user with a precise diagnosis

Do NOT auto-fix Low findings — report them only.

## Step 6 — Sync .vibe/ and commit (if fixes were applied)

If any fixes were applied in Step 5:
1. **Invoke the `vibe:sync` skill** using the Skill tool (`skill: "vibe:sync"`) — to update affected module documentation
2. Stage all modified files and commit: `refactor: apply code quality fixes from vibe:review`

## Step 7 — Report to user

Structure the report as follows:

```
## Review summary

Agents run: [list]
Files reviewed: N
Total findings: N (High: N, Medium: N, Low: N)

## Applied fixes (High + Medium)
[list of what was changed, with file + one-line description]
[or "None" if nothing was auto-fixed]

## Remaining findings

### High
[list — auto-fix failed after 3 attempts; includes diagnosis]

### Medium
[list — auto-fix failed after 3 attempts; includes diagnosis]

### Low
[list — address when convenient]

## Test status after fixes
[result of test run]
```

Keep the report scannable. Do not dump raw agent output — synthesize it.
