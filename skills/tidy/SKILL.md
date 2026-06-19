---
name: tidy
description: Trigger this when the user says "/tidy", "cleanup", "task cleanup", "rebalance", "clean up backlog", "tidy up", or wants the task list cleaned of overlaps, stale items, and rebalanced against calendar and velocity. Dedupes similar tickets, cancels stale ones, measures closing velocity, checks against calendar capacity, proposes priority rebalances and a 2-week plan. Run weekly after big research/review cycles.
---

# Tidy — task list hygiene + calendar rebalance

Keep the backlog honest. Find duplicates, prune stale items, measure velocity, check against calendar capacity, rebalance priorities, propose a next-2-weeks plan.

Run weekly. Typical session: ~5-10 min of user attention on a prepared summary, most output is automatic.

Part of the **operator loop** skill family: `/research` → `/questions` → `/review` → `/tidy`. See `~/.claude/skills/OPERATOR-LOOP.md`.

## Run modes

- **`/tidy`** (default) — full 4-stage run with user confirmations
- **`/tidy dry`** — show findings without applying any changes
- **`/tidy dedupe`** — Stage A only (dedupe + relevance)
- **`/tidy velocity`** — Stage B only (velocity report, no changes)
- **`/tidy capacity`** — Stage C only (calendar check)
- **`/tidy rebalance`** — Stage D only (priority + sequence proposal)
- **`/tidy since:[spec]`** — limit analysis to tickets/activity since date (default: last 30 days)

## Prerequisites

### Config discovery (standard pattern)

1. `~/.claude/skills/tidy/SKILL.md` — this file
2. `.claude/tidy.md` — project override
3. `~/.claude/projects/-[escaped-project-path]/memory/MEMORY.md`

### Required project config

`.claude/tidy.md` must declare:

- **Tracker source** — Notion DB ID, tracker path, or GitHub Issues
- **Velocity history window** — default 8 weeks; how far back to measure
- **Calendar source** — Google Calendar MCP if available, else local `.ics` path, else ask user for upcoming busy days
- **Deep-work day definition** — consecutive hours of uninterrupted time (e.g. "≥3h block = deep-work day")
- **Priority definitions** — P0/P1/P2/P3 meanings in this project
- **Category taxonomy** — for velocity breakdowns and batching
- **Capacity bounds** — tickets per week the user can realistically close (can be auto-learned from velocity)

## Run order

Use `TodoWrite` to track progress through stages. User confirms actions at key gates.

---

### Stage A — Dedupe + relevance

**A1 — Dedupe pass**
- Fuzzy-match all open ticket titles (use word-level cosine or Levenshtein, threshold ~80% similarity)
- For each match pair, fetch descriptions and check semantic overlap
- Propose merges to user:

```
Dedupe candidates — [N] pairs

 1. ACME-MKT-055 "SK wine directory backlinks survey"
    ACME-SEO-039 "Backlink survey for Slovak directories"
    Similarity: 87%  — likely duplicate
    Propose: merge into ACME-SEO-039 (earlier), close ACME-MKT-055

 2. ...

Respond: "merge 1, 2" / "merge 1" / "skip 2" / "keep both 3"
```

**A2 — Relevance pass**
- For each open ticket older than 30 days:
  - Scan last-30-days changelog / wiki / research outputs for references to this ticket
  - Check if a related campaign has closed or product line retired
  - Check if the ticket's premise is still valid given recent decisions
- Propose cancellations:

```
Relevance candidates — [N] tickets

 1. ACME-MKT-023 "Draft TikTok wine campaign brief" (created 60 days ago)
    Flag: TikTok Shop ruled out on Apr 19 ("alcohol restrictions, no content pipeline")
    Propose: cancel with reason "TikTok channel ruled out"

 2. ...

Respond: "cancel 1, 2" / "keep 3" / "defer 4"
```

**A3 — Thin-body pass** (per `feedback_notion_task_self_contained`)

Scan all open tickets for thin bodies — tickets where the body is empty, contains only a "see X" pointer, or has fewer than ~80 words of actual execution detail. Per the self-contained-tasks rule, every body must be executable from Notion alone.

