---
description: Update CHANGELOG.md (Keep a Changelog format) from git history since last tag
argument-hint: [optional: version number to release, e.g. 1.2.0]
---

# /vibe:changelog — CHANGELOG.md Updater

Update `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and [Semantic Versioning](https://semver.org/).

## Step 1 — Read current state

- Read `CHANGELOG.md` if it exists (note current [Unreleased] entries and latest version)
- Run `git tag --sort=-version:refname | head -5` to find recent tags
- Run `git log [LAST_TAG]..HEAD --oneline --no-merges` to get commits since last tag
- If no tags exist: run `git log --oneline --no-merges` for all commits
- Detect repo remote URL: `git remote get-url origin` (needed for compare links)

## Step 2 — Classify commits

Map each commit message to a change category. Use these heuristics:

| Commit pattern | Category |
|---|---|
| `feat:`, `add`, `new`, `implement` | Added |
| `refactor:`, `update`, `change`, `improve`, `migrate` | Changed |
| `deprecate` | Deprecated |
| `remove`, `drop`, `delete` | Removed |
| `fix:`, `bug`, `patch`, `correct`, `repair` | Fixed |
| `security`, `vuln`, `cve`, `auth` | Security |

- Rewrite each commit as a human-readable changelog entry (not a raw commit message)
- Group entries by category
- Discard noise: merge commits, version bumps, `chore: lint`, `chore: deps` unless significant
- If a commit message is ambiguous, use judgment — prefer the user-facing impact

## Step 3 — Determine target version

If `$ARGUMENTS` contains a version number (e.g. `1.2.0`): use it as the release version.

If no argument provided:
- Keep all classified entries under `[Unreleased]`
- Do NOT invent or bump a version number

## Step 4 — Update CHANGELOG.md

### If releasing (version provided in $ARGUMENTS):

1. Replace `## [Unreleased]` with `## [Unreleased]` (empty) + new `## [X.Y.Z] - YYYY-MM-DD` section
2. Move all pending [Unreleased] entries + newly classified commits into the new version section
3. Add/update compare links at the bottom of the file

### If not releasing (no version argument):

1. Add newly classified commits to the `## [Unreleased]` section
2. Do NOT create a new versioned section
3. Preserve all existing versioned sections intact

### If CHANGELOG.md does not exist:

Create it with the full header, an `[Unreleased]` section, and all classified commits in it.

## Step 5 — Report

Tell the user:
- How many commits were processed
- How many were kept vs discarded
- Which categories were populated
- Whether a version was released or entries remain under [Unreleased]
- Do NOT print the full CHANGELOG.md unless asked
