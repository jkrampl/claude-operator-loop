---
name: deploy-project
description: Trigger this when the user wants to set up a new project (greenfield) or audit + onboard an existing project (brownfield) to the ACME-style operator-loop system. Use when user says "/deploy-project", "new project setup", "set up a new company project", "onboard project X", "audit this project", "apply the ACME framework to X", or names a project they want scaffolded with: folder structure, CLAUDE.md, MEMORY.md, Karpathy-style wiki, Notion Tasks DB, operator-loop skills (/research, /review, /questions, /tidy, /draft), MCP integrations, governance rules, first research + review + backlog generation cycle. Works across any industry: tech, medical, interior design, food & beverage, finance, family office, manufacturing, etc.
---

# Deploy project — end-to-end onboarding runbook

Standardize a new or existing project onto the same operating system as Acme Co. Works across any industry. Produces a fully-configured project that the operator-loop skills work against immediately.

This skill is a **runbook**: a sequence of phases with CEO input checkpoints and Claude autonomous work between them. Not a single-shot script — a multi-hour (often multi-day) collaborative onboarding.

## What "deployed" means (the end-state)

After this runbook completes, the project has:

1. **Notion workspace as single source of truth** for tasks, wiki, campaigns, projects. Built from the canonical ACME template — hybrid structure (databases + section landing pages) so both Claude and humans can navigate efficiently. See `references/notion-workspace-architecture.md`.
2. **Local folder structure** at standard paths (`context/`, `output/`, `data/`, `.claude/`) for code, raw data, generated artifacts (HTML reports, Excel, images), scripts, and Claude configs. Split rules in `references/notion-workspace-architecture.md`.
3. **`CLAUDE.md`** — governance rules, brand rules, writing rules, per-project overrides (local).
4. **`MEMORY.md`** — index of company, active projects, team, completed work, topic memory files (local; lean, not duplicating Notion).
5. **Notion Tasks DB** — properties configured, Kanban + Today + This Week + Blocked views, ticket-ID convention, Claude MCP integration connected and tested.
6. **Notion Wiki DB** — properties (Section / Status / Summary / Owner / Original path / Last meaningful update / External refs), plus 5–6 section landing pages with `@mention-page` navigation to DB rows. Initial company / market / operations pages populated.
7. **Notion Projects DB** — one row per active initiative; Tasks relate back to Projects.
8. **Notion Campaigns DB** — campaign-level planning with Tasks relating back.
9. **ACME — Home dashboard page** — callout with brand summary + This Week priorities + 4 embedded databases in column layout + section navigation cards + "How to work here" toggles + setup-still-to-do checklist.
10. **Transition plan page** (for brownfield projects) — 8-week phased rollout from local → Notion with weekly GO/HOLD/ROLLBACK gates + rollback triggers + known-issues log. Template at `templates/notion-transition-plan.md.template`.
11. **Operator-loop skill family deployed as a unit** — `/research` + `/review` + `/questions` + `/tidy` + `/draft` + `/wrap-session` + `/resume-session`. These are NOT optional add-ons — they are the operating system the project runs on and only work as a coherent set. Configured via project-level `.claude/research.md`, `.claude/review.md`, `.claude/questions.md`, `.claude/tidy.md`, `.claude/draft.md`, `.claude/wrap-session.md`. All read/write Notion via the hosted MCP directly. See `~/.claude/skills/OPERATOR-LOOP.md` for the family overview. A "deployed" project that lacks any of these is incomplete.
12. **MCPs connected** — hosted Notion MCP (always) + Google Calendar (always recommended) + industry-specific set (analytics / e-commerce / CRM / etc.). OneDrive/SharePoint paste-preview configured if CEO uses Microsoft 365.
13. **Governance gates defined** — what needs CEO approval, what's autonomous.
14. **First research sweep run** — initial backlog + competitor landscape + market context; outputs written directly to Notion Wiki DB.
15. **First review + tidy cycle** — backlog prioritized, first-2-weeks plan.
16. **Recurring cadence** — weekly/monthly/quarterly/annual tasks scheduled.

**Anti-pattern to avoid:** building a separate "sync integration" between local markdown files and Notion. The hosted Notion MCP gives Claude direct read/write access — no second integration token, no hooks, no Python mirror script. ACME learned this the hard way on Apr 19, 2026; see `references/notion-workspace-architecture.md` § "Why no local mirror sync".

**Hard rule across every Notion task created by this deploy (and after):** every task body must be **self-contained AND connected**. A team member with ONLY Notion access — no wiki, no output folders, no campaign files, no chat history of any conversation — must be able to execute the task end-to-end from the body. No "see X for detail" without ALSO inlining the content. AND every related task must be linked via Notion's relation properties ("Depends on", "Project link", and the Campaigns dual relation — recommended name "Campaign" on Tasks DB / "Tasks" on Campaigns DB). Anything connected gets connected. This rule is enforced in Phase 7 (backlog generation), in every operator-loop skill that creates tickets, and in the per-project `feedback_notion_task_self_contained.md` memory. See `references/notion-workspace-architecture.md` § "Task body conventions" for the full spec.