For each thin ticket, locate the source content (linked file, campaign folder, prior research deliverable) and propose repopulation:

```
Thin-bodied tickets — [N] found

 1. ACME-WK17-TUE-1 "Post Vinalies Gold on GBP" (body empty, "Blocked By" field has a path pointer)
    Source content: output/campaigns/260407 - Vinalies Internationales 2026/.../Social Posts.md § GBP post
    Propose: auto-populate body with the post text + step-by-step posting instructions + asset paths

 2. ACME-OPS-019 "EDIH free assessment" (body has only "see EU funding research")
    Source content: output/reports/260422 - EU Funding Research/dossier.md § EDIH
    Propose: auto-populate body with the assessment URL, contact, deadlines, prep checklist

 3. ...

Respond: "populate 1, 2" / "skip 3" / "owner-only 4" (assign back to ticket owner with thin-body flag — they populate manually)
```

Also check **relation completeness**: tickets that mention a campaign / project / sibling task in the body but have empty "Depends on" / "Project link" / Campaigns dual relation (named "Campaign" or "Linked Tasks") get flagged for linking.

```
Unlinked tickets — [N] found

 1. ACME-WK17-TUE-2 "Post R&R primary on GBP"
    Mentions: "Vinalies catch-up" + "European R&R campaign" — neither linked
    Propose: link to R&R Campaign + Depends on Vinalies catch-up sibling

Respond: "link 1, 2" / "skip 3"
```

**A4 — Ticket ID backfill pass**

Every ticket needs a stable ID so it can be cited in chat, briefs, calendar entries, blog drafts, and cross-references. New tickets sometimes land without one (manual creation in Notion, recurring-task duplication, agent oversight). This pass closes the gap.

Steps:
1. Enumerate all open tickets and read their Ticket ID property.
2. For each ticket where Ticket ID is empty, look up the ticket's Category and resolve it to a project-defined prefix (see project override `category_to_prefix` map).
3. For each prefix, find the highest existing number across the whole tracker (open + closed). Assign the next sequential number(s) zero-padded to the project's standard width (default 3 digits, e.g. `ACME-MKT-079`).
4. Present the proposed assignments as a single batch:

```
Ticket ID backfill — [N] tickets without IDs

 Prefix counters (max → next):
   MKT  75 → 076
   SEO  60 → 061
   TECH 28 → 029
   OPS  44 → 045
   FIN   8 → 009

 Proposed assignments:
  1. "[SEO] Push 7 keywords from page 2 to page 1"        → ACME-SEO-061
  2. "[Marketing] Hit all 4 social slots this week"       → ACME-MKT-076
  3. "Board one-pager" (Admin → OPS prefix)               → ACME-OPS-048
  ...

Respond: "go" / "approve 1, 3" / "remap 2 to MKT-100" / "cancel"
```

5. Apply approved assignments via the tracker's update API (Notion: patch-page on the Ticket ID property). Sequential, never parallel — if two passes land at once they can collide on the same number.

Edge cases:
- **Category-to-prefix map missing for a category** — flag the ticket, ask user to extend the map in `.claude/tidy.md`. Do NOT guess a new prefix on the fly.
- **Mixed prefixes for one Notion category** (e.g. "Marketing" usually maps to MKT but a campaign-specific run wants CAMP) — honour user override per ticket via the "remap N to X" response form.
- **Numbering gaps** (e.g. SEO has 1-28 then 31-60) — never reuse old gap numbers; always continue from max. Gaps are evidence of past renames/cancels and look identical to current IDs after the fact.
- **Weekly week-coded IDs** like `ACME-WK17-TUE-1` — these are a separate scheme and are NOT generated by this pass. Skip them and leave their Ticket ID alone if it already follows the WKnn pattern.

Run this pass on every `/tidy` (it's cheap when there's nothing to do) and explicitly during `/tidy dedupe`.

---

### Stage B — Velocity measurement

Measure how fast the project is actually closing work.

