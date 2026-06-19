# Notion workspace architecture

Canonical pattern for ACME-style project deployments. Produces a Notion workspace that is the project's single source of truth for tasks, projects, campaigns, and wiki — while keeping code, raw data, generated artifacts, and Claude configs on local disk.

**Reference implementation:** Acme Co (ACME), deployed April 2026. Current state visible at https://www.notion.so/your-workspace (ACME — Home).

---

## Principle

Three rules decide where anything lives.

1. **Navigable, shareable, collaborative → Notion.** Knowledge, ticket state, campaign plans, drafts awaiting review. Anyone can browse, filter, search, comment, mention.
2. **Code, binary, generated, private-to-machine → local disk.** Scripts, Claude configs, .mcp.json, Excel/CSV sources, Chart.js HTML reports, images, session handoff markdown.
3. **One source of truth per artifact, always.** Never duplicate with active edits.

---

## The split — what lives where

| Artifact | Source of truth | Reasoning |
|---|---|---|
| Tasks / tickets | **Notion Tasks DB** | Team needs filter/search; CEO needs mobile; Claude writes via MCP |
| Wiki pages | **Notion Wiki DB + section landing pages** | Team browses like a website; Claude reads via MCP |
| Projects (initiatives) | **Notion Projects DB** | Tasks roll up to them; status visible |
| Campaigns | **Notion Campaigns DB** | Campaign state + approvals visible; asset files in local `output/campaigns/` |
| Drafts awaiting approval | **Notion pages** | Review workflow, comments, mentions |
| Research synthesis / findings | **Notion pages** | Human-readable, linkable, mobile |
| HTML reports with embedded Chart.js | **Local `output/reports/`**, linked from Notion | Too large (50KB–2MB+) for clean Notion rendering |
| Excel / CSV data | **Local `data/`**, linked from Notion or embedded via OneDrive paste-preview | Notion hosted-file URLs expire; source data must persist |
| Images / logos / binary assets | **Local `assets/`** | Same expiry issue; file system is better host |
| Python scripts, MCP servers, builders | **Local `context/scripts/`** | Code in file system |
| Claude configs (`.claude/`, `.mcp.json`) | **Local** | Claude-owned, can't move |
| `CLAUDE.md` agent instructions | **Local** | Read by Claude at session boot |
| `MEMORY.md` working memory | **Local** (lean index, not duplicating Notion) | Claude's session boot context |
| Session handoff docs | **Local `context/sessions/`** | Session-specific, read by Claude at boot |
| Session changelog `log.md` | **Local `context/wiki/log.md`** | Append-only, edited every session |
| Git history | **Local** | Version control belongs in VCS |

### The OneDrive exception (if CEO uses Microsoft 365)

OneDrive / SharePoint files CAN be embedded into Notion pages via paste-preview. This is a middle-ground for organizations that already have a shared drive:

- The file stays authoritative on OneDrive (versioning, team access, permissions)
- Notion shows a live preview when the Notion page is opened
- No upload to Notion — just a link

Use this for: reports the team needs to reference from Notion contexts (e.g. attach latest P&L Excel to the Financial Overview wiki page), shared spreadsheets, PDFs. See § "OneDrive / SharePoint integration" below.

---

## Workspace structure

```
[Workspace root]
└── [Project] — Home                    ← dashboard page, set as default landing
    ├── 🗄️ Databases (embedded)
    │   ├── 📋 Projects
    │   ├── ✅ Tasks
    │   ├── 📣 Campaigns
    │   └── 📚 Wiki
    ├── 🧭 Navigation
    │   ├── 🏢 Company                  ← section landing page
    │   ├── 👥 Customers                ← section landing page
    │   ├── 📊 Market                   ← section landing page
    │   ├── 📡 Channels                 ← section landing page
    │   ├── ⚙️ Operations              ← section landing page
    │   └── 📚 Wiki (meta-index)        ← section landing page
    ├── 🔁 How to Work Here             ← collapsible instructions
    ├── ⚙️ Setup Still To Do            ← manual-UI checklist
    └── 🗺️ [Project] Transition Plan   ← phased rollout doc (brownfield only)
```

### Why both a Wiki database AND section landing pages (hybrid)

The Wiki database gives you: filter by Section, filter by Status (Live/Draft), search, sort by Last Updated, views per category. Great for machines + power users.

The section landing pages give you: "this is the company section, here are the pages in it, click one." Great for humans who want to browse like a website.

