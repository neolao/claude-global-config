---
name: methodology-refinery
description: Manage a project with subject directories and conversation logs.
---

You are working inside a reflection journal repository. Maintain log files faithfully, surface relevant past context, and keep the structure clean.

## Invocation

- `/methodology-refinery <subject-name>` — open the specified subject
- `/methodology-refinery` — list existing subjects (root directories), ask which to open, then wait

## Structure

```
<repo-root>/<subject>/
  README.md     ← one or two sentences about the subject
  SUMMARY.md    ← rolling synthesis, strictly under 100 lines
  logs/
    YYYY-MM-DD.md
```

## Startup — load context

1. Read `<subject>/README.md`.
2. Read `<subject>/SUMMARY.md` if it exists. **This is the primary context source — do not read past log files unless SUMMARY.md is absent or the user asks.**
3. If today's log exists, check its last few lines only (to know where to append). Do not re-read it for context.
4. If the subject does not exist, create it (see § New subjects).

## After each exchange — write log

After composing your response, immediately run `date +%H:%M` and append both turns to `<subject>/logs/YYYY-MM-DD.md`:

```
## HH:MM — User

<verbatim user message>

## HH:MM — Claude

<your response>
```

- Create the file with `# YYYY-MM-DD` heading if it does not exist yet.
- Append only — never rewrite existing log content.
- Use today's date for the filename for the entire session.

## After each exchange — update SUMMARY.md

After writing the log, update `<subject>/SUMMARY.md`:

- Organise thematically, not chronologically — merge related points.
- Synthesise ideas, insights, decisions, recurring themes. No verbatim quotes.
- **Strictly under 100 lines.** If new content would exceed the limit, compress and merge existing entries first.

Perform both actions directly in the main thread after composing your response: run `date +%H:%M`, write the log entry, then update SUMMARY.md.

## New subjects

1. Create `<subject>/README.md`, `<subject>/SUMMARY.md`, `<subject>/logs/.gitkeep`.
2. README.md: `# Title`, `**Opened**: YYYY-MM-DD`, `## About` + one or two sentences.
3. SUMMARY.md: `# Summary — Title` + `## Main themes` + first bullet from opening message.
4. Confirm the subject name (lowercase-with-hyphens) if not explicit.

## Navigation and recall

List `<subject>/logs/` sorted by date. Read only the file(s) the user asks for — never load all logs.

## Multi-subject sessions

When the user shifts subjects, write to the new subject's log from that point forward.

## Constraints

- No global index at repo root.
- No YAML frontmatter in log files.
- No renaming of subjects or log files.
- No rewriting past log entries.
