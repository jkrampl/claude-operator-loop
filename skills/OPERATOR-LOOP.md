# Operator Loop — Skill family overview

A coherent workflow for solo operators (founders, CEOs, project leads) whose bottleneck is decision-making, not execution. The five skills below form a continuous loop that converts "lots of open work" into "lots of finished work ready for review" and then "decisions applied" with minimum desk time.

Companion: `wrap-session` + `resume-session` for session boundaries.

## The loop

```
┌─────────────┐       ┌─────────────┐
│  /research  │──────▶│  findings filed
│  (dispatch) │       │  + review tickets
└─────────────┘       └─────────────┘
      ▲                      │
      │                      ▼
┌─────────────┐       ┌─────────────┐
│   /tidy     │       │  /questions │
│ (rebalance) │       │  (generate  │
│             │       │   walk Qs)  │
└─────────────┘       └─────────────┘
      ▲                      │
      │                      ▼
┌─────────────┐       ┌─────────────┐
│  /review    │◀──────│ /questions  │
│ (decisions) │       │  (upload    │
│             │       │  transcript)│
└─────────────┘       └─────────────┘
      │
      ▼
┌─────────────┐
│   /draft    │  finished deliverables
│  (produce)  │  ready for /review gate
└─────────────┘
```

**Typical week:**
- Monday morning: `/research` before the first walk, `/questions` generates walk Qs
- Walk: voice-record answers
- Post-walk: `/questions upload` applies decisions, `/review` handles anything complex
- When research surfaces "we need a blog / email / pitch about X": `/draft` produces the finished copy; `/review` approves
- Friday / Sunday: `/tidy` rebalances for next week
- Every session: `wrap-session` on exit, `resume-session` on entry

## Each skill in one sentence

- **`/research`** — Discover research-ready tickets, batch them, dispatch parallel agents, file findings, auto-create follow-up tickets. Use before walking away.
- **`/questions`** — Generate a walk-friendly numbered question list from open decisions; upload the transcript later and apply decisions automatically.
- **`/review`** — Surface everything awaiting CEO decision, grouped by urgency and depth; accept fast responses and apply them across tracker, files, and integrated tools.
- **`/draft`** — Produce finished customer-facing / partner-facing deliverables (blog, email, social, pitch, one-pager, outreach) from a tracker ticket. Distinct from `/research`: research investigates, draft creates artifacts. All drafts go through `/review` before publishing.
- **`/tidy`** — Dedupe tickets, cancel stale work, measure velocity, check calendar, rebalance priorities, propose next-2-weeks plan.

## Research vs Draft (the key distinction)

| Axis | `/research` | `/draft` |
|------|-------------|----------|
| Goal | Find truth | Make artifact |
| Output | Findings + recommendations | Finished deliverable |
| Audience | Internal (CEO, team) | External (customers, partners, press) |
| Public-content rule | Soft | Hard — every draft is customer-facing |
| Writing standard | Relaxed | Strict (per CLAUDE.md + avoid-ai-writing) |
| Governance | None (internal) | Full (marketing approval gate always) |
| Review cadence | +7 days | +3 days (drafts go stale fast) |
| Chains to | `/review` for recommendations | `/review` for publish/reject |

## Shared conventions across all four

### Config discovery pattern

Every skill loads:
1. `~/.claude/skills/[skill-name]/SKILL.md` — generic workflow (this repo)
2. `.claude/[skill-name].md` — project-specific override
3. `~/.claude/projects/-[escaped-project-path]/memory/MEMORY.md` — project memory context

### Ticket ownership (the five-item closed list)

Every ticket touched by any operator-loop skill has one of these owners:
- **`Claude`** — autonomous execution OK
- **`Claude (post-approval)`** — CEO approves once, Claude executes via MCP / tools
- **`CEO`** — manual UI, human meeting, board decision
- **`External`** — dev, designer, vendor
- **`[Team member name]`** — specific person on the project team

This is enforced at ticket-creation time in `/research`, `/questions upload`, and `/tidy`. Items without a classified owner are rejected.

### Shared state file location

Every project has: `context/sessions/[YYYY-MM-DD] - operator-loop state.md`

Each skill appends its own section. `wrap-session` captures this as part of session close.

### Fast-confirm UX

Default response format for every skill:
- Plan is presented in one screen
- Single word confirms (`go`, `skip`, `cancel`)
- Comma-list cherry-picks (`go 1, 3, 5`)
- Inline comments for edits (`comment 2: change X to Y`)
- Batch actions in one message (`approve 1, 3; defer 2; reject 4 — reason`)

