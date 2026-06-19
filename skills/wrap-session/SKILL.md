---
name: wrap-session
description: Run before clearing the context window, ending a work session, or compacting. Discovers the current project's persistence layers (memory files, wiki, changelog, tracker), updates them with what happened this session, and produces a structured handoff so a fresh session can pick up seamlessly. Invoke when the user says "wrap up", "close out the session", "save state", "prepare for context clear", "end session", "let's finish", "commit what we did", or any similar phrase that signals the end of active work. Works across any project — auto-detects conventions, respects per-project rules defined in CLAUDE.md or `.claude/wrap-session.md`.
metadata:
  version: 2.0.0
  scope: universal
---

# Wrap-session

Close a work session cleanly. Nothing that matters should live only in the conversation transcript.

Works across any project type: code repos, business workflows, research projects, content operations. Auto-detects what the project has and degrades gracefully when a layer is missing.

## Run order

Use TodoWrite to track progress through the steps. Do Step 1 (audit) fully before any writes. Parallelize Steps 2-5 where possible.

---

## Step 1 — Audit what happened in this session

Build an inventory before touching any files. Scan your memory of the conversation and enumerate:

1. **Files created** — new files, folders, campaign structures, reports, drafts, scripts, data outputs
2. **Files edited** — anything existing that was modified
3. **Decisions made** — what the user approved, rejected, parked, pivoted on
4. **Open deliverables** — what you committed to producing but have not finished
5. **External blockers** — awaiting a person, a service, a review, an approval
6. **Scheduled tasks** — if the MCP `mcp__scheduled-tasks__list_scheduled_tasks` is available, list active tasks. Note any created/updated this session.
7. **New systems introduced** — workflows, integrations, automations that will be referenced again
8. **Metrics gathered** — numbers or insights future sessions should remember

If you run `wrap-session quick`, jump to Step 3 (log only) and Step 7 (summary).
If you run `wrap-session audit`, do Steps 1-2 only (show what WOULD be updated, do not write).

---

## Step 2 — Discover project persistence layers

Every project stores state differently. Detect what this one uses:

### Check in priority order

**Project config:**
- `CLAUDE.md` in project root — read for style rules + project context
- `.claude/wrap-session.md` or `.claude/wrap-session.yaml` — project-specific overrides

**Memory files (Claude Code auto-memory):**
- `~/.claude/projects/-[escaped-project-path]/memory/MEMORY.md` — primary memory file
- `~/.claude/projects/-[escaped-project-path]/memory/*.md` — topic memory files

The escaped path replaces `/` with `-`. Example: `/Users/jakub/work/myproj` → `-Users-jakub-work-myproj`.

**Wiki / docs:**
- `context/wiki/`, `docs/wiki/`, `wiki/`, `docs/`
- Look for `index.md`, `README.md`, `log.md`, `CHANGELOG.md`, `HISTORY.md`

**Tracker / backlog:**
- `context/tracker/BACKLOG.md` + `DONE.md`
- `TODO.md`, `TASKS.md`, `ROADMAP.md` in project root
- GitHub Issues if `.github/` is present and `gh` CLI works

**Version control:**
- Is this a `git` repo? Check for `.git/` directory
- If yes, capture the branch and uncommitted changes for the summary

**Session outputs folder:**
- `output/`, `deliverables/`, `reports/` — note new files created here

Record what you found. Future steps only touch layers that exist.

---

## Step 3 — Update the changelog / log (ALWAYS do this)

This is the one update that always happens, regardless of mode.

Find the project's change log in this priority:
1. `context/wiki/log.md`
2. `CHANGELOG.md` in project root
3. `HISTORY.md` in project root
4. `log.md` in project root

**If none exist:** create `CHANGELOG.md` in the project root with a header and add your first entry.

**Entry format:**

```
## [YYYY-MM-DD] [action] | [description]
```

Where `[action]` is one of: `create`, `update`, `refactor`, `delete`, `move`, `rename`, `decide`, `setup`, `fix`.

One entry per meaningful change from the audit. Examples:
- `## [2026-04-18] create | wrap-session and resume-session skills at user level`
- `## [2026-04-18] decide | chose Monday 7:30 AM cadence for weekly market digest`
- `## [2026-04-18] update | MEMORY.md: closed Vinalies campaign, opened European R&R`

For `wrap-session quick`, only Step 3 and Step 7 run.

---

## Step 4 — Update the project memory file

If a `MEMORY.md` (or `CLAUDE.md` used as memory) exists:

### Find the right sections

Common section names: `Active Projects`, `Current Work`, `In Progress`, `Completed Work`, `Recent Changes`, `Decisions`, `Topic Memory Files`.

### Update patterns

**Active Projects:**
- If an existing project's status changed, update its bullet in place
- If this session started a new project spanning beyond the conversation, add a bullet with "(NEW [date])" marker and include: what it is, where files live, current state
- If a project completed, move it out of Active Projects and into the Completed Work section