**B1 — Count closed tickets by rolling windows**
- Last 2 weeks
- Last 4 weeks
- Last 8 weeks
- Year-to-date

**B2 — Category breakdown**
- For each category (MKT, SEO, TECH, OPS, FIN, etc.), count closures and compute velocity in that category
- Detect drift: categories closing slower than opened are net-growing; categories closing faster are net-shrinking

**B3 — Lead-time**
- For tickets closed in the last 4 weeks, compute days from creation → closure
- Median + p75 + p90

**B4 — Output**

```
Velocity — [YYYY-MM-DD]

Last 2 weeks: [N] closed, [M] opened, net [±N]
Last 4 weeks: [N] closed, [M] opened, net [±N]
Last 8 weeks: [N] closed, [M] opened, net [±N]

Current open backlog: [N] tickets
Rolling velocity (4w avg): [N.N] tickets/week

At current velocity, clearing the backlog takes: ~[N] weeks
 (shrinking / growing / stable by: ±[N] tickets/week)

Category heat map:
  SEO:   [N] open   velocity [N/wk]   trend [↑/↓/→]
  MKT:   [N] open   velocity [N/wk]   trend [↑/↓/→]
  TECH:  [N] open   velocity [N/wk]   trend [↑/↓/→]
  ...

Lead time:
  Median: [N] days    p75: [N] days    p90: [N] days
  Slowest category: [X] (median [N] days)

Health flag: [GREEN / YELLOW / RED]
  Criteria: green = backlog shrinking or stable ≤12 weeks to clear
            yellow = growing but <20 weeks to clear
            red = growing AND >20 weeks to clear

Suggested interpretation:
 - [one or two lines of what this means for the coming weeks]
```

---

### Stage C — Calendar check

Understand what time the user actually has to close tickets.

**C1 — Read calendar**
- If Google Calendar MCP available: query next 14 days
- Else if `.ics` path: parse it
- Else: ask user for known busy blocks (travel, all-day meetings, PTO)

**C2 — Classify each day**
- **Deep-work day**: ≥3h uninterrupted block available
- **Fragmented day**: 1-3h available in small slots
- **Admin day**: <1h available (meetings-dense)
- **Off day**: PTO / travel / unavailable

**C3 — Capacity vs demand**
- Count upcoming P0 tickets requiring deep work
- Count deep-work days available
- Flag overload if demand > supply

```
Capacity — next 2 weeks

Deep-work days:    [N]  (Tue Apr 21, Thu Apr 23, ...)
Fragmented days:   [N]
Admin days:        [N]
Off days:          [N]

P0 tickets needing deep work:    [N]
P1 tickets needing deep work:    [N]

Match check:
  Deep-work supply:  [N] days
  Deep-work demand:  [N] tickets (assuming 1 ticket/day)
  Net:  [surplus of X / deficit of Y]

Overload flag: [GREEN / YELLOW / RED]
```

---

### Stage D — Priority rebalance + sequence proposal

**D1 — Detect overload / underload**
- If open P0+P1 count > 2 weeks of velocity → overload
- If open P0+P1 count < 0.5 weeks of velocity → underload (promote P2s)

**D2 — Propose priority changes**
- For overload: suggest P1 → P2 demotions (older, less-blocked tickets first)
- For underload: suggest P2 → P1 promotions (high-value, time-sensitive first)
- Present as batch confirm:

```
Priority rebalance — [N] proposed changes

Overload relief ([X] demotions):
 1. ACME-SEO-044 P1 → P2  "Hreflang audit"   rationale: no deadline, low blocking
 2. ACME-TECH-018 P1 → P2 "Cookie banner audit"  rationale: overlap with ACME-TECH-025

Promotions ([Y]):
 3. ACME-MKT-038 P2 → P1  "Email list growth plan"  rationale: opportunity-cost finding from Apr 18 ads analysis

Respond: "go" / "approve 1, 3" / "reject 2" / "cancel"
```