## Run modes

- **`/deploy-project`** (default) — interactive, full 10-phase onboarding
- **`/deploy-project greenfield`** — new project from scratch
- **`/deploy-project brownfield`** — existing project with state to discover + integrate
- **`/deploy-project hybrid`** — mix of existing + new
- **`/deploy-project audit`** — discovery only, no writes (scouting a project before deciding to fully onboard)
- **`/deploy-project resume`** — pick up from where a previous onboarding left off
- **`/deploy-project phase:[N]`** — run just one phase (useful for iterative re-work)
- **`/deploy-project portfolio-setup`** — set up ONLY the CEO portfolio layer (no specific company). Run this FIRST if the CEO has 2+ companies on the way. Subsequent project deploys register into the existing portfolio.
- **`/deploy-project register-into-portfolio`** — register an already-deployed project into an existing portfolio (catches projects deployed before the portfolio existed).

## Reference files

All loaded on demand from `~/.claude/skills/deploy-project/references/`:

- `discovery-questionnaire.md` — full business Q list used in Phase 1
- `industry-modules.md` — per-industry deltas (tech / medical / interior / food&bev / finance / family office / manufacturing)
- `notion-workspace-architecture.md` — **canonical 4-database Notion pattern** from ACME: Projects + Tasks + Campaigns + Wiki + Home dashboard + section landing pages. What lives in Notion vs on local disk. Why no local mirror sync.
- `notion-database-schema.md` — full SQL DDL for all 4 databases (was Tasks-only before)
- `file-structure-template.md` — folder conventions, naming rules (what stays local)
- `human-vs-agent-matrix.md` — ownership framework (the 5-item closed list + decision tree)
- `mcp-shopping-list.md` — recommended MCPs per industry
- `portfolio-architecture.md` — privacy-first portfolio model: workspace isolation, integration scoping, data flow, privacy audit procedure

Templates at `~/.claude/skills/deploy-project/templates/`:

- `CLAUDE.md.template` — starter governance + writing rules file
- `MEMORY.md.template` — starter memory index
- `wiki-index.md.template` — starter wiki index (local seed; migrated to Notion Wiki DB in Phase 4)
- `wiki-log.md.template` — starter changelog (stays local — session changelog is append-only and edited every session)
- `portfolio-projects-db.md.template` — Notion spec for the CEO's Portfolio workspace (Projects DB + Weekly Digests DB + Escalations DB)
- `notion-home-dashboard.md.template` — full markdown for the "Project — Home" page (callouts, column layouts, embedded databases, navigation cards)
- `notion-transition-plan.md.template` — 8-week phased rollout doc for brownfield projects moving local → Notion

## Run order — 10 phases

Use TodoWrite to track progress through phases. Each phase has a clear checkpoint gate before advancing. The CEO sees the plan before any file writes.

---

## Phase 0 — Pre-deployment prep (CEO, offline)

Before invoking this skill, the CEO should have:

- **Project name** — short handle (e.g. "ACME", "Clinic-X", "AlphaStudio")
- **Industry classification** — tech / medical / interior / food&bev / finance / family office / manufacturing / other
- **Deployment mode** — greenfield / brownfield / hybrid / audit
- **Root folder path** — where the project will live on disk (local or cloud-synced)
- **Legal entity info** — company name, registration ID, country, legal form
- **Notion workspace** — workspace name + confirmed Claude integration permissions
- **Claude Code installed** + project folder set as workspace
- **30-60 min** blocked for Phase 1 intake interview

If any of these are missing, the CEO should pause and collect them before invoking `/deploy-project`.

---

## Phase 1 — Intake interview (CEO + Claude, interactive, 30-60 min)

**Goal:** capture enough context to scaffold everything else.

**Load** `references/discovery-questionnaire.md` and walk through it with the CEO.

The questionnaire covers:
- Company basics (legal form, industry, year founded, size, jurisdiction)
- Product or service (what you sell, unit economics, price architecture)
- Customers (segments, ICP, concentration risks)
- Market + competition (tracked peers, positioning, TAM)
- Operations + team (roster, roles, external resources, suppliers)
- Finance (revenue, margins, runway, targets)
- Marketing + sales (channels, funnels, CAC/LTV, content cadence)
- Tech + tools + data (website, CRM, ERP, analytics, inventory, CMS)
- Legal + governance (approvals needed, compliance regime, board structure)
- Goals + KPIs (12-month outcomes, top 3 priorities)

