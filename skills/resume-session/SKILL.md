---
name: resume-session
description: Run at the start of a fresh session (or after /clear) to auto-load the latest project state. Discovers the project's persistence layers, reads recent changelog entries, summarizes active projects and open items, and shows the user exactly where the last session left off. Invoke when the user says "resume", "where did we leave off", "load the state", "what were we doing", "catch me up", "continue from last time", or anything similar. Companion to `wrap-session`. Works across any project.
metadata:
  version: 1.0.0
  scope: universal
---

# Resume-session

Load the latest state of whatever project you're in. This is the mirror of `wrap-session`: where wrap-session saves, resume-session loads.

## When to use

- Start of a fresh session after a context clear
- Returning to a project after a few days away
- Anytime the user says "where were we" or "catch me up"

## Run order

---

## Step 1 — Discover the project

Figure out which project's context to load. In priority order:

1. **Current working directory** — `pwd` tells you where we are
2. **Obvious project markers** — look for `CLAUDE.md`, `README.md`, `.claude/` directory
3. **If ambiguous, ask** — "Which project do you want to resume? I see A, B, C..."

---

## Step 2 — Read project config

Read these files if they exist (in order):

1. `CLAUDE.md` — project-level rules, constraints, style
2. `.claude/wrap-session.md` — any overrides relevant to state handling
3. `~/.claude/projects/-[escaped-project-path]/memory/MEMORY.md` — primary memory

The escaped path replaces `/` with `-`. Example: `/Users/jakub/work/myproj` → `-Users-jakub-work-myproj`.

---

## Step 3 — Read the latest changelog entries

Find the project's log file (same discovery pattern as wrap-session):

1. `context/wiki/log.md`
2. `CHANGELOG.md` in project root
3. `HISTORY.md`, `log.md` in project root

Read the **last 30-50 entries**. Group by date. Identify:
- What happened in the last session
- Any decisions made
- What was created / moved / refactored
- Any flagged blockers

---

## Step 4 — Scan active projects + open items

From the memory file, pull:

- **Active Projects** section → list of ongoing work
- **Completed Work** (most recent dated block) → what just finished
- **Topic Memory Files** → which ones exist and what they cover
- **Priorities** section → current P1/P2/P3

From the tracker (if exists):

- Open P0/P1 tickets
- Items awaiting external input

---

## Step 5 — Check scheduled tasks

If `mcp__scheduled-tasks__list_scheduled_tasks` is available, run it to show:

- What's running on a schedule
- Last run dates
- Next run dates

This helps the user understand what's happening in the background without them having to ask.

---

## Step 6 — Produce the resume summary

Output a structured briefing like this:

```
## Resuming [Project Name]

### Last session ([date])
[1-2 sentences summarizing the last session's theme]

Key outputs:
- [bullet list of what was created/decided last session, from latest Completed Work block]

### Active projects ([count])
[Short list — one line per active project with current state]

### Open items awaiting action
[Bulleted list of committed deliverables, blockers, pending approvals]

### Running automations
[Scheduled tasks + next-run dates, if any]

### Top 3 priorities
[From the memory file or tracker P1 items]

---

### Where to pick up
[One concrete, actionable sentence — what would make the most natural continuation]
```

### Rules for the "Where to pick up" suggestion

- Be concrete: cite a specific file, decision, or action
- Tie to the most recently flagged blocker or commitment
- If nothing is obviously next, list 2-3 options the user can pick from

---

## Edge cases

**First time working on this project** — no MEMORY.md, no log.md. Just acknowledge: "This looks like a fresh project — no prior state found. What do you want to start with?" Then offer to set up basic persistence (run `wrap-session` at the end, or create initial MEMORY.md).

**Memory file is massive (>500 lines)** — read only the first 100 lines (usually indexing + active projects) plus the latest 30 lines of Completed Work. Note the size in summary.

**Multiple projects look possible** — ask the user which one. Don't guess.

**Log has gaps of weeks** — acknowledge the gap: "Last activity was [date], [N] days ago." Then summarize that session.

**Scheduled tasks list shows failures** — flag any tasks where lastRunAt is older than expected or there are notification errors.

---

## Related skills

- **`wrap-session`** — companion, run at end of session to save state