**D3 — Sequence proposal for next 2 weeks**
- Map remaining P0 + P1 tickets onto available days
- Respect:
  - Dependencies (blocked-by tickets first)
  - Batching similar work (all SEO one block, all legal another)
  - Deep-work tickets on deep-work days
  - Admin tickets on admin days
- Leave slack: don't pack >80% of available time

```
Next 2 weeks proposed schedule

Mon Apr 20 [deep]:
  - ACME-MKT-071 P0 "AWC Vienna entry decision execute"   [deep]
  - ACME-SEO-003 P1 "Homepage H1 fix"                    [admin]

Tue Apr 21 [fragmented]:
  - ACME-OPS-017 P1 "Monthly team sync setup"            [admin]
  - ACME-SEO-006 P2 "Collection meta descriptions"       [deep-ish]

Wed Apr 22 [OFF — board call]:
  - (reserved)

Thu Apr 23 [deep]:
  - ACME-TECH-014 P0 "Mobile checkout investigation"     [deep]

...

Slack time: ~20% across the 2 weeks — good margin for interruptions / bugs

Respond: "go" / "shift 2 to Wed" / "add X" / "regenerate"
```

---

### Stage E — Apply decisions

For each user-approved change:
- Tracker updates (merges, cancels, priority changes)
- **Body consolidation on merge** — when merging two dedupes, take the more complete body as the base, then append any non-overlapping content from the loser; never lose information. Per `feedback_notion_task_self_contained`, the merged body must remain self-contained.
- **Relation merge on merge** — union the "Depends on" / "Project link" / Campaigns dual relation lists from both tickets so the surviving ticket inherits all sibling connections.
- **Body repopulation** — for tickets approved in A3 thin-body pass, write the proposed populated body to Notion via patch-block-children
- **Relation linking** — for tickets approved in A3 unlinked pass, patch the relation properties via patch-page
- **Ticket ID assignment** — for tickets approved in A4 ID-backfill pass, patch the Ticket ID property via patch-page. Apply sequentially per prefix to avoid number collisions.
- Calendar hints: create Notion ticket notes with suggested-date field (do NOT add to external calendar without explicit confirmation)
- Update `context/wiki/log.md` with the rebalance summary
- Update MEMORY.md Active Projects if any project status shifted

---

### Stage F — Summary

```
Tidy complete — [YYYY-MM-DD]

Dedupe:            [N] merged, [N] kept separate
Cancellations:     [N] (reasons logged)
Ticket IDs:        [N] backfilled
Priority changes:  [N] applied
Next 2w plan:      [N] tickets scheduled across [M] days, [X]% slack

Velocity trend:    [same / improving / declining]
Health flag:       [GREEN / YELLOW / RED]

Top 3 decisions this tidy:
 1. [decision]
 2. [decision]
 3. [decision]

Next suggested:
 - /research if backlog has research-ready items
 - /review if there are "In review" tickets
 - wrap-session at end of work day
```

## Edge cases

- **No calendar data available** — proceed without capacity stage, ask user for known blocks for next week only
- **Velocity history <4 weeks** — use whatever is available, flag as "low-confidence velocity estimate"
- **All tickets P0** — suggest the project has no real prioritization; propose demoting older P0s to P1 and surface to user
- **Nothing to tidy** — rare; end with "Backlog is healthy" and show velocity snapshot
- **User rejects all proposals** — that's fine, log the session and move on
- **Dedupe pair has different owners** — ask which owner to keep; do NOT auto-merge
- **Priority change conflicts with CLAUDE.md governance** — block the change; governance rules take precedence
- **Session interrupted** — state file captures progress; resume picks up at last completed stage

## Integration with other operator-loop skills

- **`/research`** — often produces new tickets that `/tidy` consolidates a week later
- **`/review`** — decisions made in review often trigger priority changes that `/tidy` locks in
- **`/questions` upload** — creates new tickets that need fitting into the schedule; `/tidy` rebalances
- **`wrap-session`** — captures tidy session log and updated backlog state

## Related

- `~/.claude/skills/OPERATOR-LOOP.md` — family overview
- `~/.claude/skills/wrap-session/` — session close-out
