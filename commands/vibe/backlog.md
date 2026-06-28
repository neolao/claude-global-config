---
description: Manage the feature backlog — list tasks or add a new item
argument-hint: [feature description to add] | (empty to list all)
---

# /vibe:backlog — Feature Backlog Manager

Manage the feature backlog stored in `.vibe/backlog/`. Each item is a Markdown file named `NNN-slug.md` with a YAML frontmatter `status` field (`todo`, `in_progress`, or `done`).

- **No argument** → list all backlog items with their current status
- **With argument** → create a new backlog item from the description

## Step 1 — Detect mode

If `$ARGUMENTS` is empty or blank: go to **Step 2 — List**.

If `$ARGUMENTS` is non-empty: go to **Step 3 — Compute next number**.

## Step 2 — List backlog items

1. Check if `.vibe/backlog/` exists and contains at least one `*.md` file.
   - If not: report "The backlog is empty — no items in `.vibe/backlog/`." and stop.
2. Collect all `*.md` files in `.vibe/backlog/`, sorted alphabetically (the `NNN-` prefix guarantees numerical order).
3. For each file:
   - Read the YAML frontmatter and extract the `status` value.
   - Read the first `# ` heading as the title.
4. Display a table:

| # | Title | Status |
|---|---|---|
| 001 | User Authentication | `todo` |
| 002 | Export as CSV | `in_progress` |
| 003 | Dark mode | `done` |

Stop here — do not create anything.

## Step 3 — Compute the next number

1. If `.vibe/backlog/` does not exist or contains no `.md` files: the next number is `001`.
2. Otherwise: list all filenames matching `NNN-*.md`, extract the leading numeric prefix from each, find the highest value, increment by 1, and zero-pad to 3 digits.
   - Example: if the highest existing file is `007-dark-mode.md`, the next number is `008`.

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

## Step 6 — Write the backlog file

Create `.vibe/backlog/NNN-slug.md` with this exact structure:

```markdown
---
status: todo
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
