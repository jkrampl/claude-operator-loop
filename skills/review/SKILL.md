---
name: review
description: Trigger this when the user says "/review", "run review", "review session", "what needs my review", "pending decisions", or wants a structured session to go through everything awaiting CEO/owner decisions — research deliverables, content drafts, campaign approvals, Shopify/site changes, pending tickets. NOT for PR code review (that uses a separate project-level slash command). Surfaces everything awaiting decision, groups by urgency and depth, applies decisions to trackers.
---

# Review session

Surface everything that needs an owner decision, grouped by urgency and depth. Accept fast responses ("approve 1, 3, 5"; "reject 2 — reason"; "defer 4 — next week"; "comment 6 — [text]") and apply them across the tracker, file system, and integrated tools.

Part of the **operator loop** skill family: `/research` → `/questions` → `/review` → `/tidy`. See `~/.claude/skills/OPERATOR-LOOP.md`.

> **Naming note:** in some projects `/review` is also a built-in slash command for PR code review. This skill is different — it's a generic review-session for content and decisions. If a project defines its own `/review` slash command in `.claude/commands/`, that command takes precedence. In that case, invoke this skill as "review session" or "operator review" instead.

## Run modes

- **`/review`** (default) — full pass: Stage 0 comments-inbox (apply async CEO decisions from tracker comments) + Stage 1 scan pending + surface + await responses
- **`/review inbox`** — Stage 0 only. Scan tracker comments since last run, parse intent, apply changes. No interactive surface. Designed for scheduled daily runs.
- **`/review surface`** — skip inbox, only surface pending items (rarely needed — use when inbox was just run separately)
- **`/review urgent`** — only hard-deadline items
- **`/review quick`** — only items needing <5 min to decide
- **`/review deep`** — only items needing >15 min (plan the deep-work block)
- **`/review category:[list]`** — filter by type (research / content / campaign / technical / operational)
- **`/review since:[spec]`** — only items surfaced since (e.g. `since:7d`)
- **`/review apply [response-text]`** — apply a batch response without re-scanning (use after a review session when pasting answers)

## Prerequisites

### Config discovery (standard pattern)

Load in order:
1. `~/.claude/skills/review/SKILL.md` — this file
2. `.claude/review.md` — project override (required: tracker source, output paths, approval chains)
3. `~/.claude/projects/-[escaped-project-path]/memory/MEMORY.md`

### Required project config

`.claude/review.md` must declare:

- **Tracker source** — Notion DB ID, local tracker path, or GitHub Issues
- **"In review" status value** — how this tracker names tickets awaiting decision
- **Deliverable folders** — which folders to scan for files awaiting review (e.g. `output/reports/`, `output/drafts/`)
- **Approval system** — Paperclip, email, inline-comment, none
- **Downstream-execution rules** — what happens when an item is approved (e.g. approved Shopify write → execute via MCP; approved blog post → schedule publish; approved email → send via platform)
- **Governance gates** — per CLAUDE.md or equivalent, list things that always require explicit CEO approval regardless of owner

## Run order

### Stage 0 — Comments inbox (async CEO decisions)

**Purpose:** before surfacing anything new, process any decisions the CEO already made asynchronously via tracker comments since the last `/review` run. Catches the workflow where the CEO comments on tickets throughout the day; Claude applies the changes next morning.

**Tasks:**

1. **Load last-run timestamp** from `context/sessions/_review-inbox-state.json` (project state file). If file missing, default to 24 hours ago.

2. **Fetch recent comments** from the tracker:
   - Notion: iterate over active tickets (Status != Archived AND != Done), call comment-list on each. Filter to comments `created_time > last_run_timestamp`. Exclude comments authored by the Claude integration ID (to skip self-replies).
   - Paperclip / other approval platforms: same filter if connected.
   - Email / Slack / other async channels: only if project config points to them.

