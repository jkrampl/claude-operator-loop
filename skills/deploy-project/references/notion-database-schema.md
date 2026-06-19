# Notion Tasks Database — schema specification

The canonical Tasks DB that all operator-loop skills read and write. Used across every project deployed by this runbook.

## Database creation

CEO creates in Notion via UI (faster than API for initial schema).

**Name:** `[Project handle] Tasks` (e.g. "ACME Tasks", "AlphaStudio Tasks")

**Parent:** project workspace root page

**Integration:** connect Claude integration to the database BEFORE Phase 9 testing.

## Required properties

| Property name | Type | Description | Required values |
|---|---|---|---|
| **Name** | Title | Human-readable ticket title. Concrete and action-oriented ("Draft X", "Approve Y", "Decide Z"). | Freetext |
| **Ticket ID** | Rich text | Stable ID following project convention. | `[PREFIX]-[CAT]-[###]` e.g. `ACME-MKT-071`, `ACME-TECH-005` |
| **Project** | Rich text | Project handle. Allows portfolio-level filtering. | Freetext — project handle |
| **Owner** | Rich text | Who executes. Use the 5-item closed list. | `Claude` / `Claude (post-approval)` / `CEO` / `External` / `[Team member name]` |
| **Priority** | Select | Urgency + importance. | `P0` (red) / `P1` (orange) / `P2` (yellow) / `P3` (gray) |
| **Category** | Select | Domain bucket. Extend per industry. | Typical: `Marketing`, `SEO`, `Tech`, `Sales`, `Operations`, `Finance`, `Product`, `Legal`, `Compliance`, `Content`, `Admin` |
| **Status** | Select | Pipeline stage. | `Todo` / `In progress` / `In review` / `Done` / `Cancelled` / `Deferred` / `Blocked` |
| **Source** | Select | Where the ticket came from. | `Backlog`, `Research sweep`, `Questions upload`, `Recurring`, `Campaign`, `Ad-hoc`, `Weekly plan` |
| **Start Date** | Date | Planned start. | ISO date |
| **Due Date** | Date | Hard deadline. Null = no hard deadline. | ISO date |
| **Done date** | Date | Actual completion date. Auto-set by /review. | ISO date |
| **Dependencies** | Rich text | Ticket IDs this depends on (comma-separated). Freetext to avoid Notion relation complexity. | e.g. `ACME-OPS-041, ACME-TECH-024` |
| **Blocked By** | Rich text | Description of the blocker or context. Often used as the ticket body too. | Freetext, can be long |
| **Estimated Hours** | Number | Realistic estimate in hours. | Decimal OK (e.g. 0.25 for 15 min) |
| **Actual Hours** | Number | Filled at close. Used for velocity + estimate calibration. | Decimal |
| **Deliverable URL** | URL | Link to the output (file path, Figma, Shopify URL, external doc). | URL |
| **Reflection** | Rich text | Post-close note: what we learned, anything to change next time. | Freetext, often empty |
| **Campaign** | Rich text | Campaign name if ticket belongs to one (e.g. "Vinalies 2026"). | Freetext |

## Optional properties (add per project)

- **Accuracy %** (formula) — actual vs estimated hours (for velocity calibration)
- **Depends on** (relation) — relational version of Dependencies, if you want Notion's graph view
- **Campaign link** (relation) — link to a Campaigns DB if the project uses one
- **Project link** (relation) — link to a Projects/Initiatives DB
- **Reference** (URL) — external doc or research reference
- **Place** (place type) — for ops tickets with a specific location
- **Confidential** (checkbox) — restricts visibility in shared views

## Required views

### 1. Kanban — main daily view

- **Layout:** Board
- **Group by:** Status
- **Columns ordered:** Todo / In progress / In review / Blocked / Done / Deferred / Cancelled
- **Filters:** Project = [this project handle] (for portfolio views filter by project here)
- **Sort:** Priority ascending, then Due Date ascending
- **Properties shown on card:** Ticket ID, Priority, Owner, Due Date

### 2. This Week — what's active now

- **Layout:** Table
- **Filters:** Start Date ≤ end of this week AND Status != Done AND Status != Cancelled
- **Sort:** Start Date, Priority
- **Properties shown:** Name, Ticket ID, Priority, Owner, Start Date, Due Date, Status

### 3. Backlog — everything not yet started

- **Layout:** Table
- **Filters:** Status = Todo
- **Sort:** Priority ascending, Created ascending
- **Group by:** Category
- **Properties shown:** Name, Ticket ID, Priority, Owner, Estimated Hours

