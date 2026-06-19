# Discovery questionnaire — Phase 1 intake

Full business Q list for the Phase 1 intake interview. Walk through one section at a time, not all at once. CEO should block 30-60 min. Capture answers to `context/intake/[YYYY-MM-DD] - intake interview.md`.

For every answer, tag:
- **Public** — already known externally, safe to use in any output
- **Internal** — proprietary, never leaks to public-facing content
- **Confidential** — limit access beyond project team

If CEO doesn't know an answer, mark **[GAP]** and auto-create a research or clarification ticket.

---

## Section A — Company basics

1. Legal entity name (as registered)?
2. Trading name (if different)?
3. Country of registration + registration ID (e.g. IČO, company number, EIN)?
4. Legal form (a.s. / s.r.o. / Ltd / Inc / LLP / sole trader / partnership / trust)?
5. Year founded / year of current legal entity?
6. Registered address?
7. Operating address (if different)?
8. VAT / tax ID + scope (domestic only / intra-EU / global)?
9. Industry classification (primary NACE / NAICS / SIC if known)?
10. Parent company / holding structure if any?
11. Subsidiaries or sister entities that interact with this project?

## Section B — Product or service

1. What do you sell? (One-sentence elevator pitch)
2. Product categories / SKU structure (summary, not full list)?
3. Unit economics per category (price, COGS, gross margin) — **Internal**
4. Price architecture (entry / mid / premium tiers)?
5. Seasonality pattern (peak months, trough months)?
6. Production / delivery method (in-house / outsourced / hybrid)?
7. Lead time customer orders → fulfilled?
8. Inventory model (make-to-stock / make-to-order / services only)?
9. Any discontinued or retiring products in next 12 months?
10. Any new launches planned next 12 months?

## Section C — Customers + segments

1. Primary customer segment(s)?
2. Ideal customer profile (demographics / psychographics / firmographics)?
3. Top 5 customers by revenue — **Internal**
4. Customer concentration risk (is any customer >10% of revenue?) — **Internal**
5. Customer acquisition channels today (paid / organic / referral / trade / chains)?
6. Customer retention model (subscription / repeat-buy / one-off / long-cycle B2B)?
7. Average order value / average contract value?
8. Typical customer lifetime value?
9. Top 3 reasons customers buy you (verified from data, not assumptions)?
10. Top 3 reasons customers leave?

## Section D — Market + competition

1. Who are your top 5-10 direct competitors (name + URL)?
2. Who are your top 3 indirect competitors (different solution, same problem)?
3. What differentiates you (positioning statement)?
4. TAM / SAM / SOM estimates if available?
5. Market trends — is the category growing, stable, shrinking?
6. Regulatory context — what laws / standards / licenses apply?
7. Any market consolidation / distress among peers? — often valuable
8. Industry associations / trade bodies / certifications the company holds or targets?
9. Press coverage (last 12 months) — which outlets have written about you?
10. Analyst / consultant coverage (Gartner / Forrester / IDC / industry-specific)?

## Section E — Operations + team

1. Total headcount (FTE including seasonal / contractors)? — **Internal**
2. Reporting structure (org chart summary)?
3. Names + roles of direct reports to the CEO / project lead?
4. External resources used regularly (agencies, freelancers, consultants, law firm, accountant)?
5. Key suppliers / vendors — top 5 by spend? — **Internal**
6. Any single-source dependencies that could break operations?
7. Office / facility footprint (HQ, retail, warehouse, plant, tasting room, showroom)?
8. Working-capital cycle length (invoice → cash)?
9. Insurance coverage (product liability, professional indemnity, cyber, D&O)?
10. IP portfolio (trademarks, patents, copyrights, trade secrets)?

## Section F — Finance

**Note:** much of this is internal-only. Distinguish public records (registeruz.sk, Companies House, SEC filings) from confidential management accounts.

1. Last fiscal year revenue? — **Internal / public if filed**
2. Gross margin % overall? — **Internal**
3. EBITDA / net margin %? — **Internal**
4. Cash runway / balance sheet cash? — **Confidential**
5. Debt position — senior debt, revolving credit, related-party loans? — **Confidential**
6. 12-month revenue target?
7. 12-month profitability target?
8. Top 3 cost line-items as % of revenue? — **Internal**
9. Pricing flexibility — can you raise prices? by how much?
10. Capex plans next 12 months?
11. Fundraising plans (equity / debt / grant)?
12. Accounting software + fiscal year-end date?
13. Audit / statutory filing cadence?

## Section G — Marketing + sales

