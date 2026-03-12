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

## Task workflow

Tasks are managed as Markdown files in `Tasks/`:
- `Tasks/Todo/` — to do
- `Tasks/InProgress/` — in progress
- `Tasks/Done/` — done

Conventions:
- Files named `NNN-slug.md` (3 digits, e.g. `001-config.md`)
- Content written in English
- Move file to `InProgress/` when starting work
- Only move to `Done/` when explicitly asked by the user
- Each task must include automated tests
- Update `CLAUDE.md` with any relevant architectural change