**Interview UX:**
- Ask one section at a time, not all at once
- Defer deep dives ("we'll cover that in Phase 6") to keep momentum
- Flag info user doesn't know yet — creates a research ticket for Phase 6
- Tag each answer with public/internal visibility

**Output:** `context/intake/[YYYY-MM-DD] - intake interview.md` — structured capture of all answers, gaps flagged.

**Checkpoint gate:** CEO reviews the intake document + confirms it's accurate before Phase 2.

---

## Phase 2 — Existing-state audit (Claude autonomous, 30-60 min)

**Applies to:** brownfield + hybrid modes. Skip for greenfield.

**Goal:** discover everything that already exists for this project.

**Tasks:**
1. Walk the project's file tree (if brownfield root exists) — inventory folders, note unusual patterns
2. Check Notion workspace for existing project databases / pages
3. Check for existing Google Workspace folders if CEO shared access
4. Check for existing tool accounts (GA4, GSC, Shopify, CRM, social, etc.) — query via MCPs if installed
5. Scan registry data (public records) for the legal entity — financials, directors, share capital
6. Scan public web for the company — website audit, social presence, press mentions, reviews, Wikipedia
7. Check competitor landscape — name top 5-8 tracked peers + pull public data

**Output:** `context/audit/[YYYY-MM-DD] - existing-state audit.md` — what exists, where it lives, quality assessment, integration recommendations.

**Checkpoint gate:** CEO approves the audit + flags anything the audit missed.

---

## Phase 3 — Project scaffolding (Claude autonomous, 10-20 min)

**Goal:** create the standard folder structure + starter files.

**Load** `references/file-structure-template.md` for the canonical layout.

**Tasks:**
1. Create folder tree: `context/`, `context/wiki/`, `context/tracker/`, `context/intake/`, `context/sessions/`, `output/`, `output/reports/`, `output/drafts/`, `output/campaigns/`, `data/`, `.claude/`
2. Create `CLAUDE.md` from template, filled with intake answers (company context, governance, writing rules per industry)
3. Create `MEMORY.md` from template at `~/.claude/projects/-[escaped-project-path]/memory/MEMORY.md`
4. Create `context/wiki/index.md` + `context/wiki/log.md` from templates
5. Create `.claude/` config files for each operator-loop skill (research / review / questions / tidy / draft / wrap-session)
6. Create the `.claude/settings.local.json` with industry-appropriate permission allowlist
7. Create `context/wiki/company/` starter pages: `about.md`, `registration.md`, `team.md`, `goals.md`

**Output:** populated project root + memory folder + wiki skeleton.

**Checkpoint gate:** CEO reviews the generated CLAUDE.md + MEMORY.md + wiki skeleton before Phase 4.

---

## Phase 4 — Notion workspace setup (CEO + Claude collaborative, 60-90 min)

**Goal:** build the full canonical Notion workspace — 4 databases + Home dashboard + section landing pages — that operator-loop skills will read/write. This is the project's new source of truth, not a supplement to local files.

**Load** both `references/notion-workspace-architecture.md` (the canonical pattern + why) and `references/notion-database-schema.md` (SQL DDL for all 4 DBs).

### Pre-flight: workspace decision

Ask the CEO upfront:
- Is this project going in an **existing workspace** (with other projects) or a **new workspace dedicated to this project**?
- If the CEO has 2+ projects: strongly recommend **new workspace per project** for privacy isolation. Portfolio view comes from Phase 11.
- If this is the CEO's first project on the system: single workspace is fine.

### Tasks (in order)

1. **CEO creates or chooses workspace.** If new: workspace name = project name (e.g. "ACME", "Clinic-X"). CEO confirms name + selects Free or Plus tier. (Plus tier needed eventually for permissions + team members, but Free is fine for Phase 1.)

2. **Connect hosted Notion MCP to the workspace.** CEO authorizes Claude's hosted Notion integration (comes with Claude Code — no separate token to manage). Claude runs `notion-search` to confirm it can see the workspace.

3. **Create `[Project] — Home` page** at workspace root. Use the `templates/notion-home-dashboard.md.template` content. Structure:
   - Icon: project's emoji (🍷 for wine, 🏥 for medical, etc.)
   - Hero callout: brand one-liner (company name · location · URL · key positioning stat)
   - **🔥 This Week** red-bg callout: current priorities (updated manually each Monday)
   - **🗄️ Databases** section: 4 embedded databases in 2-column layouts (Projects + Tasks on row 1, Campaigns + Wiki on row 2)
   - **🧭 Navigate** section: 6 callout-cards in a 3-column grid linking to the section landing pages (Company, Customers, Market, Channels, Operations, Wiki)
   - **🔁 How to Work Here** section: collapsible toggles for Daily routine, Weekly Friday review, Running campaigns
   - **⚙️ Setup Still To Do** section: checklist for manual UI steps the API can't do (views, permissions, default landing page)