Each wiki page is a row in the database AND reachable via `@mention-page` from its section landing page. One page, two entry points. Humans hit the landing page; Claude queries the DB.

---

## Database schemas

All schemas use the `mcp__notion-create-database` SQL DDL format (Notion flavored).

### Tasks DB

```sql
CREATE TABLE (
  "Name" TITLE,
  "Ticket ID" RICH_TEXT,
  "Category" SELECT('SEO':blue, 'Marketing':pink, 'Tech':purple, 'Ops':orange, 'Finance':green, 'Sales':red, 'Content':yellow, 'Admin':gray),
  "Priority" SELECT('P0':red, 'P1':orange, 'P2':yellow, 'P3':gray),
  "Status" SELECT('Backlog':gray, 'Todo':blue, 'In Progress':yellow, 'Blocked':red, 'In Review':purple, 'Done':green, 'Archived':default),
  "Source" SELECT('Backlog':gray, 'Weekly Plan':blue, 'Campaign':purple, 'Ad-hoc':orange, 'Recurring':yellow),
  "Owner" RICH_TEXT,
  "Start Date" DATE,
  "Due Date" DATE,
  "Done date" DATE,
  "Estimated Hours" NUMBER,
  "Actual Hours" NUMBER,
  "Dependencies" RICH_TEXT,
  "Blocked By" RICH_TEXT,
  "Project" RICH_TEXT,
  "Project link" RELATION('[Projects DB data source ID]', DUAL 'Linked Tasks'),
  "Campaign" RICH_TEXT,
  "Campaign" RELATION('[Campaigns DB data source ID]', DUAL 'Tasks'),
  "Reference" URL,
  "Reflection" RICH_TEXT
)
```

**Industry-aware Category values:** swap in per-industry options during deploy. Medical: Clinical/Admin/Ops/Finance/Research. Interior design: Design/Procurement/Install/Client/Admin. Manufacturing: Production/QC/Supply/Sales/Admin.

**Views to create via Notion UI (API can't do this):**
- Today — filter: Due Date = today, Status ≠ Done
- This Week — filter: Due Date in next 7 days, group by day
- Kanban by Status — board view
- By Category — board view, grouped by Category
- By Priority — board view, grouped by Priority
- Calendar — on Due Date
- My Tasks — filter: Owner = [CEO name]
- Blocked — filter: Status = Blocked
- Done this week — filter: Status = Done, Done date in last 7 days

### Task body conventions (HARD RULE)

Every Notion task must be **self-contained AND connected**. This rule applies at task-creation time AND when picking up an existing thin task.

**1. Self-contained body.** A team member with ONLY Notion access — no wiki, no output folders, no campaign files, no chat history — must be able to execute the task end-to-end from what's in the body. Concretely:

- Inline the actual copy, asset URLs, step-by-step instructions, source paths, approval state, and context the executor needs
- Never write "see X for detail" or "draft at [path]" without ALSO including the relevant content directly in the body
- For asset references: include the file path AND a description (size, format, what it shows). The path is for archival; the inline description is so it works without file access
- For approval status: write it explicitly ("Awaiting CEO approval as of [date]" or "Pre-approved per board decision [date]")
- For brand / writing rules that constrain the work: list them in the body, don't assume the executor will go read CLAUDE.md

**2. Related tasks linked.** Use the relation properties on the Tasks DB:
- **Depends on** — sibling tickets in the same campaign, initiative, or topic
- **Project link** — parent project (Projects DB)
- **Campaign** — parent campaign (Campaigns DB) — created as DUAL relation; auto-syncs with "Tasks" property on Campaigns side

Anything connected gets connected. Example chain: a campaign blog → its social posts → its GBP post → its newsletter slot. All four get linked via Depends on; all four also link to the parent Project + Campaign.

**Why:** Tasks created as thin pointers ("see file X") are useful only when Claude or the CEO picks them up. As soon as a team member without file access tries to execute, they're blocked. Self-contained bodies make Notion the genuine single pane of glass — which is the entire reason the system was deployed.

**Enforcement points:**
- Every operator-loop skill that creates tickets enforces this at creation time. See `~/.claude/skills/OPERATOR-LOOP.md` § "Self-contained ticket bodies + relation linking".
- `/tidy` flags thin-bodied tickets in Stage A3 for repopulation
- When picking up a thin ticket: populate the body FIRST with full execution detail, THEN execute the work. Mandatory, not optional.

Per-project rule reference: `feedback_notion_task_self_contained.md` (codify in each deployed project's memory folder).

### Wiki DB

```sql
CREATE TABLE (
  "Name" TITLE,
  "Section" SELECT('Company':blue, 'Customers':green, 'Market':red, 'Channels':orange, 'Operations':purple, 'Root':gray),
  "Status" SELECT('Live':green, 'Draft':yellow, 'Needs refresh':orange, 'Archived':gray),
  "Summary" RICH_TEXT,
  "Owner" RICH_TEXT,
  "Original path" RICH_TEXT,
  "Last meaningful update" DATE,
  "External refs" URL
)
```

**Views:**
- By Section — grouped by Section
- Recently Updated — sorted by Last meaningful update desc
- Drafts — filter: Status = Draft
- Needs refresh — filter: Status = Needs refresh

### Projects DB

```sql
CREATE TABLE (
  "Name" TITLE,
  "Status" SELECT('Active':green, 'On Hold':yellow, 'Complete':blue, 'Cancelled':gray),
  "Stage" SELECT('Discovery':gray, 'Planning':blue, 'Building':yellow, 'Rolling Out':orange, 'Steady State':green),
  "Start Date" DATE,
  "Due Date" DATE,
  "Owner" RICH_TEXT,
  "Summary" RICH_TEXT,
  "Linked Tasks" RELATION('[Tasks DB data source ID]', DUAL 'Project link')
)
```

### Campaigns DB

```sql
CREATE TABLE (
  "Name" TITLE,
  "Status" SELECT('Planning':gray, 'Approved':blue, 'Running':yellow, 'Complete':green, 'Cancelled':red),
  "Start Date" DATE,
  "End Date" DATE,
  "Budget" NUMBER FORMAT 'euro',
  "Channels" MULTI_SELECT('Email':blue, 'Social':pink, 'Paid Ads':red, 'PR':purple, 'SEO':green, 'Events':orange, 'Partnerships':yellow),
  "Owner" RICH_TEXT,
  "Summary" RICH_TEXT,
  "Tasks" RELATION('[Tasks DB data source ID]', DUAL 'Campaign')
)
```

**Two-step relation creation:** Notion requires the other DB to exist first. Create in this order: 1) Projects, 2) Campaigns, 3) Tasks (with RELATION pointing to Projects + Campaigns), 4) Wiki.

