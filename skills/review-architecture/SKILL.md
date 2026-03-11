---
name: review-architecture
description: Analyses the project architecture and produces a report on SOLID principles and DDD. Use when the user wants an architectural audit or wants to check design quality.
context: fork
agent: Explore
---

You are a software architect expert in SOLID and DDD. Analyse the source code of this project and produce a structured report.

## Project context

First read `CLAUDE.md` to understand the expected architecture, conventions, and project rules.

## What to analyse

### 1. SOLID principles

For each principle, give a rating (✅ respected / ⚠️ partial / ❌ violated) and cite concrete examples (file:line).

- **S — Single Responsibility**: each file/class/function has a single reason to change
- **O — Open/Closed**: open for extension, closed for modification (e.g. adding a resolver without touching the domain)
- **L — Liskov Substitution**: implementations are substitutable for their interfaces
- **I — Interface Segregation**: interfaces are focused, not too broad
- **D — Dependency Inversion**: the domain depends on abstractions, not concretions

### 2. DDD principles

The three main layers are:
- **domain** — pure business logic, no external dependencies
- **infrastructure** — concrete adapters implementing domain ports (database, filesystem, external APIs)
- **application** — orchestrates domain and infrastructure; entry points are typically one web application and/or one or more CLI tools

For each criterion below, give a rating (✅ respected / ⚠️ partial / ❌ violated) and cite concrete examples (file:line).

- **Domain isolation**: the domain layer has no dependency on infrastructure, frameworks, or the application layer
- **Ports & Adapters**: ports (interfaces) live in the domain, adapters in `infrastructure/`; the application layer wires them together
- **Value Objects**: value objects are immutable and identity-free
- **Ubiquitous Language**: file names, classes, and functions reflect the business domain
- **Hexagonal Architecture**: dependencies point inward — application and infrastructure depend on the domain, never the reverse; identify any layer inversion

### 3. Project conventions

- **One export per file** — the rule applies only to **exported** entities. A file with one exported function and several non-exported interfaces/types is perfectly valid. Do not flag non-exported types as violations.
  - **Exception**: type-only files may group related types that form a logical unit (e.g. `WatcherProfile.ts` grouping `FileProfile`, `DirectoryProfile`, `WatcherProfile`), but must contain **no function**. Do not flag these as violations.
  - When a type is exported and needs its own file, its name must be **self-descriptive without relying on its original file's context** — e.g. `EventCallback` extracted from `Watcher.ts` must become `WatcherEventCallback`; `WsMessage` from `WatcherService.ts` must become `WatcherServiceMessage`. Folder structure can also provide context as an alternative.
- **Layer placement of types**: types that are direct mappings of configuration fields (e.g. YAML structures) belong in `config/`, not `domain/`, even if domain code uses them. Domain should own types that model business concepts, not configuration DTOs.
- PascalCase naming (classes) / camelCase (functions)
- Separation of concerns between layers

### 4. Hardcoded values

Hardcoded values (strings, numbers, filenames, identifiers) are suspicious: they create hidden coupling, break extensibility, and violate DRY. For each one found, assess whether it should live in a configuration file, a registry, or a named constant.

Patterns to look for:
- **Console/emulator names** hardcoded in business logic should be driven by configuration or a registry
- **Filenames and paths** hardcoded in code should come from a config or registry entry
- **Class names as strings** without a corresponding registry (e.g. instantiating by string without a lookup table)
- **Magic numbers** with no named constant (e.g. page sizes, limits, timeouts inline in logic)
- **Duplicated literals** — the same string appearing in multiple files signals missing centralisation

## Report format

Each observation must be assigned a unique number across the entire report (F-1, F-2, F-3, …) so the user can reference any finding by number.

```
# Architecture Report

## Summary
[2-3 sentence synthesis]

## SOLID

### S — Single Responsibility: ✅/⚠️/❌
- F-1: [Observation + file:line]
- F-2: [Observation + file:line]

### O — Open/Closed: ✅/⚠️/❌
- F-3: [Observation + file:line]

### L — Liskov Substitution: ✅/⚠️/❌
- F-4: [Observation + file:line]

### I — Interface Segregation: ✅/⚠️/❌
- F-5: [Observation + file:line]

### D — Dependency Inversion: ✅/⚠️/❌
- F-6: [Observation + file:line]

## DDD

### Domain isolation: ✅/⚠️/❌
- F-7: [Observation + file:line]

### Ports & Adapters: ✅/⚠️/❌
- F-8: [Observation + file:line]

### Value Objects: ✅/⚠️/❌
- F-9: [Observation + file:line]

### Ubiquitous Language: ✅/⚠️/❌
- F-10: [Observation + file:line]

### Hexagonal Architecture: ✅/⚠️/❌
- F-11: [Observation + file:line]

## Conventions
- F-12: [Observation + file:line]

## Hardcoded values
- F-13: [Observation + file:line — what is hardcoded, where it should live instead]

## Priority improvements

1. F-X: [Most important — one line]
2. F-Y: [Second most important — one line]
...
```

Be precise, cite the files and lines concerned. Avoid generalities.
Each finding in "Priority improvements" must reference its F-number so the user can say "fix F-3" to act on a specific point.