4. **Create the 4 core databases** (use `mcp__notion-create-database` with SQL DDL from `references/notion-database-schema.md`):

   **A. Projects** — one row per active initiative (e.g. "Conversion + DTC optimization", "International sales launch")
   - Schema: Name · Status · Stage · Start Date · Due Date · Owner · Summary · Linked Tasks (relation back to Tasks DB)

   **B. Tasks** — the ticket tracker (canonical schema below)
   - Schema: Name · Ticket ID · Category (SEO/Marketing/Tech/Ops/Finance/Sales/Content/Admin — industry-aware) · Priority (P0/P1/P2/P3) · Status (Backlog/Todo/In Progress/Blocked/In Review/Done/Archived) · Source · Owner · Start Date · Due Date · Done date · Estimated Hours · Actual Hours · Dependencies · Blocked By · Project link (relation to Projects DB, dual) · Campaign (relation to Campaigns DB, dual — partner property "Tasks" on Campaigns side) · Reference (URL) · Reflection. CRITICAL: create both Project and Campaign relations as DUAL from day one so both sides auto-populate. ACME learned this the hard way April 2026 (single-property relation, then later had to add inverse manually + migrate data).

   **C. Campaigns** — campaign-level planning
   - Schema: Name · Status (Planning/Approved/Running/Complete/Cancelled) · Start Date · End Date · Budget · Channels (multi-select) · Owner · Summary · Linked Tasks (relation)

   **D. Wiki** — the canonical knowledge store (replaces local `context/wiki/`)
   - Schema: Name · Section (select: Company/Customers/Market/Channels/Operations/Root — industry-aware) · Status (Live/Draft/Needs refresh/Archived) · Summary · Owner · Original path (text — records the source file if migrated) · Last meaningful update (date) · External refs (URL)

5. **Create the 6 section landing pages** as subpages of Home:
   - 🏢 Company · 👥 Customers · 📊 Market · 📡 Channels · ⚙️ Operations · 📚 Wiki (meta-index)
   - Each landing page has: icon, intro callout, "Pages in this section" list using `<mention-page>` links to the relevant Wiki DB rows, plus "Other sections" cross-links at the bottom.
   - This is the "book-style" nav layer on top of the database — both are valid entry points.

6. **Create the Transition plan page** (brownfield only). Use `templates/notion-transition-plan.md.template`. Covers 8-week phased rollout from local → Notion with weekly GO/HOLD/ROLLBACK gates, rollback triggers, known-issues log, and what stays local forever.

7. **CEO runs UI-only setup** (clickable, 10-15 min):
   - Tasks DB views: Today (filter Due = today), This Week (filter Due in next 7 days), Kanban by Status, By Category, By Priority, Calendar on Due Date, My Tasks, Blocked, Done this week
   - Wiki DB views: By Section (grouped), Recently Updated, Drafts-only, Search
   - Projects DB views: Active (exclude Complete), Timeline (Gantt-style if Plus)
   - Set `[Project] — Home` as default workspace landing page

8. **Claude validates end-to-end:**
   - Creates one sample ticket in Tasks via `mcp__notion-create-pages`, verifies it shows in Kanban
   - Creates one sample Wiki entry, verifies it shows under the right Section landing page via `@mention-page`
   - Updates the sample ticket status to Done, verifies it disappears from Backlog view and shows in Done view
   - Deletes the samples

9. **CEO records the IDs** in `.claude/research.md`, `.claude/review.md`, etc. (operator-loop configs):
   - Home page ID
   - Projects / Tasks / Campaigns / Wiki database IDs + data source IDs
   - Section landing page IDs

**Output:** live Notion workspace with 4 databases, Home dashboard, 6 section landing pages, (optionally) Transition plan page. All reachable from Home in ≤2 clicks.

**Checkpoint gate:** CEO confirms:
- All 4 databases visible on Home
- Kanban + Today views work
- Each section landing page loads and shows its Wiki entries
- Claude's test ticket + test wiki row were created + cleaned up successfully
- Default landing page is set (so opening Notion drops you at Home)

### Hard "don't" from ACME's Apr 2026 deployment

**Do NOT** build a separate Notion integration with a custom token + Python sync script + SessionStart/Stop hooks to mirror Notion → local files. The hosted Notion MCP that ships with Claude Code already gives direct read/write access from every session. Claude reads Notion live when it needs state. Building a mirror is over-engineering that was tried, rolled back, and documented as an anti-pattern. See `references/notion-workspace-architecture.md` § "Why no local mirror sync".

---

## Phase 5 — MCP + integrations setup (CEO + Claude, 30-90 min)

**Goal:** connect the integrations the project needs.

**Load** `references/mcp-shopping-list.md` — recommends MCPs per industry.

