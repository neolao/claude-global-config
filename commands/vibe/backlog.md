---
description: Manage the feature backlog — list tasks or add one or several items
argument-hint: [feature description to add] | (empty to list all)
---

# /vibe:backlog — Feature Backlog Manager

Manage the feature backlog stored in `.vibe/backlog/`. Each item is a Markdown file named `NNN-slug.md` with a YAML frontmatter `status` field (`todo`, `in_progress`, or `done`).

- **No argument** → list all backlog items with their current status
- **Single-line argument** → create one new backlog item from the description
- **Multi-item argument** → create multiple backlog items (one per detected item)

## Step 1 — Detect mode

If `$ARGUMENTS` is empty or blank: go to **Step 2 — List**.

If `$ARGUMENTS` is non-empty:
- Inspect the structure of `$ARGUMENTS`:
  - **From-review mode**: the argument expresses the intent to create tasks from a previous review — e.g. "from review", "from last review", "depuis le review", "from vibe:review", "from the review findings", or any close variant. Go to **Step 2c — From-review creation**.
  - **Batch mode**: the argument contains multiple distinct items — i.e., a newline-separated list, a bulleted list (`-` or `*` prefixes), or a numbered list (`1.`, `2.`, …). Each line/entry represents a separate backlog item to create. Go to **Step 2b — Batch creation**.
  - **Single mode**: the argument is a plain prose description (possibly multi-sentence but not a list). Go to **Step 3 — Compute next number**.

## Step 2c — From-review creation

Look in the current conversation context for the most recent `/vibe:review` report. It contains sections like "Applied fixes", "Remaining findings" (High / Medium / Low), and "Test status after fixes".

Extract findings to convert into backlog items:
- **Always include**: all findings listed under "Remaining findings" (High, Medium, and Low) — these are issues the review did not auto-fix.
- **Skip**: findings listed under "Applied fixes" — they are already resolved.
- If no `/vibe:review` output is found in the conversation: stop and report "No review output found in the current conversation. Run `/vibe:review` first, or provide a list of items directly."

For each extracted finding, compose a short item description (one line) that captures the issue and its location (file, function) if mentioned.

Then treat this list exactly like a batch argument: go to **Step 2b — Batch creation** with this list.

## Step 2b — Batch creation

Parse `$ARGUMENTS` into an ordered list of item descriptions. Rules:
- Strip leading list markers (`-`, `*`, `1.`, `2.`, …) from each entry.
- Discard blank lines.
- Each non-empty entry is treated as an independent item description (equivalent to calling the skill once per item in single mode).

Before creating any file, display a preview table of all items that will be created:

| # (preview) | Title (preview) |
|---|---|
| 001 | First derived title |
| 002 | Second derived title |
| … | … |

Then create each item in order by applying **Steps 3 → 6** for each entry, using the description of that entry as the argument. The number assigned at Step 3 must be re-computed after each file is written (so each new file gets the correct next number even if files already existed).

After all items are created, go to **Step 7b — Batch report**.

## Step 2 — List backlog items

1. Check if `.vibe/backlog/` exists and contains at least one `*.md` file at the top level (not in `done/`).
   - If not: report "The backlog is empty — no active items in `.vibe/backlog/`." and stop.
2. Collect all `*.md` files directly in `.vibe/backlog/` (exclude the `done/` subfolder), sorted alphabetically.
3. For each file:
   - Read the YAML frontmatter and extract the `status` value and optional `depends_on` list.
   - Read the first `# ` heading as the title.
   - Compute blocked status: if `depends_on` is non-empty, for each dependency number find the file `NNN-*.md` in `.vibe/backlog/` (top level or `done/`) and read its `status`. Collect the numbers whose status is NOT `done` — these are the current blockers.
4. Display a table with a "Bloqué par" column:
   - If no unmet dependencies: show `—`
   - If there are blockers: show the blocker numbers (e.g. `⚠ 002, 003`)

| # | Title | Status | Bloqué par |
|---|---|---|---|
| 002 | Export as CSV | `todo` | — |
| 003 | Dark mode | `in_progress` | ⚠ 002 |
| 004 | Light theme | `todo` | — |

If `.vibe/backlog/done/` contains files, append a note: "N item(s) done — see `.vibe/backlog/done/`."

Stop here — do not create anything.

## Step 3 — Compute the next number

1. If `.vibe/backlog/` does not exist or contains no `.md` files (**including in `done/`**): the next number is `001`.
2. Otherwise: list all filenames matching `NNN-*.md` in **both** `.vibe/backlog/` and `.vibe/backlog/done/`, extract the leading numeric prefix from each, find the highest value, increment by 1, and zero-pad to 3 digits. Scanning `done/` is mandatory — otherwise, once every active item is completed and moved, numbering would restart at `001` and collide with done items.
   - Example: if the highest existing file is `done/007-dark-mode.md`, the next number is `008`.

## Step 4 — Derive title and slug

From `$ARGUMENTS`:

1. **Title**: extract or infer a concise, descriptive title (3–7 words, title case). If `$ARGUMENTS` is already short and unambiguous, use it directly; otherwise summarize it.
2. **Slug**: convert the title to kebab-case — lowercase, words separated by `-`, remove punctuation and special characters.
   - Example: "User Authentication via OAuth2" → `user-authentication-via-oauth2`
3. **Filename**: `NNN-slug.md` (e.g. `008-user-authentication-via-oauth2.md`).

## Step 5 — Generate acceptance criteria

From `$ARGUMENTS`, derive 2–4 acceptance criteria:
- Each criterion must be specific, observable, and independently testable.
- Write from the user's or system's perspective: "User can…", "System returns…", "Page displays…".
- Avoid vague criteria such as "works correctly" or "is fast" — make them falsifiable.

## Step 5b — Detect dependencies

Check whether `$ARGUMENTS` explicitly or implicitly references another existing backlog item as a prerequisite:
- If `$ARGUMENTS` mentions an item by number (e.g. "after 003", "depends on item 5") or by a title matching an existing backlog file: include those item numbers in `depends_on`.
- List all `.vibe/backlog/*.md` files and check if any is clearly a prerequisite based on the description.
- If uncertain: do not add any dependency (leave `depends_on` absent).

If dependencies are found: store them as 3-digit zero-padded numbers (e.g. `[003, 005]`).

## Step 6 — Write the backlog file

Create `.vibe/backlog/NNN-slug.md` with this exact structure:

```markdown
---
status: todo
depends_on: [003, 005]   # include only if dependencies were found in Step 5b; omit this line entirely if none
---
# [Title]

## Description
[What needs to be built and why — elaborated from $ARGUMENTS in 2–4 sentences]

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3 — if applicable]
- [ ] [Criterion 4 — if applicable]

## Notes
[Relevant constraints, technical context, or open questions inferred from $ARGUMENTS. Write "None." if nothing to add.]
```

## Step 7 — Report

Display:
- File created: `.vibe/backlog/NNN-slug.md`
- Title: [title]
- Acceptance criteria: N generated
- Next steps: run `/vibe:feature NNN` or `/vibe:feature NNN-slug` to implement it

## Step 7b — Batch report

Display a summary table of all items created:

| # | File | Title | Criteria |
|---|---|---|---|
| 001 | `.vibe/backlog/001-slug.md` | First Title | 3 |
| 002 | `.vibe/backlog/002-slug.md` | Second Title | 2 |
| … | … | … | … |

Then display:
- Total items created: N
- Next steps: run `/vibe:feature NNN` for any item to start implementing it, or `/vibe:backlog` to review the full backlog.
