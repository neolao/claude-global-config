---
description: Run code quality review using specialized sub-agents defined in CLAUDE.md
argument-hint: [optional: path or file to review — defaults to full codebase]
---

# /vibe:review — Code Quality Review Orchestrator

Run a structured code quality review by invoking the specialized agents declared in `CLAUDE.md`. Each agent focuses on one dimension only.

## Step 1 — Read configuration

Read `CLAUDE.md` and find the `## Review agents` section.
It lists which agents are active for this project and their scope.

If the section is absent: run `review-tests`, `review-naming`, `review-complexity`; add `review-solid` only if the codebase contains classes or interfaces; add `review-architecture` only if `.vibe/` exists; skip `review-ddd` (explicit opt-in required).

## Step 1b — Create task list

Based on the active agents determined in Step 1, create tasks using TaskCreate. Always include the three mandatory agents. Add optional agents only if active. The `Run review-*` tasks are independent (no `blockedBy` between them) — they run in parallel. **Keep subject names short (≤ 30 chars)** — they appear in the status line.

```
Run review-tests              ← no dependency
Run review-naming             ← no dependency
Run review-complexity         ← no dependency
[Run review-solid]            ← no dependency (if active)
[Run review-ddd]              ← no dependency (if active)
[Run review-architecture]     ← no dependency (if active)
Deduplicate and prioritize    ← blockedBy ALL "Run review-*" tasks
Apply fixes                   ← blockedBy "Deduplicate and prioritize"
Sync .vibe/ and commit        ← blockedBy "Apply fixes"
```

## Step 2 — Determine scope and collect files

- If `$ARGUMENTS` is provided: review only the specified path or file.
- If no argument: review the full codebase.

Exclude: `node_modules/`, `vendor/`, `.venv/`, `dist/`, `build/`, `out/`, `target/`, generated files (`// generated`, `# auto-generated`), `*.config.*`, `*.json` (unless it contains logic), migration files.

## Step 3 — Run active agents in parallel

Mark ALL `Run review-*` tasks `in_progress`, then launch every active agent **in parallel** — a single message containing one Agent/Skill call per active agent. They are all read-only, so there is no conflict; running them sequentially only wastes time.

- `Run review-tests` — test relevance and quality: invoke the `review-tests` skill using the Skill tool (`skill: "review-tests"`). The skill runs as a forked subagent (Explore agent). Core principle: **tests must verify observable behaviour, not implementation details** — a test that breaks on refactoring without any behaviour change is a false test. Collect all findings (coverage gaps, relevance issues, quality issues).
- `Run review-naming` — naming issues
- `Run review-complexity` — complexity hotspots
- `Run review-solid` — SOLID violations (if active)
- `Run review-ddd` — DDD alignment (if active)
- `Run review-architecture` — architectural drift: module boundaries, circular deps, layer violations, decisions violated (if active — requires `.vibe/`)

As each agent returns, collect its findings and mark its task `completed`.

## Step 4 — Deduplicate and prioritize

Mark the `Deduplicate and prioritize` task `in_progress`.

- Merge findings that point to the same issue from different angles
- Assign priority:
  - **High** — affects correctness, testability, or maintainability significantly
  - **Medium** — reduces readability or violates a clear principle
  - **Low** — style preference or minor improvement

Mark the task `completed`.

## Step 5 — Apply fixes

Mark the `Apply fixes` task `in_progress`.

Apply High and Medium priority fixes directly — this is vibe coding: the user does not fix code manually.

For each fix:
1. Apply it
2. Run the test command (from manifest) to confirm nothing broke
3. Run the lint command (from manifest)
4. If tests break: revert, diagnose, try an alternative approach — repeat up to 3 times
5. If still failing after 3 attempts: skip that fix and escalate to the user with a precise diagnosis

Do NOT auto-fix Low findings — report them only.

Mark the task `completed`.

## Step 6 — Sync .vibe/ and commit (if fixes were applied)

Mark the `Sync .vibe/ and commit` task `in_progress`.

If any fixes were applied in Step 5:
1. **Invoke the `vibe:sync` skill** using the Skill tool (`skill: "vibe:sync"`) — to update affected module documentation
2. Stage all modified files and commit: `refactor: apply code quality fixes from vibe:review`

Mark the task `completed`.

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
