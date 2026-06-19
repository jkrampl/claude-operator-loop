# Portfolio architecture — privacy-first model

Detailed reference for Phase 11 of `/deploy-project`. Describes how a single CEO with multiple companies keeps per-company teams isolated while maintaining a consolidated portfolio view.

## Core principle

**Workspace isolation is the wall.** Everything else (ticket visibility, file access, integration scope) is enforced by which Notion workspace the user is a member of.

If you only take one thing from this doc: **one Notion workspace per company, never shared team membership across companies, CEO portfolio is its own separate workspace.**

## The three options (ranked)

### Option A — Notion portfolio workspace (recommended)

```
┌───────────────────────────────────────────────────────┐
│   CEO Portfolio Workspace                              │
│   Members: CEO only (+ up to 2 trusted delegates)     │
│                                                        │
│   Databases:                                          │
│   • Projects — one row per company                    │
│   • Weekly Digests — Claude auto-populates Mondays   │
│   • Escalations — cross-project issues               │
│   • Integration Map — which integration where        │
└───────────────────────────────────────────────────────┘
            ▲                  ▲                  ▲
            │                  │                  │
    Claude reads each, writes summary to portfolio
            │                  │                  │
┌───────────┴─────┐  ┌─────────┴───────┐  ┌──────┴──────┐
│ Company A       │  │ Company B        │  │ Company C    │
│ Workspace       │  │ Workspace        │  │ Workspace    │
│                 │  │                  │  │              │
│ Members:        │  │ Members:         │  │ Members:     │
│ • CEO           │  │ • CEO            │  │ • CEO        │
│ • A team (5-20) │  │ • B team (3-10)  │  │ • C team     │
│                 │  │                  │  │              │
│ Databases:      │  │ Databases:       │  │ Databases:   │
│ • Tasks DB      │  │ • Tasks DB       │  │ • Tasks DB   │
│ • Wiki pages    │  │ • Wiki pages     │  │ • Wiki pages │
│ • Campaigns     │  │ • Projects DB    │  │ • Campaigns  │
└─────────────────┘  └──────────────────┘  └──────────────┘
```

**Pros:** mobile access, shareable with delegates, automated weekly digest, full Notion query power.
**Cons:** one more workspace to maintain, Notion storage fees if scale gets large, slight integration complexity.

**Best for:** CEOs running 2-10 companies who want team members to have Notion access and need mobile/remote oversight.

### Option B — File-based portfolio (local only)

```
~/[CEO-handle]-portfolio/
├── README.md                          # portfolio overview
├── projects/
│   ├── company-a.md                   # mirrors the Notion Projects DB row
│   ├── company-b.md
│   └── [...]
├── digests/
│   ├── YYYY-MM-DD-weekly.md           # Claude writes here every Monday
│   └── [...]
├── escalations/
│   ├── YYYY-MM-DD-[issue-slug].md
│   └── [...]
└── operations/
    ├── integration-map.md             # which integration is authenticated where
    └── privacy-audit-log.md           # quarterly privacy audit results
```

**Pros:** zero Notion overhead at the portfolio layer, maximum privacy (no cloud for portfolio data), simple.
**Cons:** not mobile-accessible, harder to share with delegates, manual backup required.

**Best for:** privacy-maximalist CEOs, small portfolios (2-5 companies), CEOs who work primarily from one machine.

### Option C — Hybrid

```
Notion (thin layer):          Files (deep layer):
- Projects DB (status scan)   ~/[CEO-handle]-portfolio/
- Weekly Digest summaries     ├── digests/    (full detail)
                              ├── escalations/ (full detail)
                              └── operations/
```

Only the 1-line-per-project status scan + the most-recent weekly digest headline live in Notion. Full digests, full escalation briefs, integration maps, and privacy audits stay in files.

**Best for:** CEOs who want quick mobile scan ("any project on fire?") but keep sensitive detail off Notion.

## Privacy model — how isolation is enforced

### Notion's permission primitives