---

## Home dashboard anatomy

See `templates/notion-home-dashboard.md.template` for the full markdown. Key blocks:

1. **Hero callout** (icon: project emoji, bg: purple_bg). One sentence: company name · location · URL · positioning stat.
2. **🔥 This Week callout** (icon: 📅, bg: red_bg). Manually updated each Monday with P0 priorities. Points to "Tasks filtered by Source = Weekly Plan".
3. **🗄️ Databases section** — 2 columns × 2 rows = 4 embedded databases. Each uses `<database url="..." inline="false" data-source-url="collection://..."/>` block.
4. **🧭 Navigate section** — 3 columns × 2 rows = 6 callout cards. Each card is a colored callout with page link. Use `<page url="..."/>` inline.
5. **🔁 How to Work Here** — 3 collapsible toggles: Daily routine, Weekly Friday review, Running campaigns. Covers the expected workflow.
6. **⚙️ Setup Still To Do** — 2 collapsible toggles with checkboxes. Lists manual UI steps the API can't do (views, permissions, default landing page, delete the onboarding "Notion Basics" page).

---

## Section landing pages

Each of the 5–6 section landing pages follows the same template:

```markdown
<page icon="🏢">
  <callout icon="🏢" color="blue_bg">
    [One sentence: what this section covers.]
  </callout>

  ## Pages in this section
  - 🏆 <mention-page url="[URL of Wiki DB row 1]"/> — [short reason]
  - 🎨 <mention-page url="[URL of Wiki DB row 2]"/> — [short reason]
  - ...

  ---
  ### Other sections
  👥 Customers · 📊 Market · 📡 Channels · ⚙️ Operations
</page>
```

This is the "book" layer. The Wiki database underneath is the "library" layer. Both are valid entry points.

---

## Why no local mirror sync

**The anti-pattern to avoid:** Creating a separate Notion internal integration, writing a Python script that pulls Notion → local BACKLOG.md/DONE.md/IDEAS.md mirror files, and configuring SessionStart + Stop hooks to run the sync on every session boundary.

**Why this is wrong:**