3. **Classify each comment's intent** (the 5 canonical classes):

   | Class | Signal | Action |
   |---|---|---|
   | **Direct decision** | "approve", "rejected", "defer to X", "yes", "no" | Apply to ticket: status change + owner shift per decision |
   | **Pivot** | "we're not doing this, do X instead", "this replaces that" | Archive original with Reflection + create replacement ticket carrying over context |
   | **Amendment** | "also add Y", "and also", "while you're at it" | Update ticket scope + Blocked By, OR create child ticket |
   | **Rule codification** | "from now on...", "we always...", "never do X" | Create `feedback_*.md` memory rule + update CLAUDE.md governance if applicable |
   | **Clarification request** | "need more info on X", "check with Y first", "find out Z" | Update ticket Blocked By + status → Blocked + re-assign as needed |

   If a comment doesn't fit cleanly, flag as "unclassified" and surface to CEO in Stage 1.

4. **Apply actions atomically per ticket.** Each action:
   - Updates tracker via API (patch-page for Status / Reflection / dates)
   - Creates follow-up tickets where needed (via create-page)
   - Creates memory rules where needed (`feedback_*.md`)
   - Posts a reply comment on the original ticket acknowledging what was done ("Acknowledged — archived + created ACME-MKT-XXX; added feedback_zzz.md rule")
   - Logs to `context/sessions/[YYYY-MM-DD] - Review inbox.md`

5. **Update state file** with new last-run timestamp.

6. **Summary to user** (or to scheduled-task output):
   ```
   Comments inbox — [N] comments processed
     — [X] direct decisions applied
     — [Y] pivots (archived + replaced)
     — [Z] amendments
     — [W] rule codifications
     — [V] clarifications
     — [U] unclassified (surfaced in Stage 1)
   New tickets created: [list]
   New rules codified: [list]
   ```

**If no new comments since last run:** skip silently, proceed to Stage 1.

### Stage 1 — Scan all sources

Collect every item awaiting decision from:

1. **Tracker — "In review" tickets** — owner = CEO (or equivalent), status = "In review" / "Awaiting approval" / "Pending"
2. **Tracker — "Awaiting input" tickets** — owner = CEO, status = "Needs input" / "Blocked on CEO"
3. **Deliverable folders** — files in `output/reports/` (or project equivalent) modified in last 30 days with no corresponding "reviewed" marker
4. **Drafts flagged ready** — files in `output/drafts/` containing marker `[REVIEW READY]` or equivalent
5. **Approval system** — pending Paperclip requests or equivalent external approval queue
6. **Integrated tools** — depending on project:
   - Shopify: draft products, unpublished pages, unpublished blog posts modified in last 7 days
   - Website CMS: scheduled-but-unpublished content
   - Email platform: drafted-but-unsent campaigns
   - Ad platforms: pending campaign creatives
7. **Governance-gated items** — anything the project's `CLAUDE.md` flags as requiring board/CEO approval (per the governance rule), check if any ship-ready assets exist in this category without approval

### Step 2 — Classify each item

For each, assign:

| Axis | Values |
|------|--------|
| Urgency | hard-deadline (specific date) \| soft-deadline (week range) \| no-deadline |
| Depth | quick (<5 min to decide) \| medium (5-15 min) \| deep (>15 min, needs focus block) |
| Type | research deliverable \| content draft \| campaign asset \| technical change \| operational decision \| strategic decision |
| Dependency | standalone \| blocks other work (list IDs) \| waits on other decision (list IDs) |
| Executability | approve-only (CEO says yes, done) \| approve-then-execute (CEO approves, Claude or tool acts) \| approve-then-external (needs handoff) |

### Step 3 — Group and sort

Primary grouping — urgent × quick first, in this order:

1. **Urgent + quick** — grab-and-go decisions with hard deadlines, rank by deadline
2. **Urgent + medium/deep** — deadline-critical decisions, time-block suggested
3. **Not urgent + quick** — batch list for fast triage
4. **Not urgent + deep** — defer to scheduled deep-review block, surface only the top 3-5

### Step 4 — Present

Format:

