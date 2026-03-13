# CLAUDE.md

## Project documentation
- The entry point is README.md, keep this file relatively small
- Organize the README.md in sections, like
  - Ubiquitous Language
  - Projet setup: how to setup the project
  - Architecture & Algorithms: Global architecture (including diagrams) and algorithms that deserve an explanation
  - Tests: How to run the different kind of tests
  - CI: How to setup and run
  - Miscellaneous
  - Dev tools
- Create a dedicated page linked from README.md if necessary

## Development rules
- Always run the full test suite after any code change (npm test)
- Add tests for every new feature or behaviour change
- Keep task files up to date when requirements evolve
- Document all architectural decisions, subtleties, and constraints in CLAUDE.md (not only in personal memory) — this must be done during each task
- File naming and separation rules (see above) apply to every new file created, without exception — never group multiple exported functions or classes in one file
- Document the functional changes. Create dedicated documentation linked in the README.md if necessary.
- After every task or significant change, proactively update CLAUDE.md and personal memory (MEMORY.md) if any information is now outdated or missing — without waiting for the user to ask
- Update the Ubiquitous Language when a new domain term is introduced. Ask the user more precisions if necessary.

## Layer responsibilities

Projects follow SOLID principles and DDD. Typical layers and their rules:

**`lib/`** — standalone technical utilities with no domain or project knowledge. A file belongs here if it could be extracted as a generic npm/pip package without modification. Usable from any other layer. Examples: filesystem helpers, string formatters, math utilities.

**`domain/`** — core business logic and rules. No infrastructure or framework dependencies. Ports (interfaces) are defined here; concrete implementations live in `infrastructure/`. This is the most valuable and stable layer.

**`config/`** — configuration loading and structural types. Separate from domain because it involves I/O (file reading) and framework-specific parsing, not business logic.

**`infrastructure/`** — concrete adapters implementing domain ports. One subfolder per external system or data source. Adding a new adapter must not require changes to `domain/`.

**`services/`** — application services shared across applications (read models, query helpers, ZIP streaming, resolver factories, etc.). No business logic here. Usable from both CLI and web applications.

**`applications/`** — entry points that consume the shared domain. Multiple applications can coexist (e.g. a web server and a CLI) and all share the same `domain/`, `infrastructure/`, and `server/` layers. Each sub-folder is one deployable application:
- `applications/cli/` — command-line interface; thin wrappers that parse arguments and delegate to domain/server functions
- `applications/web/` (or similar) — the main application started with `npm start`; orchestrates HTTP, WebSocket, SvelteKit, etc.

No business logic belongs in `applications/`. The domain must be usable independently of which application runs it.

**Key rule:** if a piece of logic could belong to two layers, prefer the innermost (closest to domain). Move outward only when the dependency on an external concern is unavoidable.

## Versioning and changelog

Projects use **Semantic Versioning** (semver: `MAJOR.MINOR.PATCH`) and maintain a `CHANGELOG.md` following the [Keep a Changelog](https://keepachangelog.com/) format.

### CHANGELOG.md structure

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
### Changed
### Deprecated
### Removed
### Fixed
### Security
```

### Rules

- Always maintain an `[Unreleased]` section at the top — accumulate changes there during development
- When the user asks to release a version:
  1. Determine the version bump: `MAJOR` (breaking change), `MINOR` (new feature, backward-compatible), `PATCH` (bug fix)
  2. **Before doing anything**, present to the user: the proposed version number, the bump type and its justification, and the full list of changes from `[Unreleased]` — then wait for explicit confirmation
  3. Rename `[Unreleased]` to `[X.Y.Z] - YYYY-MM-DD` (ISO 8601 date)
  4. Add a fresh empty `[Unreleased]` section above it
  5. Add comparison links at the bottom of the file (e.g. `[1.2.0]: https://github.com/org/repo/compare/v1.1.0...v1.2.0`)
  6. Update the version in `package.json` (if applicable)
  7. Commit with message: `chore: release vX.Y.Z`
  8. Create an annotated git tag: `git tag -a vX.Y.Z -m "vX.Y.Z"`
  9. **Do not push the tag** unless the user explicitly asks
- Empty subsections (`### Added`, `### Fixed`, etc.) must be removed before releasing
- Use past tense, imperative mood in entries: "Add X", "Fix Y", "Remove Z"
- Each entry should describe the *what* and *why* for the user, not the implementation detail
- Never edit released sections (only `[Unreleased]` is mutable)

### When to update CHANGELOG.md

Update `[Unreleased]` alongside every code change — include it in the **same commit**, do not batch it at release time:
- New feature or behaviour → `### Added` or `### Changed`
- Bug fix → `### Fixed`
- Breaking change → `### Changed` or `### Removed`, and note in commit/task that MAJOR bump is required
- Security fix → `### Security`

## Task workflow

Tasks are managed as Markdown files in `Tasks/`:
- `Tasks/Todo/` — to do
- `Tasks/InProgress/` — in progress
- `Tasks/Done/` — done

Conventions:
- Files named `NNN-slug.md` (3 digits, e.g. `001-config.md`)
- Content written in English
- Move file to `InProgress/` when starting work
- Only move to `Done/` when explicitly asked by the user — never declare a task "done" or "complete" verbally either; that decision belongs to the user
- Each task must include automated tests
- Update `CLAUDE.md` with any relevant architectural change