1. **The hosted Notion MCP already gives Claude direct read/write access** to any Notion page in any session. No second integration needed.
2. **Adding a sync integration = second token to manage, second point of failure, second permission boundary to maintain.**
3. **Sync creates drift risk.** If Claude writes to Notion during session A, then someone edits Notion between sessions, then Claude reads local mirror in session B — Claude sees stale data. Direct MCP reads are always fresh.
4. **The mirror files encourage grepping local over querying Notion** — but Notion queries are cheap and always current.
5. **Adds ~60 lines of Python + hook config + backup rotation for zero benefit.**

**ACME's lesson (Apr 19–20, 2026):** Claude built exactly this — a separate "ACME Sync" integration with a sync script + SessionStart/Stop hooks — before realizing the hosted MCP was already sufficient. Rolled back the same day. Archive kept at `context/archives/notion-sync-rolled-back-260420/` as a cautionary reference.

**The right pattern:** Claude queries Notion directly via the hosted MCP whenever it needs state. No mirror files. Old local `BACKLOG.md` / `DONE.md` / `IDEAS.md` can stay as frozen legacy reference during the transition, then archive at cutover.

---

## Integration model

**Always use the hosted Notion MCP that ships with Claude Code.**

- Connected to the CEO's Notion account during initial Claude Code setup
- One token, managed by Claude Code
- Accessible to Claude in every session, no per-project auth needed
- Permissions follow Notion's native model — whatever pages the CEO's account has access to
- Tool names: `mcp__[hash]__notion-search`, `notion-fetch`, `notion-create-pages`, `notion-update-page`, `notion-create-database`, `notion-query-data-source`, etc.

**Do not create:** a second internal integration (`notion.so/my-integrations → New integration`) just for this project. That's only justified for: a) standalone scripts that run outside Claude sessions (CI jobs, scheduled tasks that must run without Claude), or b) team members needing programmatic access. For ordinary Claude-driven work, the hosted MCP is always sufficient.

---

## Cloud file storage integration

The choice of file backend is decided in Phase 1 intake (Section H question 2). Every project picks ONE primary backend. Notion supports paste-preview for all three major backends — the flow is nearly identical, only the integration identifier differs. The CEO's answer in Phase 1 determines which one gets demoed + templated in Phase 5.

### Backend-by-backend quick reference

| Backend | When it fits | Tier 1 slash command | Tier 2 (AI connector) | Tier 3 (automation) |
|---|---|---|---|---|
| **Google Drive / Google Workspace** | Tech / startups / creative agencies; already on Gmail + Calendar + Docs | `/gdrive` or paste share link | Google Drive AI Connector (Notion Business+) | Zapier / Make / n8n |
| **OneDrive / SharePoint / M365** | SMEs, family offices, regulated (finance / medical / legal), Outlook + Office users | `/onedrive` or paste share link | SharePoint & OneDrive AI Connector (Notion Business+, beta) | Zapier / Make / n8n |
| **Dropbox** | Legacy preference, small orgs, mixed-platform teams | Paste share link (no dedicated slash command) | No native Notion AI connector | Zapier / Make / n8n |
| **Local only** | Solo operators, very early stage, paranoid-private | N/A — file paths in page body only | N/A | N/A |
| **Hybrid (rare)** | Split: OneDrive for finance/legal, Google Drive for creative | Both slash commands work | Both connectors can coexist (one per product) | One automation per flow |

### Tier 1 — Paste-preview embeds (zero setup, use immediately, all backends)

**How:** In any Notion page, paste a Google Drive / OneDrive / SharePoint / Dropbox share link → Notion prompts "Paste as preview" → select it → live embedded viewer appears. Or type the backend-specific slash command (`/gdrive`, `/onedrive`) for a file picker (first-use OAuth required).

**Use for:** attaching specific files to specific Notion pages. E.g., the latest P&L Excel embedded in the Financial Overview wiki row; the branded HTML report embedded in the relevant campaign row.

**Updates:** re-open the Notion page → preview reflects the current cloud version.

**Limitations:** one file per embed; not full-text-searchable inside Notion; doesn't expose Excel/CSV data to Notion formulas; for locked-down files the sharing permission needs to be at least "People with the link" (not "People you specify") for Notion to preview.

### Tier 2 — AI Connector (Notion Business+ tier)

**What it does:** Notion AI indexes the cloud's contents (respects source permissions) on a cadence (hourly for MS, similar for Google). Enables semantic search across cloud files from inside Notion AI prompts.