**Typical core MCPs across industries:**
- Notion (hosted, ships with Claude Code) — always. Already connected via Phase 4.
- Google Calendar (scheduled tasks alignment) — always recommended
- **Cloud file storage integration** — branch on the file-storage backend chosen in Phase 1 (Section H question 2). Notion has similar paste-preview patterns for Google Drive, OneDrive/SharePoint, and Dropbox. See `references/notion-workspace-architecture.md` § "Cloud file storage integration" for the full comparison + tier breakdown.
  - If **Google Drive / Google Workspace** → Phase 1 "Tier 1" is the `/gdrive` or paste-preview flow. Tier 2 (AI connector) exists in Notion Business+ for semantic search. Recommended for tech, creative agencies, startups, projects already on Google Calendar/Gmail.
  - If **OneDrive / SharePoint / Microsoft 365** → paste-preview + `/onedrive` slash command. Tier 2 AI Connector is specifically named "SharePoint & OneDrive AI Connector (beta)" in Notion. Recommended for SMEs on Outlook + Office, family offices, regulated industries.
  - If **Dropbox** → paste-preview works via shared link. No native Tier 2 AI connector. Tier 3 automation (Zapier/Make) is the usual upgrade path.
  - If **Local only** → skip this step; all references live as local file paths. Add a cloud later when the team grows.
  - If **Hybrid (rare)** → configure both paste-preview flows, document in `CLAUDE.md` which content maps to which cloud. Added complexity; usually not worth it.
  - **Recommendation pattern (all backends):** start with Tier 1 paste-preview on Phase 4 setup day. Upgrade to Tier 2 only when team onboarding makes semantic search across cloud files valuable. Skip Tier 3 unless a specific recurring flow demands it.
  - Demo step (done same day as Phase 4): embed one real file from the chosen cloud into one real Notion page (e.g., latest P&L Excel embedded on the Financial Overview wiki page). Validates the flow and gives a reusable template.
- Industry-specific:
  - E-commerce: Shopify, Google Analytics 4, Search Console, Meta, Clarity, Shopify Inbox
  - Medical: patient-record system, scheduling (if compliant), billing
  - Interior design: project-mgmt tool, asset library, client portal, accounting
  - Food & bev: inventory, POS, reservation, supplier portal
  - Finance: portfolio management, CRM, compliance reporting
  - Family office: accounting, document vault, portfolio, estate tools
  - Manufacturing: ERP, QMS, supplier, shipment tracking
  - Tech / SaaS: GitHub, analytics, support desk, status page

**Tasks:**
1. For each recommended MCP: CEO authenticates, confirms connection, Claude runs a smoke-test query
2. For each integration not yet ready: create a ticket (ACME-TECH-XXX equivalent) with owner=CEO or External
3. Document connected integrations in `context/wiki/operations/integrations.md`

**Output:** working MCP stack + documented integrations page.

**Checkpoint gate:** CEO confirms all "must-have" MCPs for the industry are connected.

---

## Phase 6 — Initial content + research seeding (Claude autonomous, 30-60 min)

**Goal:** populate the wiki with current-state knowledge before operator loop starts.

**Tasks:**
1. Dispatch parallel research agents on:
   - Market landscape for this industry + this geography
   - Competitor tracking (5-10 peers identified in Phase 1)
   - Public record financials (if available) for the company + peers
   - Brand mentions + press + review aggregation
   - Regulatory context for the industry (compliance regime, licensing)
2. Populate wiki pages:
   - `context/wiki/market/competitors.md`
   - `context/wiki/market/industry-overview.md`
   - `context/wiki/company/finance.md` (if public data)
   - `context/wiki/company/regulatory.md`
   - `context/wiki/customers/segments.md`
3. For any "unknown" from Phase 1 intake, draft either a research ticket or a CEO clarification ticket

**Output:** wiki is no longer a skeleton; it has real content.

**Checkpoint gate:** CEO reviews wiki pages + flags inaccuracies.

---

## Phase 7 — Backlog generation + prioritization (collaborative, 30-60 min)

**Goal:** turn all identified gaps and goals into a prioritized ticket backlog in Notion.

**Load** `references/human-vs-agent-matrix.md` for the ownership framework.

**Tasks:**
1. Based on intake + audit + initial research, enumerate work items
2. For each item:
   - Write a concrete, **self-contained** ticket: Name + a body containing every piece of execution detail (steps, copy, links, asset URLs, source paths, approval state, conventions to follow) so a team member with only Notion access can execute it end-to-end. No "see X for detail" without ALSO inlining the content. See `references/notion-workspace-architecture.md` § "Task body conventions".
   - Plus standard properties: Priority, Category, Owner, Estimated Hours, Source, Due Date.
   - Plus **relations** to any related tickets via "Depends on" / "Project link" / "Campaign" (the Campaigns dual relation). Sibling tickets in the same campaign or initiative get connected. Parent project + campaign get linked.
   - Assign owner per the 5-item closed list: Claude / Claude (post-approval) / CEO / External / [team member name]
   - Apply ownership decision tree (from `human-vs-agent-matrix.md`)