1. What channels drive traffic / leads / sales today? (rank by volume + by quality)
2. Website CMS + URL?
3. Core analytics stack (GA4, Mixpanel, Amplitude, Segment, other)?
4. Ad platforms active (Google, Meta, LinkedIn, TikTok, trade publications)?
5. Email platform + list size?
6. Social platforms + follower counts?
7. CRM in use (if any)?
8. Content cadence (blogs/month, emails/month, social posts/week)?
9. Current CAC + LTV (even rough) — **Internal**
10. Conversion rates at key stages of the funnel — **Internal**
11. Partnerships / affiliate / referral programs active?
12. Press / PR activity + agency if retained?
13. Trade show / event calendar for next 12 months?

## Section H — Tech, tools, data

1. Core operational systems (ERP, POS, inventory, booking, LIMS, PMS, EMR, CAD, etc.)?
2. **File storage backend for this project — CRITICAL DECISION.** Pick one primary cloud:
   - **Google Drive / Google Workspace** — default for tech startups, many creative agencies, projects already on Google (Gmail, Calendar, Docs)
   - **OneDrive / SharePoint / Microsoft 365** — default for most SK/EU SMEs, family offices, regulated industries (finance, medical, legal), anyone using Outlook + Office
   - **Dropbox** — legacy choice; less tightly integrated with either Notion or Microsoft/Google productivity suites
   - **Local only** — for solo operators, very early-stage, paranoid-private. Later backfill with a cloud when the team grows.
   - **Hybrid (rare, flag it)** — some teams split: OneDrive for finance/legal, Google Drive for creative. Notion can embed both but this adds onboarding complexity.

   This choice drives (a) how reports + data files are embedded into Notion during Phase 5, (b) the shared-drive folder structure set up in Phase 3, (c) which cloud integration tiers are recommended. Document the answer in `CLAUDE.md` so every future session uses the same backend.
3. Source of truth for customer data?
4. Source of truth for product / inventory data?
5. Source of truth for financial data?
6. Existing data warehouse or BI stack?
7. Any automation tools in use (Zapier, Make, n8n, Airtable, retool)?
8. API access + documentation for each critical tool?
9. Authentication model (SSO / password manager / shared accounts — risk flag)?
10. Technical debt list — anything mentioned as "we need to fix that someday"?

## Section I — Legal, governance, compliance

1. Board structure (formal or informal)?
2. Who signs off on decisions above what €/$ threshold?
3. Who has authority to commit the company to contracts?
4. Specific compliance regimes (GDPR, HIPAA, SOC2, ISO, HACCP, FDA, MDR, MiFID, other)?
5. Last compliance audit date + outcome?
6. Known compliance gaps or open findings?
7. Pending legal matters (disputes, litigation, regulatory)?
8. Privacy policy / terms of service / returns policy — last-updated date?
9. Cookies consent / banner compliance per local law?
10. Specific content restrictions (e.g. alcohol = age gate, medical = no health claims, finance = no advice claims, food = allergen + labeling)?

## Section J — Goals + KPIs

1. Top 3 priorities for next 3 months?
2. Top 3 priorities for next 12 months?
3. Single biggest constraint right now (time / capital / talent / demand / supply / regulation)?
4. One specific thing that would 10× growth if solved?
5. One specific thing that would kill the business if it went wrong?
6. Key metrics you watch daily / weekly / monthly?
7. Lagging vs leading indicators — which do you trust?
8. Success criteria for this project in 12 months?

## Section K — Ways of working

1. Preferred communication channel (email / Slack / WhatsApp / in-person)?
2. Response-time expectation for the CEO (same day / next day / weekly)?
3. Working hours + time zone?
4. Days of week / blocks reserved for deep work?
5. Days reserved for meetings / external?
6. Tolerance for interruptions during a working block?
7. Preferred work-review cadence (daily / weekly / monthly)?
8. Travel schedule next 30 / 60 / 90 days — affects velocity planning?

## Section L — Agent / LLM adoption readiness

1. Comfort level with AI agents taking autonomous actions (1-5)?
2. Any actions you specifically reserve for yourself (even trivial ones)?
3. Any data specifically off-limits to LLMs (confidentiality, regulatory)?
4. Cloud vs local preference for sensitive processing?
5. Team member comfort level with AI-assisted workflows?
6. Willing to trial: autonomous research / content drafting / Shopify writes / email sends / social posts?
7. Preferred review cadence for auto-generated content (every piece / weekly batch / sample-audit)?

---

## After the interview

Claude produces:
- **Summary document** at `context/intake/[YYYY-MM-DD] - intake interview.md` with all answers + public/internal tags
- **Gap list** — every [GAP] tagged in the interview becomes either a research ticket (can be discovered) or a clarification ticket (needs CEO to dig)
- **Initial tag cloud** — extracted keywords that inform wiki structure + ticket categorization
- **First-draft CLAUDE.md skeleton** ready for Phase 3

CEO reviews the summary document before Phase 2 begins.