**Completed Work:**
- Add a new dated heading at the TOP: `### [YYYY-MM-DD] — [session theme]`
- Under it, use `- ✅ **[Item]:** [description with file paths, numbers, key facts]` bullets
- Be specific. A future Claude cannot find "the report we made" — it needs `output/reports/YYMMDD - Title.html`

**Topic Memory Files section (if present):**
- If Step 5 created a new topic memory file, add a one-line reference: `` - `topic_xyz.md` — [one-line purpose] ``

### Size hygiene

If MEMORY.md is growing past ~400 lines, flag this in the summary and suggest archiving old "Completed Work" blocks to separate topic memory files.

### Rule: only document what actually happened

Every bullet must correspond to something that occurred in the session. Do not pad with aspirational items or pre-existing state.

---

## Step 5 — Create or update topic memory files (selectively)

Create a new topic memory file only when the session introduced:

- A new ongoing system (e.g. `project_awards_database.md`, `project_marketing_calendar.md`)
- A new workflow that will be repeated (e.g. `medal_workflow.md`, `digest_workflow.md`)
- A new user preference that should apply to all future sessions (e.g. `feedback_no_em_dashes.md`)
- A new integration, tool, or automation worth isolating

**Do not** create topic memory files for:
- One-off session outputs
- Content that is already captured in MEMORY.md or the wiki

### Format

```markdown
---
name: [short name]
description: [one line — when to read this]
type: project | feedback | integration | workflow
---

## [Sections as needed]

**Why:** [what this is for]
**How to apply:** [when future sessions should reference this]
```

Store at `~/.claude/projects/-[escaped-project-path]/memory/[filename].md`.

Add a reference to MEMORY.md Topic Memory Files section (Step 4).

---

## Step 6 — Update the tracker (if applicable)

Only touch the tracker if the session actually moved tickets:

- Tickets completed → move from BACKLOG to DONE
- New tickets created from session work → add with proper ID format (read existing tickets to infer format)
- Priority changes → update in place

If unsure whether a ticket should be formal, ask the user before creating one.

For GitHub Issues projects, use the `gh` CLI if permissions allow.

---

## Step 7 — Confirm + produce handoff summary

### Verify (quick sanity check)

- Check modified timestamps on files you edited
- Confirm new files exist at claimed paths
- Note any permission errors or write failures

### Output the summary

Send this structured message to the user:

```
## Session wrap-up complete

### What got persisted
- **Memory:** [one-line summary of MEMORY.md changes, or "no changes"]
- **Changelog:** [count] new entries in [path]
- **Wiki / docs updated:** [list paths or "none"]
- **New topic memory files:** [list or "none"]
- **Tracker changes:** [list or "none"]
- **Scheduled tasks active:** [list running automations]
- **Git status:** [if git project: branch + whether uncommitted changes remain]

### Still open for next session
[Bulleted list of: committed deliverables, awaiting external input, parked decisions]

### Suggested opener for next session
"[One concrete sentence the user can paste into a fresh session to resume seamlessly]"
```

### Rules for the suggested opener

- Be specific with file paths and the next concrete action
- Reference blockers by name where relevant
- Keep it under 30 words

**Bad:** "Continue with the marketing work."
**Good:** "Pick up the European R&R campaign. Board needs to approve the blog draft at `output/campaigns/260417 - European Red & Rosé 2026/260417 - Blog draft SK.md` before Monday's launch."

---

## Project-specific rules (from CLAUDE.md and `.claude/wrap-session.md`)

When writing any content during updates, respect the project's style rules. Read them from:

1. `CLAUDE.md` in project root — most projects put style, tone, and constraint rules here
2. `.claude/wrap-session.md` — project-specific overrides for this skill

Common rules to watch for:
- Banned words / phrases
- Required terminology substitutions
- Company/brand context (legal form, values, things never to claim)
- Language(s) to write in
- Ticket prefix format

If the project has both an SK and EN audience (e.g. bilingual companies), apply the writing rules for both.

---

## Edge cases

**Nothing really happened this session** — still add one log.md entry noting the session's focus. Skip memory updates.

**Session was interrupted mid-task** — prioritize capturing interruption state. Write "paused mid-X" in MEMORY.md, and make the "Still open" section of the summary detailed enough to resume cleanly.

**User already wrapped up earlier and is still working** — if log.md already has today's entries with similar content, consolidate or note "continuing from earlier wrap-up" rather than duplicating.

**Major decisions were made but nothing was written** — document the decisions in MEMORY.md even without file deliverables. Decisions are state too.

**No persistence layers detected at all** — create `CHANGELOG.md` in project root. Offer to scaffold a `MEMORY.md` and basic structure for next time. Summary output is still useful even without files.

**Context is already compacted mid-session** — focus on what's still visible in the current context. Note in the summary that parts of the session may not have been captured.

---

## Modes

- **`wrap-session`** (default) — full 6-step wrap. ~30-60 seconds of work.
- **`wrap-session quick`** — Steps 1, 3, 7 only. Log entry + summary. ~10 seconds.
- **`wrap-session audit`** — Steps 1-2 only. Show what would be updated without writing.

---

## Related skills

- **`resume-session`** — companion skill, run at the start of a fresh session to auto-load the latest state
