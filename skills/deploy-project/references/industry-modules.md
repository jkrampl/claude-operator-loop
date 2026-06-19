# Industry modules — per-industry deltas

Each industry has its own conventions, regulations, typical stack, content rules, and recurring cadences. Use these modules during Phase 1 intake (which questions matter more), Phase 5 (which MCPs to connect), Phase 8 (what governance to codify), and Phase 10 (what to put on recurring cadences).

## Tech / SaaS / Software

**Typical stack:** GitHub, Linear/Jira, Vercel/Netlify, Stripe, Intercom/Crisp, Mixpanel/Amplitude, Customer.io, Segment, PostHog.

**Core MCPs:** GitHub, Notion, Linear, Stripe, Slack, Sentry.

**Governance gates:**
- Production deploys → staged rollout + feature flag
- Customer data export / delete → CEO or DPO approval
- Pricing changes → board approval
- Open-source release → license review

**Writing rules:**
- Plain technical English, no corporate jargon
- No made-up benchmarks, cite sources
- Release notes follow keepachangelog convention

**Recurring cadence adds:**
- Weekly: GitHub PR hygiene, production monitoring
- Monthly: security + dependency audit, uptime review
- Quarterly: SOC2/ISO if applicable, pricing review
- Annual: security pen-test, terms of service review

**Public-content notes:** no undisclosed beta features, no customer names without permission, no roadmap commitments beyond 90 days.

---

## Medical / healthcare / clinic

**Typical stack:** EMR (e.g. Doctolib, Medesk, Epic-lite), patient portal, scheduling, lab integrations, billing.

**Core MCPs:** Notion (non-PHI only), scheduling, accounting. **NO PHI through general MCPs.**

**Governance gates:**
- Any patient-facing content → medical advisory board (if exists) or qualified clinician sign-off
- Any health claim → require peer-reviewed source cited
- Privacy/data handling → DPO-equivalent + regulatory (HIPAA / GDPR Art. 9 / local health data law)
- Marketing claims → compliance with advertising standards for healthcare

**Writing rules (HARD):**
- No unverified health claims (resveratrol, cure, treats, reverses, boosts)
- Disclaimers required on any outcome-suggestive content
- Regulatory-aware terminology (e.g. EU MDR for devices)
- No before/after imagery without documented patient consent

**Recurring cadence adds:**
- Weekly: patient feedback / complaint review
- Monthly: clinical outcomes tracking, staff training compliance
- Quarterly: regulatory updates check (FDA / EMA / local), insurance-panel compliance
- Annual: HIPAA or local privacy audit, professional indemnity renewal, licensing renewal

**Public-content notes:** never include patient identifiers (names, faces, dates of visit), no internal case volumes, no revenue per procedure.

---

## Interior design / architecture / studio

**Typical stack:** project management (Notion / Monday / Asana), CAD (AutoCAD, SketchUp, Revit), rendering (V-Ray, Lumion), asset libraries (Houzz Pro, Designer Pages, Studio Designer), accounting, client portal.

**Core MCPs:** Notion, Google Drive, accounting software.

**Governance gates:**
- Client proposals > $X → CEO/partner sign-off
- Vendor specification changes → project lead approval
- Portfolio publishing (new project images) → client's permission + brand-use agreement
- Press features → PR alignment

**Writing rules:**
- No stock tourism-brochure prose ("stunning", "breathtaking")
- Cite materials, finishes, suppliers accurately (trade partnerships may require credit)
- Client names used only with explicit permission
- Respect non-disclosure on high-profile residential

**Recurring cadence adds:**
- Weekly: project status (one line per active project)
- Monthly: invoice aging, supplier payment cycle
- Quarterly: portfolio refresh + case study publication
- Annual: trade show participation (Salone del Mobile, Maison&Objet, ICFF), awards submissions

**Public-content notes:** no budget figures per project unless cleared, no client personal addresses, no "after" photos before client approves.

---

## Food & beverage / restaurant / winery / brewery