```
# Review session — [timestamp]

Scanned: [N] sources, [M] total items awaiting decision

## Urgent + quick ([N] items, ~[T] min)
R-01 [2 min] [TICKET-ID] — [short description]
R-02 [3 min] [TICKET-ID] — [short description]
...

## Urgent + deep ([N] items, ~[T] min — block time for these)
R-NN [20 min] [TICKET-ID] — [short description]
    Link: [deliverable path]
    Decision needed: [specific question]
...

## Not urgent + quick ([N] items)
R-NN ...

## Not urgent + deep (top 3)
R-NN ...

## Governance gates awaiting approval
R-NN [campaign name] — blocked per CLAUDE.md marketing governance rule

---

Response format:
  approve [N,M,...]           — approve in batch
  reject [N] [reason]         — reject with reason
  defer [N] [new-due-date]    — defer
  comment [N] [text]          — add comment, re-assign to relevant owner
  edit [N] [changes]          — substantive change, not approve/reject
  more [N]                    — show full detail for item N before deciding

Tip: chain multiple in one response.
```

### Step 5 — Parse user response

User responds with one or more actions. Parse:

- Space/comma-separated ID lists: `approve 1, 3, 5`
- Inline reasons: `reject 2 — launched too late`
- Natural phrasing: `yes to all marketing items`
- Mixed: `approve 1, 3; comment 5: yes but check with Jana first; defer 6 to next Monday`

**Ambiguity:** if any action is unclear, list the ambiguous ones back and ask before executing. Do NOT guess.

### Step 6 — Apply each decision

For each parsed action:

**Approve:**
- Set ticket status → Done (or `Approved` if workflow distinguishes)
- Update `Decision date` + `Decided by` fields
- Execute downstream action per project config:
  - Shopify change → run pre-drafted GraphQL via MCP (respecting project Shopify governance rule)
  - Blog post → schedule/publish
  - Email campaign → queue send
  - Research action → file per source research's Recommendations
- Log to session log

**Reject:**
- Set ticket status → Cancelled / Rejected
- Append rejection reason to ticket description
- Notify any dependent tickets (e.g. blocked work) that this dependency is resolved negatively
- Log

**Defer:**
- Set ticket status → Deferred
- Update `Due date` to new date
- Add deferred-reason note if provided
- Log

**Comment:**
- Append comment to ticket description with date
- Re-assign to relevant owner (inferred from comment or explicitly stated)
- Status stays in review or moves to "In progress" based on comment intent
- Log

**Edit:**
- This is substantive change, not simple approval. Surface as a mini-brief requesting what specifically to change. May create a new ticket for the edit work.

### Step 7 — Session summary

```
Review session complete — [timestamp]

Decisions applied: [N]
  — [X] approved (→ [Y] executed downstream, [Z] handed off)
  — [X] rejected
  — [X] deferred
  — [X] commented / re-assigned
  — [X] flagged for edit
  
Remaining unreviewed: [N]  (deferred to next session)

Session log: [path to session-log file]

Next suggested step: [one concrete thing based on remaining items]
```

### Step 8 — Audit trail

Append full session log to `context/sessions/[YYYY-MM-DD] - Review session.md`:
- Timestamp
- All items surfaced with their classifications
- All user decisions with parsed actions
- All downstream executions attempted + outcomes
- Errors and blockers

This file is consumed by `wrap-session` on session close.

## Edge cases

- **Nothing to review** — "All caught up. No pending decisions." End.
- **User response is ambiguous** — list ambiguous items back, do NOT guess or partially-apply.
- **Downstream execution fails** — mark the item as approved in tracker but flag execution failure; don't auto-retry without user confirmation.
- **Governance-gated item** without CEO explicit approval — require fresh approval in-session regardless of other workflow status (CLAUDE.md rule takes precedence).
- **Approval would violate a project rule** (e.g. publishing content that contains banned internal facts per public-content rule) — block, surface the conflict, do NOT auto-apply.
- **Item is already approved by someone else** — acknowledge, skip, log.
- **Session interrupted** — state file at `context/sessions/[date] - Review session.md` captures decisions applied so far; resume picks up from where parsing left off.
- **Item disappears during session** (someone else edits/closes) — skip, log as "superseded during session".

## Integration with other operator-loop skills

- **`/research`** — produces "In review" tickets that this skill consumes
- **`/questions`** — walk questions and their answers can produce decisions that flow into `/review`
- **`/tidy`** — runs after `/review` to rebalance backlog given just-made decisions
- **`wrap-session`** — captures session log

## Related

- `~/.claude/skills/OPERATOR-LOOP.md` — family overview
- Project `CLAUDE.md` — governance rules that constrain what can be approved / executed
