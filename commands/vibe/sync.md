---
description: Sync .vibe/ — generate on first run, incrementally update on subsequent runs
argument-hint: [optional: --full to force full regeneration]
---

# /vibe:sync — Codebase Map Sync

Sync the `.vibe/` directory — a structured context map that lets Claude understand the codebase quickly in any new session.

- **First run** (`.vibe/` absent): full generation
- **Subsequent runs**: incremental — only modules whose source files changed since the last sync are updated
- **`--full`** in `$ARGUMENTS`: force full regeneration regardless

## Step 1 — Read configuration

Read `CLAUDE.md` to identify source directories and stack.

## Step 2 — Determine scope

**If `.vibe/index.md` does not exist** (or `--full` in `$ARGUMENTS`): full mode — collect all source files, skip to Step 4.

**Otherwise — incremental mode:**
1. Read the generation date from `.vibe/index.md` (first line: `> Généré par /vibe:sync le YYYY-MM-DD`)
2. Run `git log --since="YYYY-MM-DD" --name-only --pretty=format: -- <source_dirs>` to list files changed since last sync
3. If no files changed: report "already up to date" and stop
4. Identify which functional zones are affected by the changed files → only those zones will be updated

## Step 3 — Collect source files

Gather source files for the zones to update. Exclude:
- Tests (`*.test.*`, `*.spec.*`, `test_*.py`, `tests/`, `__tests__/`)
- Dependencies (`node_modules/`, `vendor/`, `.venv/`)
- Build output (`dist/`, `build/`, `out/`, `target/`)
- Generated files (`// generated`, `# auto-generated`)
- Configuration files (`*.config.*`, `*.json` unless it contains logic)
- Migration files

## Step 4 — Group by functional area

Infer functional zones from the directory structure (e.g. `src/auth/`, `src/api/`, `src/domain/`).
If the project is flat, group by file type or responsibility.

## Step 5 — Write `.vibe/modules/[name].md`

For each functional zone, create one file:

```markdown
# Module : [name]
**Rôle :** [one sentence describing what this zone does]
**Fichiers :** `src/auth/index.ts`, `src/auth/middleware.ts`
**Exports :** `signIn(email, password): Promise<Session>`, `AuthMiddleware`
**Dépend de :** `modules/db.md`, `modules/config.md`
```

## Step 6 — Write `.vibe/models.md`

In incremental mode: only update entries whose source files are in the changed set.

Collect all key types, interfaces, schemas, and data structures across the codebase.

```markdown
# Modèles de données

## [ModelName]
| Champ | Type | Notes |
|---|---|---|
| id | string | UUID |
| ... | ... | ... |
Défini dans : `src/types/user.ts`
```

## Step 7 — Write `.vibe/glossary.md`

Detect domain term candidates from: class names, type names, interface names, and domain-specific vocabulary in the code.

If `.vibe/glossary.md` already exists: **preserve all existing definitions**, only add newly detected terms.

New terms are added with a minimal entry — definitions are to be refined by the user:

```markdown
# Ubiquitous Language

## [Domain term]
**Définition :** [what this concept means in the domain — fill in]
**Code :** `ClassName`, `functionName()` in `src/...`
**À ne pas confondre avec :** [similar but distinct term, if applicable]
```

## Step 8 — Write `.vibe/index.md`

```markdown
# [PROJECT_NAME] — Codebase index
> Généré par /vibe:sync le YYYY-MM-DD. Relancer pour mettre à jour.

## Modules
- [`modules/auth.md`](.vibe/modules/auth.md) — [one-line description]
- [`modules/api.md`](.vibe/modules/api.md) — [one-line description]
...

## Patterns observés
[Patterns and conventions actually found in the code — not those declared in CLAUDE.md]
```

## Step 9 — Report

- Mode used: full or incremental
- N modules updated (out of N total)
- N data models updated
- N terms in glossary (N new, N preserved)
- Do NOT print file contents unless asked