3. Auto-create all tickets in Notion (per `feedback_ticket_everything` rule — never "pending", always create)
4. Propose a first-2-weeks sequence respecting CEO calendar + velocity assumptions
5. Flag quick wins (can be completed this week) vs strategic work (>2 weeks)

**Output:** Notion Tasks DB populated with complete backlog (typically 30-80 tickets for a full-deploy project).

**Checkpoint gate:** CEO does a `/review` session on the first-2-weeks plan, approves or re-sequences.

---

## Phase 8 — Governance codification (CEO-led, 15-30 min)

**Goal:** lock in what requires approval vs what's autonomous.

**Tasks:**
1. Review the Ownership closed list: Claude / Claude (post-approval) / CEO / External / [team member name]
2. CEO confirms governance gates for this project:
   - What content requires board approval before publishing?
   - What tool writes require per-use approval (e.g. Shopify writes, social posts, ad spend)?
   - What is explicitly pre-approved (e.g. SEO metadata, alt text, schema markup)?
   - Who signs off on legal/compliance changes?
   - Industry-specific (medical: patient-data handling; finance: investment decisions; manufacturing: supplier contracts)
3. Update `CLAUDE.md` with the governance section, filling in industry-appropriate rules
4. Draft industry-specific writing rules (e.g. medical = no unapproved health claims; food&bev = allergen compliance; finance = no investment advice)
5. Add any project-specific public-content rule (`~/.claude/projects/-[escaped]/memory/feedback_public_content_only.md`)
6. Add any brand/naming/tone rules (`feedback_always_humanize.md`, `feedback_no_em_dashes.md`, etc. — copy from ACME if appropriate)

**Output:** finalized `CLAUDE.md` + project-specific feedback memory files.

**Checkpoint gate:** CEO signs off on governance in writing in the session (not verbal).

---

## Phase 9 — First operational cycle (live test, 60-90 min)

**Goal:** validate that the operator-loop works end-to-end.

**Tasks:**
1. Run `/research dry` — verify skill sees the Notion DB, identifies research-ready tickets, proposes a plan. If yes → run `/research` for real.
2. While research is running, run `/questions` — verify it scans correctly for open decisions.
3. After research completes, run `/review` — CEO makes first batch of decisions (approve / reject / defer / comment).
4. Run `/tidy dry` — verify velocity + calendar + rebalance logic works. Propose changes, CEO approves selectively.
5. Before ending the session, run `wrap-session` — verify it captures everything to MEMORY + wiki/log + session log.
6. Next day, run `resume-session` — verify state loads cleanly.

**Output:** validated operator-loop running against the new project.

**Checkpoint gate:** all five skills run without error + produce useful output.

---

## Phase 10 — Recurring cadence setup (Claude + scheduled-tasks MCP, 15-30 min)

**Goal:** schedule all recurring work so nothing relies on human memory.

**Tasks:**
1. Weekly operator loop: reminder every Monday morning to run `/research` + `/questions`
2. Weekly digest (if industry warrants): market news, competitor changes, analytics summary
3. Monthly team sync: first Monday of each month (see `feedback_quarterly_register.md`)
4. Quarterly assessments: Jan 1 / Apr 1 / Jul 1 / Oct 1 (AI readiness, PageSpeed, security headers, schema, email deliverability, LLM brand monitoring, review platforms, backlinks — industry-specific subset)
5. Annual: registry data refresh, tax filing deadlines, insurance renewals, board review
6. Industry-specific recurring items (see `industry-modules.md` for each)
7. Create scheduled tasks via `mcp__scheduled-tasks__create_scheduled_task` for automations that don't need human input (market digests, crawls, reports)

**Output:** all recurring work scheduled + documented in `context/wiki/operations/recurring-register.md`.

**Checkpoint gate:** CEO reviews the recurring register, adjusts anything that feels wrong cadence.

---

## Phase 11 — Portfolio integration (required for CEOs with 2+ projects)

**Goal:** give the CEO a consolidated view across all their companies, while keeping each company's team strictly isolated from every other company's data.

**Load** `references/portfolio-architecture.md` for the full privacy model + integration scoping rules. Load `templates/portfolio-projects-db.md.template` for the Notion spec.

### Privacy-first default (recommended)

**Architecture:** separate workspaces per company + CEO-only portfolio workspace.