Never multi-step wizards. Never more than one screen of content before action.

### Auto-ticket creation

Every finding, every new work item, every open question turns into a tracker ticket automatically. No "TODO in a file" state that can drift. Tickets are the only long-term state container.

Rule reference: `feedback_ticket_everything` (per ACME memory; adopt for other projects).

### Self-contained ticket bodies + relation linking (HARD RULE)

Every ticket created or touched by any operator-loop skill must be:

1. **Self-contained** — body must be executable from Notion alone. Inline the actual copy, asset URLs, step-by-step instructions, source paths, approval state, context. No "see X for detail" without ALSO inlining the content. A team member with only Notion access must be able to execute the task without opening anything else.

2. **Connected** — every related ticket must be linked via the project's relation properties (typically "Depends on", "Project link", and the Campaigns dual relation on the Notion Tasks DB — named "Campaign" or "Linked Tasks" depending on how Notion auto-named it when the dual was created; check the actual property name on the Tasks DB before writing). Sibling tickets in the same campaign or initiative get linked. Parent project / campaign get linked.

**Enforcement across the family:**
- `/research` Step 6 (file + action) — auto-created follow-up tickets get full bodies + relations
- `/draft` Step 6 (file + governance gate) — review tickets carry the draft inline + link to source ticket
- `/questions` upload Step 6 (apply in batch) — new tickets from voice answers get full bodies + relations to question source
- `/tidy` Stage A3 (thin-body pass) — flags ticket bodies that are empty or pointer-only for repopulation
- `/tidy` Stage E (apply) — when merging dedupes, consolidate bodies AND merge relation lists

**When picking up an existing thin ticket** (no body, or body with only a pointer): populate the body FIRST with full execution detail, THEN execute the work. Mandatory, not optional.

Rule reference: `feedback_notion_task_self_contained` (per ACME memory; adopt for other projects).

### Public-content sanity

Any output meant for external eyes (website, social, press, email) goes through a public-content filter before filing. Banned patterns (internal revenue, headcount, volumes, internal file paths) are grep-checked and matches block auto-file.

Rule reference: `feedback_public_content_only` (per ACME memory; generalize per project).

### Writing rules

