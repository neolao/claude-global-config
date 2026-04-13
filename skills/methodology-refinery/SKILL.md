---
name: methodology-refinery
description: Manage a project with subject directories and conversation logs.
---

You are working inside a reflection journal repository. Your job is to maintain the log files faithfully, surface relevant past context when needed, and keep the project structure clean.

## Invocation

The skill is invoked with an optional subject argument:

- `/methodology-refinery <subject-name>` — work on the specified subject
- `/methodology-refinery` (no argument) — ask the user which subject to open before proceeding

If no subject is provided, list existing subjects (directories at the repo root) and ask:

> Which subject would you like to work on? (existing subjects: <list>, or "none yet" if empty)

Wait for the user's answer before doing anything else.

## Project structure

```
<repo-root>/
  <subject-name>/          ← one directory per reflection subject
    README.md              ← brief description of the subject (create if absent)
    SUMMARY.md             ← rolling summary of all discussions (max 100 lines)
    logs/
      YYYY-MM-DD.md        ← one log file per day
```

Subjects are plain directories at the root of the repository. Every subject that has at least one conversation must have a `logs/` subdirectory. Do not create any files outside of these locations.

## On startup — read context

Once the active subject is known:

1. Read `<subject>/README.md` to load the subject brief.
2. Read `<subject>/SUMMARY.md` if it exists — this gives a condensed history of all past discussions.
3. Read the most recent log file(s) in `<subject>/logs/` — at minimum the current day's log if it exists, and the previous session's log if today's is absent or very short.
4. If the subject does not exist yet, create it (see "New subjects" below) before proceeding.

## During conversation — maintain SUMMARY.md

After every exchange, update `<subject>/SUMMARY.md` to incorporate the new information from the current turn.

Rules:
- Organise content thematically, not chronologically — merge related points together.
- Keep the file **strictly under 100 lines** at all times.
- If adding new content would push the file past 100 lines, **re-summarise the existing content first**: compress older entries, merge redundant points, and keep only what remains meaningful, then add the new content.
- Do not copy verbatim exchanges — synthesise ideas, insights, decisions, and recurring themes.
- The file is always a living summary, never an append-only record.

Adapt the headings to what is actually relevant for the subject. Keep sections concise.

## During conversation — write logs

After every exchange (one user message + your response), append both turns to `<subject>/logs/YYYY-MM-DD.md` using today's date. Use the following format:

```markdown
## HH:MM — User

<verbatim user message>

## HH:MM — Claude

<your response>
```

Rules:
- Time is local time, 24-hour clock, format `HH:MM`.
- The date in the filename is the date the session started (do not roll over to the next day mid-session).
- If the file does not exist yet, create it with a top-level heading first: `# YYYY-MM-DD`
- Append only — never rewrite or delete existing log content.
- Write the log entry immediately after composing your response.

## Log file example

```markdown
# 2026-04-13

## 09:14 — User

I've been thinking about whether to change careers.

## 09:15 — Claude

That ambivalence often signals that both options have genuine weight...

## 09:47 — User

I think it's mostly fear of losing stability.

## 09:48 — Claude

Fear of losing stability is worth taking seriously as data, not just as an obstacle...
```

## New subjects

When the user wants to start a new subject that does not yet exist:

1. Create `<subject-name>/` at the repo root. Use lowercase-with-hyphens. Confirm the name with the user if not explicitly stated.
2. Create `<subject-name>/README.md`:

```markdown
# <Subject Name>

**Opened**: YYYY-MM-DD

## About

<One or two sentences — ask the user or draft from first message and confirm.>
```

3. Create `<subject-name>/SUMMARY.md` with initial content derived from the first message:

```markdown
# Résumé — <Subject Name>

## Thèmes principaux

- <draft from first message>
```

4. Create `<subject-name>/logs/` (add `.gitkeep` if needed to make it trackable in git).
5. Proceed with the normal session flow.

## Navigation and recall

If the user asks about past sessions:
- List available log files in `<subject>/logs/` sorted by date.
- Read only the requested file(s) — do not load all logs at once.

## Multi-subject sessions

If the user shifts to a different subject mid-session, start writing to the new subject's log from that point forward.

## What NOT to do

- Do not create a global index file at the repo root.
- Do not summarise or rewrite past log entries — they are an immutable record.
- Do not rename subjects or log files.
- Do not add YAML frontmatter to log files.
- Do not let `SUMMARY.md` exceed 100 lines — re-summarise proactively before that limit is reached.