- Each company has its own Notion workspace. Company team members are invited only to their workspace. They never see the portfolio or any sibling company.
- The CEO has a SEPARATE portfolio workspace (e.g. "[Name] — Portfolio"). Only the CEO (and optionally trusted COO / family office staff) has access.
- Claude integrations are scoped per workspace. Integration connected to Workspace A cannot read Workspace B unless explicitly re-authenticated there.
- Data flows: each company workspace → Claude (authenticated there) → aggregated summary → CEO portfolio workspace.
- Internal figures (revenue, FTE, volumes) can live in the portfolio because only the CEO sees it. Per-company public-content rules still apply at each company's level — nothing about ACME revenue leaks into a ACME-facing output.

### Three implementation options

| Option | Best for | Setup effort | Mobile access |
|---|---|---|---|
| **A. Notion portfolio workspace** (recommended) | CEOs with 2+ companies + team members in each | 1-2 hrs | Full |
| **B. File-based portfolio** (local `~/CEO-portfolio/`) | Paranoid-private CEO, no team access needed, Mac-only workflow | 30 min | None (files only) |
| **C. Hybrid** (Projects DB in Notion + detailed files local) | Wants quick mobile scan but keeps deep detail off Notion | 1 hr | Partial |

### Tasks (Option A — Notion portfolio)

1. **Create portfolio workspace** (once per CEO, first time only):
   - In Notion, create a new workspace named "[CEO handle] — Portfolio" (e.g. "Jakub — Portfolio")
   - Set it to CEO-only access (no default team members invited)
   - Decide trusted delegates if any (typically 0-2 people max: COO, chief of staff, family office principal)

2. **Create the three portfolio databases** per `templates/portfolio-projects-db.md.template`:
   - **Projects** — one row per company with identity, stage, key metrics, links
   - **Weekly Digests** — one entry per project per week, Claude auto-populates via recurring task
   - **Escalations** — cross-project issues needing CEO attention, triaged weekly

3. **Connect Claude integration**:
   - Authenticate Notion integration on the portfolio workspace
   - Also authenticate on EACH company's workspace (separate auth per workspace)
   - Claude can then read from each company + write summaries to the portfolio in one session

4. **Register this project** into the Projects DB:
   - Add one row with: Project handle, Industry, Stage, Notion workspace URL, MEMORY.md path, Current P0 count, Team size, Latest material event, Next review date
   - Add internal-only fields (revenue band, cash runway, FTE) since only CEO sees the portfolio

5. **Set up recurring weekly digest**:
   - Scheduled task: every Monday morning, Claude queries each registered project's Notion Tasks DB + MEMORY.md + latest research deliverables
   - Writes one-line status per project into Weekly Digests DB
   - Emails / notifies the CEO with the digest

6. **Standardize escalation rules**:
   - What triggers a cross-project escalation? (P0 blocker for >72h / material financial event / regulatory concern / team issue)
   - How does a company team flag "need CEO attention"? (typically: ticket owner set to CEO + comment)
   - Escalations auto-promote to the Escalations DB

7. **Cross-project pattern extraction**:
   - As you deploy more projects, note patterns that repeat (campaign structures, KPI definitions, vendor relationships)
   - Promote repeating patterns to shared skills in `~/.claude/skills/` so all projects benefit
   - Examples: if 3 of your companies need Packeta integration, that's a candidate for a shared `packeta-shopify-setup` skill

### Tasks (Option B — file-based portfolio)

1. Create `~/[CEO-handle]-portfolio/` folder on your Mac
2. Subfolders: `projects/` (one MD per company, mirrors the Projects DB row), `digests/` (weekly rollups), `escalations/` (ad-hoc)
3. Register this project: `projects/[project-handle].md` with YAML frontmatter + status
4. Scheduled task generates weekly digest as HTML file opened on Monday morning

### Privacy audit checkpoint (do NOT skip)

Before ending Phase 11, verify:

- [ ] Team member in Workspace A cannot access Workspace B (test: ask a team member to try — they should see nothing)
- [ ] Team member in Workspace A cannot see the portfolio workspace
- [ ] Claude integration on Workspace A has scopes limited to Workspace A's databases
- [ ] Portfolio workspace member list = only CEO + any approved delegates
- [ ] No cross-workspace page links that could leak metadata (names, existence of other projects)
- [ ] No team-accessible Notion pages mention the portfolio workspace name
- [ ] Internal-only fields (revenue, FTE, etc.) in portfolio DB are NOT replicated to any company workspace

If any check fails, fix before proceeding. Privacy violations at this layer are hard to retroactively contain.

### Failure modes to avoid

