---
description: Create a versioned release — bump version, finalize CHANGELOG.md, commit and tag
argument-hint: <version, e.g. 1.2.0> | major | minor | patch
---

# /vibe:release — Release Workflow

Create a versioned release following Semantic Versioning.

## Step 1 — Determine version

**If `$ARGUMENTS` is a version number** (e.g. `1.2.0`): use it directly.

**If `$ARGUMENTS` is `major`, `minor`, or `patch`:**
- Run `git tag --sort=-version:refname | head -1` to find the latest tag
- Bump accordingly (semver rules)

**If no argument:**
- Read `CHANGELOG.md` [Unreleased] section
- Suggest: `patch` if only Fixed entries; `minor` if any Added; `major` if any Removed or breaking changes
- Present the suggestion and **wait for confirmation** before proceeding

## Step 2 — Pre-release checks

1. Run the test command (from manifest) — all tests must pass
2. Run the lint command (from manifest) — must exit 0
3. Check for uncommitted changes — if any exist, stop and warn the user

## Step 3 — Finalize CHANGELOG.md

**Invoke the `vibe:changelog` skill** using the Skill tool (`skill: "vibe:changelog"`, `args: "[version]"`) — to move [Unreleased] entries into the new version section with today's date.

## Step 4 — Bump version in manifest

Update the version field in the project manifest:

| File | Field |
|---|---|
| `package.json` | `"version": "X.Y.Z"` |
| `Cargo.toml` | `version = "X.Y.Z"` |
| `pyproject.toml` | `version = "X.Y.Z"` |
| `pom.xml` | `<version>X.Y.Z</version>` |
| `build.gradle` | `version = 'X.Y.Z'` |
| `composer.json` | `"version": "X.Y.Z"` |
| `Gemfile` / `*.gemspec` | `s.version = 'X.Y.Z'` |

## Step 5 — Commit and tag

1. Stage: `CHANGELOG.md` + manifest file(s)
2. Commit: `chore: release vX.Y.Z`
3. Tag: `git tag vX.Y.Z`

## Step 6 — Report

- Version released: `vX.Y.Z`
- Changes included: N Added, N Fixed, N Changed, etc.
- Tag created: `vX.Y.Z`
- **Do NOT push unless the user explicitly asks**
