---
name: research
description: Trigger this when the user says "research", "/research", "run research", "do research", "research sweep", or wants to dispatch research work before going away so they return to finished deliverables. Scans the project task tracker for research-candidate tickets, batches them, dispatches parallel agents, auto-files outputs, auto-creates follow-up tickets, and queues everything for review.
---

# Research sweep

Discover research-ready tickets, batch them, dispatch parallel agents, file outputs, create follow-up tickets. The goal is a single invocation that converts "lots of open research work" into "lots of finished deliverables ready for review" while the user is away from the machine.

Part of the **operator loop** skill family: `/research` → `/questions` → `/review` → `/tidy`. See `~/.claude/skills/OPERATOR-LOOP.md`.

## Run modes

- **`/research`** (default) — full scan, fast-confirm plan, parallel dispatch, auto-file, auto-ticket
- **`/research dry`** — show the plan, do not dispatch
- **`/research auto`** — skip confirmation, dispatch immediately (use only when user has just approved a recent plan)
- **`/research single [TICKET-ID]`** — one-off on a single ticket
- **`/research resume`** — pick up from last interrupted sweep state
- **`/research category:[list]`** — filter by ticket category (e.g. `category:seo,mkt`)
- **`/research priority:[list]`** — filter by priority (e.g. `priority:P0,P1`)
- **`/research since:[spec]`** — filter tickets added since (e.g. `since:7d`, `since:2026-04-15`)
- **`/research keyword:[term]`** — filter by free-text keyword

Modes combine: `/research category:seo priority:P0,P1`.

## Prerequisites

### Config discovery (standard pattern)

Load configuration in order; later overrides earlier:

1. **Generic defaults** — this skill file (`~/.claude/skills/research/SKILL.md`)
2. **Project override** — `.claude/research.md` in the project root (version-controlled with the project)
3. **Project memory** — `~/.claude/projects/-[escaped-project-path]/memory/MEMORY.md`

The project override must at minimum declare:
- **Task tracker source** — Notion DB ID + data source ID, OR local tracker path (`context/tracker/BACKLOG.md`), OR GitHub Issues repo
- **Ticket ID format** — e.g. `ACME-[CATEGORY]-###` or `ENG-###`
- **Research trigger keywords** — list of words in ticket titles/descriptions that mark them as research-ready
- **Output path conventions** — where drafts and final reports live
- **Ownership classification** — the closed list of valid owner values

If project override is missing, ask the user once for these fields and offer to save them.

### Ticket ownership (closed list — standardized across all operator-loop skills)

Every ticket created by this skill MUST be assigned one of these five owners:

- **`Claude`** — autonomous execution OK (pure research, pure drafting, read-only tool use)
- **`Claude (post-approval)`** — needs one-line CEO approval then Claude executes via MCP / tools
- **`CEO`** — manual UI work, human meeting, board decision, personal judgment call
- **`External`** — dev, designer, vendor, contractor
- **`[Team member name]`** — specific person on the team (e.g. "Jana", "Dáša")

If a research finding produces a new ticket, the agent must classify its owner. No exceptions.

## Run order

Use `TodoWrite` to track progress. Step 1 runs fully before Step 2; then 3-4 can interleave with async agent callbacks.

---

### Step 1 — Inventory

Query the project's task tracker for research-candidate tickets.