- **Adding a team member to the portfolio "just so they can see the dashboard"** — instead, create a project-specific view in their company's workspace
- **Cross-workspace linking** — Notion will show the link target's name even if the recipient doesn't have access. Don't paste portfolio URLs into company workspaces
- **Over-broad integration scope** — when you authenticate Notion on a company workspace, explicitly select only the Tasks DB + wiki pages; don't grant workspace-wide access
- **Mixing personal + business in one portfolio** — if you have a family office + investment side, consider a SECOND portfolio workspace for investments separate from operating companies
- **Losing track of which integration is authenticated where** — document in `~/CEO-portfolio/[handle]-portfolio/operations/integration-map.md`

### Multi-device / backup considerations

- Notion is cloud-hosted, accessible from any device with your login
- File-based portfolios need manual sync if you use multiple Macs (Dropbox, iCloud, git, or manual rsync)
- Scheduled tasks run on the machine they were created on — if you use multiple Macs, pick one "portfolio home" Mac

**Output:** CEO has consolidated portfolio view, teams stay isolated, privacy verified.

**Checkpoint gate:** CEO signs off on the privacy audit + confirms the first weekly digest runs correctly the following Monday.

---

## Deployment verification checklist

When the runbook is complete, verify all of these:

- [ ] Folder structure at standard paths — `context/`, `output/`, `data/`, `.claude/`
- [ ] `CLAUDE.md` present + governance filled + writing rules filled
- [ ] `MEMORY.md` at correct path in `~/.claude/projects/-[escaped]/memory/`
- [ ] Wiki has index + log + 6+ initial pages
- [ ] Notion Tasks DB exists + Kanban view + This Week view + Claude integration connected
- [ ] Operator-loop skill config files at `.claude/research.md` etc. — all six
- [ ] Minimum MCP set for industry connected + smoke-tested
- [ ] At least 30+ tickets in Notion Tasks DB across appropriate categories
- [ ] First-2-weeks plan approved by CEO
- [ ] Recurring tasks scheduled (weekly + monthly + quarterly minimum)
- [ ] One complete operator-loop cycle executed (research → review → tidy)
- [ ] `wrap-session` + `resume-session` both tested
- [ ] Governance rules signed off in writing
- [ ] Public-content rule filled with project-specific banned patterns
- [ ] Writing rules filled with project-specific banned words

## Time budget estimate

- Greenfield full deploy: **4-8 hours** of CEO time spread over 2-3 days + 4-8 hours of Claude autonomous work
- Brownfield full deploy: **6-12 hours** CEO + 6-12 hours Claude (audit is expensive)
- Audit-only mode: **1-3 hours** CEO + 2-4 hours Claude
- Maintenance after deploy: ~4-6 hours/week of CEO time on the operator loop

## Edge cases

- **CEO drops off mid-deployment** — state file at `context/sessions/[date] - deploy-project state.md` captures phase progress; `/deploy-project resume` picks up cleanly
- **Industry not in list** — apply closest analog + note "custom industry" in CLAUDE.md; extract patterns as you go
- **No existing Notion workspace** — CEO creates one first, or skip to local tracker (`context/tracker/BACKLOG.md` + `DONE.md`) temporarily; add Notion later
- **Highly regulated industry (medical, finance)** — add an extra "compliance review" checkpoint before any content is published, even SEO-only
- **Multi-entity / holding company** — treat each entity as separate project; set up portfolio layer (Phase 11)
- **CEO has low tolerance for setup** — run minimal viable deploy: phases 0-4 + 7-9, skip the research + governance depth, iterate later
- **Project is pre-launch / pre-product** — focus Phase 1-3 on the build, not the market; Phase 6 research is "market validation" not "competitor tracking"
- **Legacy CLAUDE.md or MEMORY.md already exists** — preserve, integrate, don't overwrite. Create a backup timestamp copy first.

## Integration with other skills

- **`research` / `review` / `questions` / `tidy` / `draft`** — all become available immediately after Phase 4 + 8 complete (need Notion DB + governance rules)
- **`wrap-session` / `resume-session`** — both work from Phase 3 onwards
- **`avoid-ai-writing`** — referenced by writing-rules section of CLAUDE.md
- **`superpowers:dispatching-parallel-agents`** — underlying pattern for Phase 6 (research seeding)

## What this skill does NOT do

- Does NOT replace a lawyer, accountant, or industry specialist for compliance-critical setup
- Does NOT build physical infrastructure (servers, factories, shops)
- Does NOT negotiate contracts, hire employees, or make investment decisions
- Does NOT guarantee compliance — surfaces known rules, flags unknowns, but humans own final compliance sign-off

## Related skills and references

- `~/.claude/skills/OPERATOR-LOOP.md` — the loop this skill deploys
- `~/.claude/skills/wrap-session/` + `~/.claude/skills/resume-session/` — session boundaries
- `~/.claude/skills/avoid-ai-writing/` — content quality filter
- `~/.claude/skills/deploy-project/references/` — sub-files used in each phase
- `~/.claude/skills/deploy-project/templates/` — starter files for scaffolding
