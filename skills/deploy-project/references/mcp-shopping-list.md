# MCP shopping list — recommended integrations per industry

Which MCPs to install during Phase 5 of `/deploy-project`. Load per-industry + pick the "always" core + apply judgment.

## Always install (every project, every industry)

| MCP | Purpose | Criticality |
|---|---|---|
| **Notion** (`mcp__notionApi__*`) | Tasks DB, wiki-adjacent content | Critical |
| **Scheduled tasks** (`mcp__scheduled-tasks__*`) | Recurring automations, digests, reminders | Critical |
| **Google Calendar** (if available) | Capacity checks in `/tidy`, meeting awareness | High |
| **Claude-mem** or equivalent (`mcp__claude-mem__*`) | Cross-session memory search | Medium |

## E-commerce / DTC (ACME pattern)

| MCP | Purpose |
|---|---|
| **Shopify Admin** (`shopify-plugin:shopify-admin` + execution) | Read + write to store |
| **Google Analytics 4** (`mcp-server-google-analytics`) | Traffic, conversion, behavior |
| **Google Search Console** (`mcp-server-gsc`) | Search performance, indexing |
| **Meta organic** (custom Python MCP or equivalent) | FB/IG insights, post-performance |
| **Microsoft Clarity** (`@microsoft/clarity-mcp-server`) | Session recordings, heatmaps |
| **Klaviyo / Shopify Email** (when API access) | Email metrics |

Optional: Shopify Inbox, Google Merchant Center (feed health), Facebook Ads (spend), PayPal/Stripe (payments if Shopify Payments not used).

## Tech / SaaS

| MCP | Purpose |
|---|---|
| **GitHub** (`mcp__github__*`) | Repo, PR, issues, CI/CD |
| **Linear / Jira** | Engineering workflow (if separate from Notion) |
| **Stripe** | Subscription, billing, MRR tracking |
| **Sentry** | Error tracking |
| **Segment / Mixpanel / Amplitude / PostHog** | Product analytics |
| **Slack** | Team comms observability |
| **Intercom / Crisp** | Customer support metrics |

Optional: AWS / GCP / Azure for infra; Vercel / Netlify for deploys; Statuspage for reliability; dependencies audit (Snyk / Dependabot).

## Medical / healthcare

**Caution:** PHI routing through general-purpose MCPs is usually NOT compliant. Verify each MCP's data handling before connecting.

| MCP | Purpose |
|---|---|
| **Notion** (de-identified admin only) | Admin tasks, ops |
| **Accounting** (QuickBooks, Xero, local equivalent) | Non-clinical finance |
| **Scheduling** (Calendly, Doctolib, system-specific) | Booking + availability |

**Typically NOT installed:** generic email MCP, file-share MCPs that could touch PHI, analytics tools that observe patient journeys without BAA.

Clinical systems (EMR, LIMS, imaging) usually have their own APIs — integrate only with explicit compliance sign-off.

## Interior design / architecture

| MCP | Purpose |
|---|---|
| **Notion** | Project management, client docs |
| **Google Drive / Dropbox / OneDrive** | Asset storage, renders, specs |
| **Accounting** | Time tracking, invoicing |
| **Asana / Monday** (if used) | Project workflow |

Optional: Houzz Pro, Studio Designer, Designer Pages for trade partnerships. CAD tools typically have no MCP yet; integrate via file exports.

## Food & beverage / restaurant / winery

| MCP | Purpose |
|---|---|
| **Shopify / Squarespace / WooCommerce** | DTC sales |
| **POS** (Toast, Square, Lightspeed) | Retail sales |
| **Reservation** (OpenTable, Resy, Tock) | Booking data |
| **Google Business** | Local SEO, reviews |
| **Delivery platforms** (Uber Eats, Wolt, Bolt, DoorDash) | Third-party sales channels |
| **Inventory** (MarketMan, BlueCart, Shopify inventory) | Stock levels |

## Finance / investment / fund

**Caution:** similar to medical. Client financial data needs strict compliance review.

| MCP | Purpose |
|---|---|
| **Notion** (non-client-data) | Internal ops |
| **CRM** (Wealthbox, Redtail) — check compliance | Client relationship metadata |
| **Accounting** | Firm finance |
| **Calendar** | Client meeting ops |

**Typically NOT installed:** trading APIs (too risky), client-data lakes (compliance), performance reporting (compliance).

## Family office

Minimal MCP footprint. Each integration needs explicit principal sign-off.

| MCP | Purpose |
|---|---|
| **Notion** (admin only) | Non-financial admin |
| **Calendar** | Schedule coordination |
| **Document management** (Box / Vaultime) — with care | Non-sensitive docs |

**Default posture:** install nothing beyond absolute necessity. Add only when a specific use case justifies and principal approves.

## Manufacturing / industrial

| MCP | Purpose |
|---|---|
| **Notion** | Admin, ops tracking |
| **ERP** (SAP, NetSuite, Dynamics — if API) | Production, inventory, orders |
| **Shipping / logistics** (FedEx, DHL, freight partners) | Shipment tracking |
| **Quality mgmt (QMS)** (if API) | Non-conformance, audits |
| **Accounting** | Cost tracking |

Optional: supplier portal integration, IoT / sensor data (for smart factories), customer portal.

## Professional services / consulting / agency

| MCP | Purpose |
|---|---|
| **Notion** | Tasks, client workspace |
| **CRM** (HubSpot, Salesforce, Pipedrive) | Pipeline |
| **Time tracking** (Harvest, Toggl, Clockify) | Billable hours |
| **Accounting** | Invoicing, revenue |
| **Document mgmt** (Google Drive, Box) | Deliverables |

## Installation sequence

Within Phase 5 of `/deploy-project`:

1. **Always-install core** first (Notion + scheduled tasks + calendar)
2. **Industry core** next — the 3-5 most important per industry
3. **Optional / experimental** last — only if CEO actively wants them

For each MCP:

1. CEO reads the MCP's docs/permissions
2. CEO authenticates (OAuth flow or token)
3. Claude runs a smoke-test read query
4. Document in `context/wiki/operations/integrations.md` with: MCP name, auth method, scopes, smoke-test date, connected-by (CEO name), status

## Smoke-test templates

### Notion
```
API-post-search query="[project handle]" page_size=1 → verify returns
```

### Scheduled tasks
```
list_scheduled_tasks → verify empty list or existing tasks visible
```

### Shopify
```
shopify store execute --query 'query { shop { name myshopifyDomain } }'
```

### GA4
```
runReport on property → current-month sessions → verify non-zero
```

### GitHub
```
list repos → verify at least 1 visible
```

## Red flags during MCP install

- **OAuth flow never completes** — token or scope issue; don't retry blindly, read docs
- **Smoke test returns empty when data should exist** — usually scope issue, not auth
- **MCP crashes repeatedly** — server-side issue; file support ticket, park the MCP
- **Data returned has PII you weren't expecting** — STOP, review compliance before any further use

## Deprecation awareness

MCPs evolve. Check `~/.claude/plugins/` or equivalent for version info, and note in integrations wiki:

- Date installed
- Version at install
- Known breaking changes upcoming
- Quarterly re-verification (part of `feedback_quarterly_register.md`)