**Backend-specific:**
- Google Drive AI Connector — stable
- SharePoint & OneDrive AI Connector — still marked **beta**; see https://www.notion.com/help/notion-ai-connector-for-microsoft-sharepoint-and-onedrive
- Dropbox — no native AI connector; use Tier 3 (automation) if cross-search is needed

**Use for:** team-wide "find me the slide deck about X" across cloud files + Notion pages in one query.

**Defer until:** team onboarding phase (when multiple team members need cloud-file search from Notion). For CEO-only projects, Tier 1 is sufficient through cutover.

### Tier 3 — Automation platforms (Zapier, Make, n8n, Pipedream)

**What:** trigger-based flows like "new file uploaded to folder X → create Notion row in DB Y with metadata" or "Notion status changed to 'Ship' → copy file from template folder to delivery folder".

**Use for:** concrete recurring workflows only. Don't set up preemptively. Backend-agnostic.

**Cost:** most have free tiers for low-volume automations.

### Recommended deployment pattern (any backend)

**Tier 1 on Phase 4 day.** The CEO picks a real file from the chosen cloud → creates a share link → Claude creates a Notion page that expects that file → CEO pastes the link → "Paste as preview" confirms the flow. Demo takes 2 minutes. The Notion page becomes the template for future embeds.

Revisit Tier 2 during Phase 4 team onboarding (typically Weeks 5–6 of the ACME-style transition plan). Skip Tier 3 unless a specific recurring workflow demands it.

### Document the chosen backend

The backend choice goes into `CLAUDE.md` governance section so future sessions + agents know which cloud to target:

```markdown
## File storage backend

Primary: [Google Drive | OneDrive / SharePoint | Dropbox | Local only]

When referencing files from Notion:
- Share links always from: [chosen backend]
- Never mix backends unless explicitly a Hybrid project (rare, see architecture doc)
```

### ACME reference point

ACME chose **OneDrive / Microsoft 365** (April 2026) because:
- Sales @example.com is a Microsoft 365 tenant
- Existing shared drive already at `/Users/jakubkrampl/Library/CloudStorage/OneDrive-YourOrg/ACME/`
- Team (Jana, Katarína, accountants) already use Outlook + Office
- Slovak regulated industries commonly default to Microsoft stack

Demo page + embedding rollout pattern documented at https://www.notion.so/your-workspace. AI Connector upgrade planned for Phase 4 team onboarding (May 18–31, 2026).

---

## Known issues + workarounds

From ACME's Apr 2026 deployment.

| Issue | Severity | Workaround | Permanent fix |
|---|---|---|---|
| Markdown tables lose rendering during bulk migration (become pipe-text paragraphs in some pages) | Medium | Identify affected pages via `grep -l '|.*|.*|' context/wiki`. Manually fix in Notion UI post-migration (convert to native tables). | Better markdown→Notion converter; or migrate tables page-by-page via API |
| Notion views cannot be created via API | Low | 15-min manual UI setup per database after Phase 4 creation | None — accept UI workflow |
| Moving a page breaks integration permission inheritance | Medium | Re-add Claude connection at the new parent. Always connect at workspace root, not leaf pages | Connect integration at workspace-root level only |
| Recurring tasks are one-off dates (no native repeat) | Medium | Weekly Claude scheduled task recreates them from a template | Use a dedicated recurring-task tool (e.g., Notion + cron via MCP) |
| Notion hosted file URLs expire (~1h) | High | Do NOT link to Notion file URLs externally. Use OneDrive embeds for durable file references | Host binaries on OneDrive/SharePoint/S3/Vercel; Notion links to those |
| Token in chat history (if a second integration is created unnecessarily) | High | Rotate immediately. Don't create second integration — use hosted MCP | Never paste integration tokens into chat; use env vars or ~/.token files |
| "The Notion Basics" onboarding page persists as orphan parent | Low | Delete via UI after setup | N/A |

---

## Privacy + workspace isolation

For CEOs with 2+ projects: see `portfolio-architecture.md` for the full multi-workspace privacy model. Key rule: **separate Notion workspace per project**. Team members in Project A cannot access Project B. CEO-only portfolio workspace aggregates across projects.

Single-project CEOs can use one workspace for everything — the split only matters when multiple companies + team members are involved.

---

## References and sources

- Notion docs: https://developers.notion.com/docs/working-with-databases
- Notion SDK: `mcp__notion-*` tools (hosted MCP)
- ACME reference implementation: https://www.notion.so/your-workspace
- ACME transition plan doc: https://www.notion.so/your-workspace
- OneDrive Notion help: https://www.notion.com/integrations/onedrive