### 4. In Review — CEO decision queue

- **Layout:** Table
- **Filters:** Status = In review AND Owner contains "CEO"
- **Sort:** Due Date ascending
- **Properties shown:** Name, Ticket ID, Due Date, Deliverable URL

### 5. Done — recent completions

- **Layout:** Table
- **Filters:** Status = Done AND Done date is within the past 30 days
- **Sort:** Done date descending
- **Group by:** Category (collapsed)
- **Properties shown:** Name, Ticket ID, Category, Owner, Done date, Actual Hours

### 6. Blocked — stuck tickets needing attention

- **Layout:** Table
- **Filters:** Status = Blocked
- **Sort:** Priority ascending
- **Properties shown:** Name, Ticket ID, Priority, Owner, Blocked By

### 7. By Owner — team view

- **Layout:** Board
- **Group by:** Owner
- **Filters:** Status != Done AND Status != Cancelled
- **Sort:** Priority ascending within each group

### 8. Velocity — closed work

- **Layout:** Timeline or calendar
- **Filters:** Status = Done AND Done date is within this quarter
- **Use:** /tidy velocity stage reads this view

### 9. Campaigns (optional) — if project runs named campaigns

- **Layout:** Board
- **Group by:** Campaign
- **Filters:** Campaign is not empty AND Status != Done

## Ticket ID convention

Format: `[PROJECT-PREFIX]-[CATEGORY]-[###]`

- **PROJECT-PREFIX:** 3-5 letters, uppercase, no special chars (e.g. ACME, ACME, ALPHA). Pick at Phase 0; stable forever.
- **CATEGORY:** 3-4 letters abbreviating the Category property. Map:
  - Marketing → `MKT`
  - SEO → `SEO`
  - Tech → `TECH`
  - Sales → `SLS`
  - Operations → `OPS`
  - Finance → `FIN`
  - Product → `PRD`
  - Legal → `LGL`
  - Compliance → `COM`
  - Content → `CON`
  - Admin → `ADM`
- **###:** zero-padded 3-digit sequence per category, starting at 001. Never reuse numbers.

Additional format for week-scheduled one-offs: `[PROJECT-PREFIX]-WK[NN]-[DAY]-[N]` (e.g. `ACME-WK16-MON-3` = project ACME, calendar week 16, Monday's 3rd scheduled task).

## Ownership values (the 5-item closed list)

Enforced across all operator-loop skills + `/deploy-project`.

| Value | Meaning | When Claude uses this |
|---|---|---|
| `Claude` | Autonomous execution | Research, drafting, analysis, read-only data pulls |
| `Claude (post-approval)` | CEO approves once, then Claude executes via MCP/tools | Shopify writes, scheduled sends, Notion batch-updates, any action that touches external state after approval |
| `CEO` | Manual CEO-only work | UI clicks in external systems, human meetings, board decisions, judgment calls |
| `External` | Dev / designer / vendor / contractor | Code changes, design work, engineering, specialist tasks |
| `[Team member name]` | Named person | Specific operational tasks assigned to team member |

## Status lifecycle

```
Todo → In progress → In review → Done
       ↓              ↓            ↓
       Blocked        Cancelled   (terminal)
       ↓              (terminal)
       Deferred
       ↓
       Todo (after unblock)
```

- Tickets can cycle back (In review → In progress for revisions)
- Deferred has a due-date reset
- Cancelled is terminal with reason in Blocked By / Reflection

## Source values

- **Backlog** — legacy or manually-added
- **Research sweep** — auto-created by `/research`
- **Questions upload** — auto-created by `/questions upload`
- **Recurring** — created by cadence (weekly/monthly/quarterly/annual)
- **Campaign** — created within a specific campaign plan
- **Ad-hoc** — user created in the moment
- **Weekly plan** — created during `/tidy` rebalance

## Archival policy

- Done / Cancelled / Deferred tickets older than 90 days can be archived (Notion archive, not deleted)
- Archived tickets stay queryable for velocity + historical lookup
- Never delete tickets (audit trail matters)

## Integration sanity test

After DB creation, Claude runs this smoke test:

1. Create one test ticket via `mcp__notionApi__API-post-page` — Name "DEPLOY-PROJECT smoke test", all required properties filled, Status=Todo, Owner=Claude
2. Search for it via `mcp__notionApi__API-post-search` — verify returned
3. Update status to Done via `mcp__notionApi__API-patch-page` — verify Done date set
4. Archive it via API (in_trash: true) — verify it disappears from active views

If all four pass, DB is operational.