- **Workspace membership** — the top-level wall. A user who is not a member of Workspace X cannot see anything in Workspace X.
- **Page permissions** — within a workspace, pages can be restricted to specific users/groups. Useful for limiting CEO-only content within a shared workspace (less critical if you do Option A).
- **Integration scope** — integrations are added to a workspace and then explicitly granted access to specific pages/databases. An integration is workspace-scoped and cannot read sibling workspaces.
- **Guest access** — Notion allows inviting guests to specific pages without full workspace access. Useful for contractors or advisors.

### Claude integration scoping

When you authenticate the Notion integration (e.g. Claude Code's built-in Notion MCP) on a workspace:

1. The authentication is SPECIFIC to that workspace
2. You explicitly select which pages/databases the integration can access
3. The integration has NO visibility into sibling workspaces even if you're a member of both
4. To work across workspaces, authenticate SEPARATELY on each — same Claude Code, different auth per workspace

**Rule:** maintain an explicit integration map at `[portfolio]/operations/integration-map.md` listing: which Notion integration instance is authenticated on which workspace, what scopes it has, when last audited.

### Data flow rules

**Allowed flows:**
- Company workspace → Claude (authenticated there) → CEO portfolio workspace (authenticated there) — this is how digests get generated
- CEO portfolio → CEO (direct access by workspace membership)
- CEO personal devices → all workspaces (CEO has access to all)

**Blocked flows (by design):**
- Company A team → Company B data (not a member of B's workspace)
- Company A team → CEO portfolio (not invited to portfolio workspace)
- Integration in Workspace A → Workspace B (integration scope wall)
- CEO-written summary in portfolio → leaking back to Company A workspace (don't cross-link)

## Privacy audit procedure

Run this at Phase 11 setup + quarterly thereafter (add to `feedback_quarterly_register.md` anchors).

### Checklist

- [ ] List all workspaces in your Notion account. Confirm each is clearly labeled by company or as "Portfolio".
- [ ] For each company workspace, list members. Verify: no unexpected members, no sibling-company staff, integration members are documented.
- [ ] For the Portfolio workspace, verify members = CEO + at most 2 trusted delegates (named + role-documented).
- [ ] For each workspace, list connected integrations. Confirm each integration's scope (which pages/DBs it can read/write).
- [ ] Try to access Workspace B while logged in as a Workspace-A-only team member (test account if possible). Confirm: no visibility.
- [ ] Grep each company workspace for the portfolio workspace name or URL. Should be zero matches.
- [ ] Grep each company workspace for the name of any other company. Zero matches expected.
- [ ] Review the portfolio's Projects DB. Verify internal-only fields (revenue, FTE, cash) are NOT mirrored into any company workspace.
- [ ] Review the company-level Tasks DB + wiki. Verify no portfolio-level observations (e.g. "this is how Company B handles X" — would leak existence of Company B).
- [ ] For file-based portfolios: verify `~/[CEO-handle]-portfolio/` is NOT in any shared cloud folder accessible to team members.

Document audit results at `[portfolio]/operations/privacy-audit-log.md` with date + any findings + any remediations.

## Failure modes (real-world mistakes to avoid)

### 1. "Let me give the team just a view of the portfolio so they feel informed"

**Don't.** Even a read-only view leaks metadata: names of sibling companies, CEO's priorities, relative standings.

Instead: run a weekly per-company digest in that company's own workspace, written for their audience. Content-filtered per audience.

### 2. Cross-linking Notion URLs

When you paste a Portfolio workspace URL into a Company workspace comment or page, the link preview shows the target page name even to users who don't have access. This leaks the existence of the Portfolio.

**Fix:** never paste portfolio URLs into company-accessible content. If you must reference, describe in words ("this came up at the CEO level").

### 3. Over-broad integration authorization

When authenticating Claude's Notion integration on Workspace A, Notion offers "All pages in workspace" or "Select pages". Always choose Select pages and pick only the Tasks DB + wiki root.

**Why:** limits blast radius if integration is compromised + limits what Claude can accidentally read.

### 4. Shared email alias as workspace owner

If multiple people access the CEO's email (assistant, spouse, etc.), consider that they have portfolio access.

**Fix:** use a dedicated login for the Portfolio workspace that only the CEO personally logs into.

### 5. Forgotten archived pages

Archived pages in a workspace still exist and can be un-archived by admins. If sensitive data was moved to archive, it's still there.

**Fix:** delete sensitive archived pages outright (they're gone after 30 days). Don't rely on archive as isolation.

### 6. Portfolio entries with data the team should see

If you copy a project status line into the Projects DB and that line says something like "Team member X is underperforming", that's CEO-only context in the portfolio. Don't let it leak back when you generate company-facing digests.

**Fix:** the portfolio is one-way — data flows IN from companies (sanitized at read time), never OUT to companies. Any content written back to a company workspace is generated fresh for that audience.

## Multi-device considerations

**Notion:** cloud-hosted. Access from any device with your login. No sync concerns.

**File-based portfolio:** lives on one Mac by default. Options if you use multiple Macs:
- iCloud Drive / Dropbox / OneDrive — includes `~/[CEO-handle]-portfolio/` in the sync
- Git repo (private) — `cd ~/[CEO-handle]-portfolio && git init`, push to a private repo, pull on other Macs
- Manual rsync — CLI sync when needed
- Single "portfolio home" Mac — simplest, just always run portfolio work from one machine

**Scheduled tasks** (the MCP used for weekly digests) run on the machine they were created on. If you use multiple Macs, pick one Mac as the "portfolio home" for scheduled tasks.

**`~/.claude/skills/`** — this is where the operator-loop + deploy-project skills live. Sync this across Macs if you want the same skill set everywhere. Dotfiles sync (iCloud / GitHub / yadm / chezmoi) handles this cleanly.

## Graduation path: file → Notion → Notion + delegate

**Phase 1: 1-2 companies — file-based** is plenty. Set up `~/[handle]-portfolio/` and run weekly from one Mac.

**Phase 2: 3-5 companies — promote to Notion Option A**. You start wanting mobile access + quick scan + the Projects DB makes comparison easier.

**Phase 3: 5+ companies or hiring a COO / chief of staff — add delegates to the portfolio**. Use Notion's page-level permissions if you need finer control (e.g. delegate sees 3 of 8 projects).

**Phase 4: investment / fund side emerges — consider a SECOND portfolio workspace** for investments separate from operating companies. Same architecture, different data.

## Cross-project pattern extraction

As you deploy multiple projects, patterns repeat. Examples likely for you:

- Packeta / shipping-logistics setup (repeats for any EU DTC)
- Google Business management (repeats for any local-retail company)
- Shopify e-commerce hygiene (repeats for any DTC)
- Financial reporting cadence (repeats for any entity)
- Board-pack structure (repeats at holding level)

When a pattern appears in 2+ projects, consider promoting to a shared skill at `~/.claude/skills/[pattern-name]/`. Example candidates:

- `shopify-dtc-hygiene` — quarterly Shopify checklist for any DTC brand
- `google-business-manager` — how to maintain GBP for any local-retail project
- `packeta-shopify-setup` — the Packeta integration playbook
- `registeruz-peer-financials` — SK peer-company financial benchmark pull
- `family-office-quarterly-report` — if multiple entities need the same quarterly format

The goal: every repetition you do more than twice is a skill waiting to be extracted. Promotes leverage across the portfolio.

## When NOT to use a portfolio layer

- **One company** — skip Phase 11. Nothing to consolidate.
- **Multiple companies but truly independent + different CEOs per company** — each CEO runs their own portfolio. No shared layer.
- **Short-term projects (<6 months)** — portfolio overhead isn't worth it; manage ad-hoc.
- **Privacy-regulated domains where even CEO consolidation is risky** (certain medical or finance jurisdictions) — keep everything isolated, no cross-consolidation; rely on external roll-ups from qualified professionals.