**Typical stack:** POS (Toast, Square, Lightspeed), reservation (OpenTable, Resy, Tock), inventory (MarketMan, BlueCart, Shopify), accounting, delivery (Uber Eats, Wolt, Bolt), wholesale (if B2B).

**Core MCPs:** Shopify (if DTC), Google Analytics, Notion, Google Business.

**Governance gates:**
- Allergen label changes → safety officer + packaging compliance
- Recipe changes → chef/winemaker + regulatory labeling
- Supplier substitutions → spec compliance (organic, BIO, halal, kosher if claimed)
- Wholesale pricing → board
- Retail chain pitches → board + legal
- Alcohol-specific: age-gate compliance, trade advertising standards

**Writing rules:**
- Allergens listed per local law (EU Reg 1169/2011 or local equivalent)
- No health claims beyond approved list (EU Reg 432/2012)
- Alcohol content stated in local convention (%ABV)
- Origin claims (PDO / PGI / DOC / AVA / AOC) verified
- Tasting notes in consistent structure per SKU

**Recurring cadence adds:**
- Weekly: inventory count, waste tracking
- Monthly: supplier audit, menu engineering (food cost %), staff training
- Quarterly: food safety / HACCP review, licence compliance
- Annual: vintage review (wine), audit renewals, certification renewals

**Public-content notes:** no actual production volumes, no wholesale margins, no chain-pricing figures, no internal aggregate awards counts beyond public website.

---

## Finance / investment / wealth / fund

**Typical stack:** portfolio management (Addepar, Eton), CRM (Wealthbox, Redtail), compliance (Smarsh, Global Relay), trading (Interactive Brokers, Bloomberg), accounting, custodian integration.

**Core MCPs:** Notion (non-personal-data), calendar, accounting. **NO client financial data through general MCPs without compliance review.**

**Governance gates:**
- Any investment recommendation → registered investment advisor sign-off
- Client communications → compliance pre-approval
- Performance reporting → GIPS-compliant or equivalent + disclaimers
- Advertising / marketing → SEC / FCA / CSSF / local regulator rules
- Crypto / digital assets → additional KYC/AML

**Writing rules (HARD):**
- No "investment advice" unless licensed
- No "guaranteed returns" ever
- No specific stock picks without full disclosure of position
- Past performance disclaimer on any historical return figure
- No back-testing without disclosure of methodology
- Client testimonials often prohibited or heavily restricted

**Recurring cadence adds:**
- Weekly: portfolio review, flag unusual activity
- Monthly: client reporting, compliance register
- Quarterly: rebalancing, client review meetings, compliance filing (Form PF etc.)
- Annual: audit, Form ADV update, tax package prep

**Public-content notes:** no AUM figures if material non-public, no client names unless disclosure allowed, no specific holdings unless quarterly filing, no projections without full assumptions.

---

## Family office

**Typical stack:** accounting (QuickBooks Enterprise / Sage Intacct / Addepar), document vault (Box / Vaultime), portfolio reporting, estate planning tools, private-jet / yacht management if applicable.

**Core MCPs:** Notion (non-financial), Google Drive (personal files), accounting read-only. **Highly restricted MCP scope.**

**Governance gates:**
- Any asset transaction → principal sign-off + legal review
- Any communication mentioning family members → principal approval
- Any external publication → explicit consent from every family member referenced
- Estate-related changes → estate attorney + principal

**Writing rules (HARD):**
- Extreme privacy: no names, no locations, no asset counts publicly
- Press-inquiry handling: default no-comment
- Internal-only documents mark with PRIVATE / CONFIDENTIAL watermark
- Social media footprint: typically minimal or zero

**Recurring cadence adds:**
- Weekly: cash management, upcoming obligations
- Monthly: consolidated balance sheet, portfolio mark-to-market
- Quarterly: entity-by-entity reporting, tax planning
- Annual: tax filings across jurisdictions, insurance renewals, trust reviews, estate plan review, LLC reports

