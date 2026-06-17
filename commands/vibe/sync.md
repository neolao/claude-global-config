---
description: Generate or regenerate .vibe/ — living codebase map for Claude context (modules, models, glossary)
argument-hint: [optional: path to scan — defaults to full source]
---

# /vibe:sync — Codebase Map Generator

Generate or fully regenerate the `.vibe/` directory — a structured context map that lets Claude understand the codebase quickly in any new session.

Run this when starting the project for the first time, or after significant growth.

## Step 1 — Read configuration

Read `CLAUDE.md` to identify:
- Source directories (from the Architecture section)
- Stack and project type

If `$ARGUMENTS` is provided: scan only that path.

## Step 2 — Collect source files

Gather all source files from the identified directories. Exclude:
- Tests (`*.test.*`, `*.spec.*`, `test_*.py`, `tests/`, `__tests__/`)
- Dependencies (`node_modules/`, `vendor/`, `.venv/`)
- Build output (`dist/`, `build/`, `out/`, `target/`)
- Generated files (`// generated`, `# auto-generated`)
- Configuration files (`*.config.*`, `*.json` unless it contains logic)
- Migration files

## Step 3 — Group by functional area

Infer functional zones from the directory structure (e.g. `src/auth/`, `src/api/`, `src/domain/`).
If the project is flat, group by file type or responsibility.

## Step 4 — Write `.vibe/modules/[name].md`

For each functional zone, create one file:

```markdown
# Module : [name]
**Rôle :** [one sentence describing what this zone does]
**Fichiers :** `src/auth/index.ts`, `src/auth/middleware.ts`
**Exports :** `signIn(email, password): Promise<Session>`, `AuthMiddleware`
**Dépend de :** `modules/db.md`, `modules/config.md`
```

## Step 5 — Write `.vibe/models.md`

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

## Step 6 — Write `.vibe/glossary.md`

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

## Step 7 — Write `.vibe/index.md`

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

## Step 8 — Report

- N modules documented
- N data models found
- N terms in glossary (N new, N preserved)
- Do NOT print file contents unless asked