Every output passes the project's writing-rule check:
- No AI-isms (delve, leverage, landscape, tapestry, etc.)
- No em-dashes (or respect project's tolerance)
- Plain verbs, sentence-case headings
- Project-specific brand rules (e.g. ACME: not "family winery", no hectares)

Rule reference: `CLAUDE.md` at project root; `feedback_always_humanize` memory file.

## Cross-project portability

All four skills are generic. They discover project context from:
- Task tracker (Notion / local file / GitHub Issues)
- Output folder conventions
- Memory file
- CLAUDE.md governance rules

A new project can adopt the operator loop by:
1. Copying the 4 `.claude/[skill].md` override files as templates
2. Filling in tracker IDs, folder paths, team roster, category taxonomy
3. Running `/research dry` once to verify config

## Updating the skills

Because the logic is at `~/.claude/skills/[skill-name]/SKILL.md`, one edit updates all projects. Adding a feature (e.g. new input source for `/questions`) ripples to every project's next run.

Project overrides at `.claude/[skill-name].md` can:
- Add project-specific sources
- Override defaults (word caps, thresholds, etc.)
- NOT contradict the generic workflow (if they do, generic wins)

## Related skills

- **`superpowers:dispatching-parallel-agents`** — the underlying pattern used by `/research` for parallel agent dispatch
- **`wrap-session`** — session close with state capture
- **`resume-session`** — session startup with state load
- **`avoid-ai-writing`** — output quality filter shared with all four

## Future extensions

- **Scheduled runs** — e.g. weekly `/tidy` + Monday `/research` via cron / scheduled tasks.
- **`/digest`** — produce a condensed summary of the week's research + decisions + drafts for async stakeholders (board, advisors).
- **Cost tracking** — report token spend per sweep so operators can optimize.
- **Success metrics** — measure how often research → shipped outcome, decision → action, tidy → velocity improvement, draft → approved → published.
- **`/publish`** — automate the post-approval publish step for approved drafts (Shopify writes, email sends, social posts). Currently manual per governance.

## Change log

- **2026-04-19** — Initial build: research, questions, review, tidy, draft. ACME reference config at `.claude/[skill].md`. Draft includes type templates for blog, email, social, outreach, pitch, one-pager.
- **2026-06-14** — v2 cross-project upgrades (below): workspace guard, /monday + /work commands, decision-debt, owner-routing, close-out ritual + decision log, notion-registry standard, canonical status vocab, self-distilling playbook.

---

# Operator Loop v2 — cross-project upgrades (2026-06-14)

Applies to every project. Per-project wiring lives in each project's `.claude/` (registry + override blocks); the conventions are here.

## 1. Workspace guard (HARD — for multi-workspace operators)

The hosted Notion connector is **single-workspace**. An operator with several Notion-based projects can run a skill while the connector is pointed at the wrong company → wrong-workspace writes. Guard against it:

- Every project keeps a **`.claude/notion-registry.json`** (schema below) with its `home_page` ID + `home_title`.
- **Before any Notion write**, the skill `notion-fetch`es the `home_page` ID. Resolves to the expected title → right workspace, proceed. Errors / different page → **STOP** and tell the user to switch the connector. (Don't rely on ID-prefix matching — IDs created at different times don't share a clean prefix. Fetch-and-confirm is reliable.)
- `/monday` and `/work` run this guard as step 1.

### `.claude/notion-registry.json` standard (generalize ACME-RE's pattern)
```json
{
  "project": "<name>", "root": "<abs path>",
  "workspace": "<human label>",
  "home_page": "<uuid>", "home_title": "<exact page title to verify>",
  "databases": { "tasks": {"database_id":"…","data_source_id":"…"}, "projects": {…}, "campaigns": {…}, "wiki": {…} },
  "section_pages": { "…": "<uuid>" },
  "ticket_prefix": "<PREFIX>", "categories": ["…"],
  "guard": "notion-fetch home_page; if it doesn't resolve to home_title, STOP — wrong workspace."
}
```
Sessions read this **first** for IDs (replaces ad-hoc `notion-ids.md`).

## 2. Global commands (`~/.claude/commands/`)

- **`/monday`** — guard → resume-session → /review (oldest-first) → /research. The whole Monday loop in one word, project-agnostic.
- **`/work [N|filter]`** — execute the next N Claude-owned, unblocked `Todo` tickets in priority order, with the close-out ritual; stop at any approval/external-write gate. Default N=3.

## 3. Shorthand (document in each project; honor everywhere)
`go`/`approve` · `skip` · `defer [period]` · `reject — reason` · `comment N: …` · batch `approve 1,3; defer 2; reject 4 — reason` · `next P0` · `work SCK-X`.

## 4. Decision-debt (add to /review + the weekly brief)
Decisions rotting in the queue is the #1 solo-operator failure. The weekly brief + `/review` must flag items in **In Review / Awaiting CEO aging > 7 days**, surface them first, and report a WIP count. Cap concurrent In-Review at a project-set number; warn over it.

## 5. Owner-routing: do-now vs blocked-on-a-hire (add to /tidy + brief)
Separate tickets **Claude can do now** (Owner Claude / Claude (post-approval), unblocked) from those **blocked on a human the operator hasn't engaged yet** (Owner External when no such person exists, or CEO-manual). Don't surface the latter as "actionable" — surface them as "engage X to unblock x P0s." Prevents phantom-owner noise.

## 6. Close-out ritual + decision log (add to /review + /work)
On any ticket → `Done`/`In Review`: fill `Actual Hours` + one-line `Reflection`; append the decision/finding to **`context/decisions.md`** (`## [date] [TICKET-ID] — what + why`). Gives real velocity for /tidy and a durable decision log (valuable for diligence / a future raise).

## 7. Self-distilling playbook (generalize ACME's creative-playbook loop)
For any repeated work-type (creative gen, outreach, drafting), keep a `*-playbook.md` with a `last_distilled` cursor that `wrap-session` advances: distill "what worked / failure modes learned" from reviewed outcomes into durable lessons. The loop *learns*.

## 8. Canonical status vocabulary (fix the cross-project drift)
Standardize on: **`Backlog · Todo · In Progress · Blocked · In Review · Done · Archived`**. "Needs CEO decision" = Status `In Review` + Owner `CEO` (don't invent parallel statuses). Projects on legacy sets (ACME's `Not started`/`Awaiting CEO`/`Blocked on decision`) keep working, but new projects + shared logic use the canonical set; map legacy → canonical in the project's `.claude/tidy.md`.

