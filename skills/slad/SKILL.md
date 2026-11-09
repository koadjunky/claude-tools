---
name: slad
description: Leave a trace ("ślad") of the current session in the topic folder's CLAUDE.md so a later session, on any machine, can pick up where this one stopped — what was established, a dated timeline, the files in the folder, tool gotchas and what is still open. Use for /slad and whenever the user asks to "zostaw ślad", "zapisz ślad", "zrób notatkę z sesji w CLAUDE.md", "zapisz rezultat i ślad w CLAUDE.md" or similar.
argument-hint: "[extra instruction, e.g. which file or what to stress]"
---

# slad — session trace in the folder's CLAUDE.md

The user works in one folder per topic (a ticket, a bug, a document, a meeting series). That
folder's `CLAUDE.md` is the topic's logbook and is loaded into every later session started there.
Files in the folder are portable between machines; the agent's own memory (`~/.claude*/`) is not.
So the trace goes into files in the folder, never into agent memory instead.

Extra instruction from the user, if any: `$ARGUMENTS`. It overrides the defaults below.

## 1. Read what is already there

- `./CLAUDE.md` of the topic folder (the working directory unless the user named another one).
- Every `CLAUDE.md` in the parent directories up to the home directory: they hold rules shared
  by sibling topics, and a gotcha already recorded there must not be repeated.
- `ls` the folder, so the file list in the trace is complete.

If `./CLAUDE.md` is tracked in a shared git repository (a team codebase, not a personal notes
folder), stop and ask where the trace should go: a session log does not belong in a team file.

## 2. Collect from this session

| Category | What goes in | Where |
|---|---|---|
| **Established** | Facts, root causes, decisions and the evidence behind them: ids, links, numbers, dates, who decided. Corrections of earlier statements, marked as corrections. | `./CLAUDE.md` |
| **Timeline** | One dated block per session: what was done and every artifact created or changed — ticket keys, comment ids, MR numbers, files, messages sent. | `./CLAUDE.md` |
| **Files** | One line per file in the folder that a later session needs to know: what it is, who it is for, whether it is final. | `./CLAUDE.md` |
| **Gotchas** | Tool and API behaviour learned the hard way: the exact command or field that worked, the error that the wrong one gives. | Topic-specific → `./CLAUDE.md`. Reusable across topics (a CLI quirk, a field id, a way of working the user asked for) → the nearest parent `CLAUDE.md` whose scope fits, e.g. `../../CLAUDE.md`. |
| **Still open** | Next steps, open questions, what is waiting on whom. | `./CLAUDE.md` |

Take only what the session actually established. Do not record guesses as facts; a hypothesis
goes in as a hypothesis, with what would confirm it.

## 3. Write

- **Update, do not append blindly.** Merge into the existing sections. Rewrite a statement this
  session proved wrong, remove "Still open" items that were closed, and keep the file's own
  structure if it has one.
- **Long results go to their own file.** When the session produced a long analysis (more than
  ~40 lines), write it to a separate file in the folder (`investigation.md` unless the user or the
  folder uses another name) and keep in `CLAUDE.md` a pointer and a few lines of summary.
  `CLAUDE.md` is loaded into every session there; it is a map of the topic, not an archive.
- **Absolute dates** (`2026-10-22`), never "today" or "yesterday".
- **No secrets**: no tokens, passwords or capability URLs, even when they appeared in the session.
- Write in the language the folder's `CLAUDE.md` already uses; for a new file, use English.

Skeleton for a new `./CLAUDE.md`:

```markdown
# <TOPIC-KEY> — <one-line subject>

Source: <ticket / page / thread link>

## Files
## Established
## Timeline
### <YYYY-MM-DD>
## Still open
## <tool> notes learned here
```

Save without asking for approval: the user asked for the trace.

## 4. Report

One line per file changed, then one line per section changed in it: what was added, corrected
or removed. Nothing else.