**Public-content notes:** **the default for family office is NO public content.** Any exception requires explicit principal sign-off. Publicly discoverable info (deeds, filings) is acknowledged but not amplified.

---

## Manufacturing / industrial

**Typical stack:** ERP (SAP / NetSuite / Microsoft Dynamics / Odoo), MES, QMS, PLM, EAM, supplier portal, shipment tracking.

**Core MCPs:** Notion, ERP (if API available), shipping / logistics, analytics.

**Governance gates:**
- Supplier contract changes → procurement + legal + quality
- Process changes → quality manager + regulatory (ISO / FDA / CE / RoHS / REACH)
- Product spec changes → engineering change order (ECO) process
- Customer-facing product claims → quality + legal
- Safety data sheet changes → regulatory

**Writing rules:**
- Technical spec language, not marketing-hype
- Compliance-first labeling
- Cite standards (ISO 9001, IATF 16949, AS9100, etc.) only if currently certified
- No capacity claims beyond verified throughput

**Recurring cadence adds:**
- Weekly: OEE review, defect rate, safety incidents
- Monthly: supplier performance scorecard, inventory turns, customer complaints
- Quarterly: quality audit, management review (ISO 9001 clause 9.3)
- Annual: certification recertifications, insurance renewals, environmental compliance

**Public-content notes:** no customer names without permission, no pricing, no capacity utilization data, no quality defect rates externally, no supplier names (usually confidential).

---

## E-commerce / DTC consumer brand (generalized)

**Typical stack:** Shopify / WooCommerce / BigCommerce, Klaviyo / Mailchimp, Meta + Google ads, Google Analytics, Google Merchant Center, Shop Pay, returns platform, review platform.

**Core MCPs:** Shopify, GA4, Meta organic, Klaviyo, GSC.

**Governance gates:**
- Product changes → CEO per item
- Pricing / promotion → board above threshold
- Ad campaign creative → board
- Email campaigns → board
- Landing pages → board

**Writing rules:**
- No AI-isms, em-dashes, hollow intensifiers
- Product descriptions factual, benefits-led
- Shipping/returns policy linked on every page
- Brand voice consistent across channels

**Recurring cadence adds:**
- Weekly: revenue review, CAC/ROAS by channel, inventory alert
- Monthly: email performance, ad campaign optimization, review response
- Quarterly: conversion rate deep-dive, unit economics review, CRO tests
- Annual: full catalog refresh, tax compliance (VAT, sales tax nexus)

---

## Professional services / consulting / agency

**Typical stack:** CRM (HubSpot / Salesforce), project management (Asana / Monday), time-tracking (Harvest / Toggl), accounting, document management, client portal.

**Core MCPs:** Notion, Google Calendar, CRM, accounting.

**Governance gates:**
- Client proposals → partner sign-off above threshold
- Team assignments → resource manager
- Content publication (case studies) → client consent
- Pricing changes → board

**Writing rules:**
- Case studies must cite client + metrics with permission
- Thought leadership backed by data
- No claims about "transformation" without outcomes

**Recurring cadence adds:**
- Weekly: billable hours by consultant, project status
- Monthly: revenue recognition, pipeline review
- Quarterly: partner meeting, utilization review, pricing review
- Annual: client satisfaction survey, team development plans

---

## Universal overlays (apply to every industry)

**Always present across all industries:**
- Public-content rule (what can never go external)
- AI-writing rule (no AI-isms, em-dashes, hollow filler)
- Ticket-everything rule (every emergent task goes to Notion)
- Quarterly assessment anchor (Jan 1 / Apr 1 / Jul 1 / Oct 1)
- Monthly team sync anchor (first Monday of month)
- Session boundaries: wrap-session / resume-session always installed
- Operator-loop skills always deployed

**Always customize per project:**
- Legal entity identifiers in CLAUDE.md
- Currency + tax jurisdiction
- Ticket ID prefix (ACME / ACME / ALPHA / etc.)
- Category taxonomy per the industry
- Team roster
- Governance thresholds