**Filters applied:**
- Status ∈ {Not started, Backlog, New, Open} (names vary by tracker)
- Owner contains `Claude` OR `Claude (post-approval)` OR `Content Lead` OR `Research` (writable by autonomous agent)
- No deliverable URL/link field set yet (ticket hasn't already produced output)
- Title OR description contains research-trigger keyword: *research, audit, analyze, analyse, investigate, benchmark, compare, draft, survey, evaluate, map, scan, review (of external state), deep-dive*
- Not already locked by another running sweep (check state file)
- Apply any CLI filters passed

**Deduplication:** fuzzy-match ticket titles against tickets completed in the last 30 days. If a similar title already has a deliverable, flag as potential duplicate rather than dispatch.

**Output of Step 1:** an in-memory list of candidate tickets with full metadata.

---

### Step 2 — Scope assessment

For each candidate, classify:

| Axis | Values |
|------|--------|
| Data source | web-only \| Shopify \| GSC \| GA4 \| internal-memory \| mixed |
| Publishability | green (public output OK) \| yellow (internal only, filter before publish) \| red (can never publish, research stays internal) |
| Duration bucket | S (<10 min) \| M (10-30 min) \| L (30-60 min) |
| Parallelizable | yes \| no (shares rate-limited resource, must serialize) |
| Dependencies | list of other ticket IDs that should complete first |
| Scope clarity | clear \| ambiguous (needs user clarification before dispatch) |

**Ambiguity detection:** if ticket description is under 15 words, or has no verb indicating what deliverable is expected, mark as ambiguous. Do NOT auto-dispatch ambiguous tickets.

---

### Step 3 — Plan (fast-confirm UX)

Present the plan in one screen. Format:

```
Research sweep — [N] candidates, [B] batches, ~[T] min

Batch 1 (concurrent, [duration]):
 1. [TICKET-ID] [P#] "[title]"  [data-source, duration-bucket]
 2. [TICKET-ID] [P#] "[title]"  [data-source, duration-bucket]
 3. [TICKET-ID] [P#] "[title]"  [data-source, duration-bucket]

Batch 2 (concurrent, runs after batch 1, [duration]):
 4. ...

Ambiguous — needs clarification before dispatch:
 X. [TICKET-ID] "[title]" — reason: [why]

Skipped — possible duplicates:
 Y. [TICKET-ID] "[title]" — may overlap with [other-TICKET-ID] done [date]

Reply: "go" / "go 1,3,5" / "skip 7" / "edit 2 [new-scope]" / "cancel"
```

**Fast-confirm rule:**
- ≤15 tickets in total → single `go` is enough
- >15 tickets → require `go force`
- Any ticket with publishability = red → always require explicit confirmation, even in auto mode

**Concurrency limit:** max 5 agents per batch. If more candidates, split into sequential batches.

---

### Step 4 — Dispatch

For each approved ticket, spawn a background Agent using the standardized prompt template at `~/.claude/skills/research/templates/agent-prompt.md`.

The template requires filling these slots:

```
{{TICKET_ID}}        — e.g. ACME-SEO-055
{{TICKET_TITLE}}     — verbatim from tracker
{{TICKET_DESCRIPTION}} — verbatim from tracker
{{SCOPE}}            — what the deliverable must cover
{{DELIVERABLE_PATH}} — e.g. output/reports/260420 - ACME-SEO-055 - SK directory survey.md
{{PUBLISHABILITY}}   — green / yellow / red
{{PROJECT_RULES}}    — pointer to CLAUDE.md or project writing rules
{{PUBLIC_CONTENT_RULE_PATH}} — if the project has a "public content only" rule, paste its contents
{{OWNERSHIP_LIST}}   — the 5-item closed list above
{{WORD_CAP}}         — default 1500; override per ticket if complex
```

Dispatch in parallel using the Agent tool with `run_in_background: true` for each ticket in the batch. Collect returned agent IDs.

---

### Step 5 — Track

As each agent completion notification arrives:

1. **Quality gates** — verify the deliverable file:
   - Exists at the claimed path
   - ≥400 words (for non-trivial research)
   - Has `## TL;DR` or equivalent summary section
   - Has a sources/citations section with ≥3 URLs
   - Has a `## Recommendations` section
   - Has a `## New tickets suggested` section (may be empty, but must exist)

2. **Publishability sanity check** — if ticket was tagged green (public-publishable), grep the output for banned-pattern list from project's public-content rule. If any match, flag + block auto-filing.

3. **AI-writing sanity check** — run the banned-word patterns from project's writing rules (e.g. `delve`, `leverage`, `landscape`, `tapestry`, `robust`, `comprehensive`, em-dashes over threshold). Flag but do not block.

4. **Update state** — append to `context/sessions/[YYYY-MM-DD] - research sweep state.md` with: ticket ID, deliverable path, quality-gate status, new-ticket count, flags.

5. **Brief the user** — one-line completion notice per ticket with any critical finding that changes priorities.

---

### Step 6 — File + action

For each passing research result:

1. **Move file** — if deliverable is in `output/drafts/`, move to `output/reports/[YYMMDD] - [ticket-id] - [slug].md`.
2. **Update source ticket** — set `Deliverable URL` property (or equivalent link field), status → "In review", owner → CEO, due date → +7 days.
3. **Append to changelog** — one line in `context/wiki/log.md` (or project's log-file equivalent): `## [YYYY-MM-DD] research | [TICKET-ID] [title] → [deliverable-path]`
4. **Create follow-up "Review" ticket** in the tracker:
   - Title: `Review: [original-title]`
   - Description: `Deliverable at [path]. Research sweep [date]. Source ticket [TICKET-ID].`
   - Owner: CEO
   - Priority: match source ticket
   - Due: +7 days
5. **Auto-create new tickets** from the agent's "New tickets suggested" section:
   - Parse each suggestion
   - Each must have a classified owner from the closed 5-item list
   - Each must have a priority (P0/P1/P2/P3)
   - Each must have a category matching the project's taxonomy
   - Each must have a **self-contained body** with all execution detail inlined (steps, copy, links, asset URLs, source paths, approval state). No "see X for detail" pointers — a team member with only Notion access must be able to execute it. Per `feedback_notion_task_self_contained`.
   - Each must be **linked** to related tickets via the tracker's relation fields (Depends on / Project link / the Campaigns dual relation in Notion — typically named "Campaign" or "Linked Tasks"; verify property name before writing). Sibling tickets in the same campaign / initiative / topic get connected. Parent project + campaign get linked.
   - Set status = "Not started"
   - Set Source = "Research sweep [date] / parent [TICKET-ID]"
   - Create in tracker via MCP
6. **Update MEMORY.md** — if the research produced a material finding that changes an active project, add/update the relevant bullet.

---

### Step 7 — Summary

Output to user:

```
Research sweep complete — [timestamp]

Completed:           [N] research tasks
Files produced:      [N] (paths listed)
New tickets created: [N] total
  — [X] for Claude (autonomous)
  — [Y] for Claude (post-approval)
  — [Z] for CEO
  — [W] for External
  — [V] for team members
Priority bumps:      [N] (each listed with old → new)
Pending CEO review:  [N] tickets, due [+7 days]
Failed / blocked:    [N] (each with reason)

Top findings that change strategy:
 1. [finding + ticket link]
 2. [finding + ticket link]
 3. [finding + ticket link]

Suggested next step: /review (queues the CEO reviews)
```

---

## Agent prompt template

Lives at `~/.claude/skills/research/templates/agent-prompt.md`. Used verbatim at dispatch time with slots filled. Must include:

- The verbatim ticket title + description
- Publishability flag with explicit "if green, check output against [public content rule]; if red, do NOT produce publishable output"
- Project writing rules pointer
- Output path + required sections
- The closed ownership list for new-ticket classification
- Instruction to cite every factual claim with a source URL
- Word cap

## Edge cases

- **No candidates found** — end with "No research ready. All caught up." No state file written.
- **Project has no tracker configured** — ask user for tracker location once, save to `.claude/research.md`.
- **Agent crashed / no response after timeout** — mark ticket blocked with reason "research agent failure", surface to user, do not auto-file.
- **Output fails publishability check** — do NOT file, save output to `output/drafts/_blocked/[timestamp]/`, ping user with banned-pattern list.
- **Duplicate output vs existing deliverable** — flag, don't auto-replace; ask user whether to archive old or replace.
- **Network / MCP failure mid-sweep** — save state to resume file; `/research resume` picks up.
- **Session compaction during sweep** — agents are autonomous, they continue; state file is the source of truth on resume.
- **New tickets section empty or malformed** — log as warning, no auto-ticketing for that research; file output normally.
- **Research ticket has NDA / red publishability** — agent still runs, but output stays in `output/drafts/` with `_internal` prefix; no auto-move to `reports/`.

## Integration with other operator-loop skills

- **`/review`** — consumes the "In review" tickets this skill creates
- **`/questions`** — scans deliverable outputs for "Open questions for CEO" sections
- **`/tidy`** — re-balances priorities after big research sweeps shift the backlog
- **`wrap-session`** — captures the state file and changelog entries

## Related

- `~/.claude/skills/OPERATOR-LOOP.md` — family overview
- `~/.claude/skills/wrap-session/` — session close-out
- `~/.claude/skills/resume-session/` — session startup
- The parent skill `superpowers:dispatching-parallel-agents` — the underlying pattern for parallel agent dispatch
