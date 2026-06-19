---
name: draft
description: Trigger this when the user says "/draft", "draft this", "draft a blog", "draft email", "draft captions", "draft pitch", "draft one-pager", or wants a finished customer-facing or partner-facing deliverable produced from a tracker ticket or an ad-hoc brief. Distinct from /research — research investigates, draft creates the finished artifact. Outputs go through the project's marketing governance gate before publishing.
---

# Draft — produce finished deliverables

Turn a tracker ticket (or ad-hoc brief) into a polished, review-ready customer-facing deliverable. Covers blog posts, emails, social captions, landing pages, pitch decks, one-pagers, product sheets, outreach emails, press releases, and product descriptions.

Distinct from `/research`:
- `/research` = investigate, produce findings + recommendations
- `/draft` = create the finished artifact ready for review + approval

Part of the **operator loop** skill family: `/research` → `/questions` → `/review` → `/tidy` → `/draft`. See `~/.claude/skills/OPERATOR-LOOP.md`.

## Run modes

- **`/draft`** (default) — scan for draft-ready tickets, present plan, fast-confirm dispatch
- **`/draft dry`** — show plan, no dispatch
- **`/draft auto`** — skip confirmation (use sparingly)
- **`/draft single [TICKET-ID]`** — one specific ticket
- **`/draft [brief-description]`** — ad-hoc draft without a ticket (creates ticket retroactively)
- **`/draft type:[blog|email|social|landing|pitch|one-pager|outreach|press|product-desc|wiki]`** — filter by deliverable type
- **`/draft type:[...] TICKET-ID`** — coerce type if ticket tagging unclear
- **`/draft language:[sk|en|bilingual]`** — override default bilingual behavior
- **`/draft resume`** — pick up from interrupted state

## Prerequisites

### Config discovery (standard pattern)

1. `~/.claude/skills/draft/SKILL.md` — this file
2. `.claude/draft.md` — project override
3. `~/.claude/projects/-[escaped]/memory/MEMORY.md`

### Required project config

`.claude/draft.md` must declare:

- **Task tracker** — Notion DB ID / tracker path / GitHub Issues
- **Draft-candidate status** — what status means "ready to draft" (e.g. `Ready to draft`, `Needs draft`, `Not started` + draft-type tag)
- **Deliverable-type mapping** — how the project classifies tickets into types (by category, keyword, or explicit field)
- **Bilingual behavior** — default language(s), locales to produce
- **Writing-rule files** — path to project's CLAUDE.md, avoid-ai-writing skill override, banned-word lists, terminology rules
- **Governance gate** — what must happen BEFORE publishing (per CLAUDE.md marketing content approval rule)
- **Asset library locations** — where images, PDFs, brand assets live (so drafts reference correct paths)
- **Ownership classification** — the standardized 5-item closed list

## Deliverable types

### Blog post (`type:blog`)
SEO-optimized article, 600-1800 words. Structure: title / meta / lead / H2 sections / closing CTA / internal links.

### Email (`type:email`)
Transactional or marketing. Structure: subject (<50 chars) / preheader (90 chars) / body (300-600 words) / clear CTA. Plain-text compatible.

### Social caption (`type:social`)
Platform-aware. FB native multilingual SK+EN; IG SK+EN separated; X (SK only operational, SK+EN thread for campaigns). No hashtags per CLAUDE.md. Platform-specific character limits.

### Landing page (`type:landing`)
Hero + value prop + features + social proof + CTA. Copy-first (no HTML unless requested).

### Pitch deck (`type:pitch`)
B2B slide-by-slide brief. 12-18 slides typical. Structure: cover / problem / solution / product / proof / ask / appendix. Output as markdown structured by slide; HTML if requested.

### One-pager / product sheet (`type:one-pager`)
Single printable page. Structure: hero / key value props / specs / pricing / contact. Typically for trade / partner distribution.

### Outreach email (`type:outreach`)
Cold or warm B2B. Subject / opener / value hook / soft CTA / signature. Per project: follow-up sequence structure.

### Press release (`type:press`)
Traditional press format: headline / dateline / lead / body / boilerplate / contact.

### Product description (`type:product-desc`)
E-commerce product page copy. SEO-friendly title + meta + body. Under 300 words typical. Platform: Shopify-ready HTML if requested, else plain.

### Wikipedia article (`type:wiki`)
Wikitext format. Neutral encyclopedic tone. Every factual claim sourced. Infobox + sections per Wikipedia convention. (Governance note: WP:COI rules apply — outputs typically go to an editor, not self-published.)

## Run order

### Step 1 — Inventory

Scan tracker for draft-ready tickets:
- Status = draft-ready status value (per project config)
- Owner includes `Claude` or `Claude (post-approval)` or `Content Lead`
- No `Deliverable URL` set, OR set but user tagged it "revise"
- Title / description implies draft work (keywords: *draft, write, compose, create copy, captions, post, blog, email, one-pager, pitch, outreach*)

Deduplicate: if a published deliverable already exists for the ticket, skip unless user explicitly asks for revision.

### Step 2 — Type classification per ticket

For each candidate, classify deliverable type:
1. Explicit field if tracker has `Type` property
2. Keyword match in title/description (blog → `blog`, email → `email`, captions → `social`, pitch deck → `pitch`, etc.)
3. Category + context inference (MKT + "launch" → `social` + `email` + `blog` likely all needed)
4. If ambiguous: mark for user clarification

### Step 3 — Plan (fast-confirm)

```
Draft sweep — [N] candidates, [B] batches, ~[T] min

Batch 1 (concurrent, [duration]):
 1. [TICKET-ID] [type:blog]      "Vinalies Gold blog post SK+EN"
 2. [TICKET-ID] [type:email]     "European R&R launch email SK+EN"
 3. [TICKET-ID] [type:social]    "SPAR AT product announcement captions"
Batch 2 (concurrent):
 4. [TICKET-ID] [type:one-pager] "Frankovka 2025 product sheet"
 5. ...

Ambiguous (needs clarification):
 X. [TICKET-ID] "..." — unclear if blog or landing page

Reply: "go" / "go 1,3" / "skip 4" / "type 5 email" / "cancel"
```

### Step 4 — Dispatch

Spawn background Agent per ticket using the filled type-template from `~/.claude/skills/draft/templates/[type].md` (or project override at `.claude/draft-templates/[type].md`).

Each agent gets:
- Ticket context
- Deliverable type + template
- Target audience
- Must-include facts (from ticket)
- **Public-content rule** (HARD — every draft is customer-facing)
- **Writing rules** (project CLAUDE.md + `avoid-ai-writing` skill)
- Brand rules (banned words, terminology, legal form constraints — e.g. ACME: never "family winery", no hectares)
- Bilingual requirement if applicable
- Output path
- Length target
- Assets available (paths to brand assets, campaign images, etc.)

### Step 5 — Track + quality-gate each completion

For each agent completion:

1. **Public-content grep** — scan against banned patterns from project's `feedback_public_content_only.md` (or equivalent). Matches block auto-file.
2. **AI-writing grep** — run project's banned AI-word list. Matches flag but don't block.
3. **Length check** — within target range for deliverable type.
4. **Bilingual completeness** — if project requires SK+EN, both must exist; neither may contain untranslated placeholder text.
5. **Brand rule check** — banned terms (e.g. "family winery" for ACME), required terminology (e.g. "víno s prívlastkom neskorý zber" not just "neskorý zber").
6. **Asset references valid** — any referenced images/PDFs/links must resolve to existing files.

### Step 6 — File + governance gate

1. **Store draft** — `output/drafts/[YYMMDD] - [TICKET-ID] - [slug]/`. Subfolders per language if bilingual. Include `README.md` with rationale, target audience, intended channel, publish date.
2. **Update source ticket** — status → `Draft ready`, set `Deliverable URL` to folder, owner → CEO for review, due +7 days (faster than research reviews because drafts go stale).
3. **Governance-gated** — do NOT auto-publish. Even if the ticket owner is `Claude (post-approval)`, drafts must pass through `/review` session. Per CLAUDE.md marketing governance rule: all campaigns, ad copy, emails, social, landing pages, blog posts require prior board approval.
4. **Create review ticket** — `Review draft: [title]`, owner CEO, due +3 days (drafts need faster review than research). Body must be **self-contained** per `feedback_notion_task_self_contained`: include the draft path, the deliverable text inline (or a complete summary if too long for Notion limits), the channel + intended publish date, the must-include / must-not facts that constrained the draft, and any open questions for the reviewer. **Link** via "Depends on" to the source draft ticket and any sibling review tickets in the same campaign. Link via the Campaigns dual relation (named "Campaign" or "Linked Tasks" — check the Tasks DB schema) to the parent campaign.
5. **Append to changelog** — one line in `context/wiki/log.md`: `## [YYYY-MM-DD] draft | [TICKET-ID] [type] [title] → [path]`

### Step 7 — Summary

```
Draft sweep complete — [timestamp]

Completed:           [N] drafts ([breakdown by type])
Files produced:      [N] (paths)
Bilingual pairs:     [N] SK + [N] EN
Pending CEO review:  [N] drafts, due [+3 days]
Quality flags:       [N] (each listed with flag type)
Blocked:             [N] (failed public-content check; see `output/drafts/_blocked/`)

Top 3 drafts ready for review:
 1. [title] — [channel] — [characters/words] — [path]
 2. ...
 3. ...

Suggested next: /review to approve or edit
```

---

## Agent prompt template (stored at `~/.claude/skills/draft/templates/_base.md`)

Slots filled per dispatch:

```
{{TICKET_ID}}          — ticket reference
{{TICKET_TITLE}}       — verbatim
{{TICKET_DESCRIPTION}} — verbatim
{{TYPE}}               — blog / email / social / etc.
{{TYPE_TEMPLATE}}      — contents of type-specific template
{{AUDIENCE}}           — who reads this (retail customer / chain buyer / press / influencer / partner / internal team)
{{CHANNEL}}            — where it will ship (Shopify blog, Klaviyo, FB, IG, X, email, print)
{{PUBLISH_DATE}}       — intended publish date (drives urgency framing)
{{MUST_INCLUDE}}       — specific facts, offers, dates, links that MUST appear
{{MUST_NOT}}           — banned facts per public-content rule + per brand rules
{{WRITING_RULES}}      — paste of project CLAUDE.md writing rules
{{PUBLIC_CONTENT_RULE}} — paste of project's feedback_public_content_only.md
{{BANNED_WORDS}}       — AI-ism list + project brand-banned terms
{{TERMINOLOGY}}        — required terminology (e.g. ACME "kyseliny" not "kyselinky")
{{LENGTH_TARGET}}      — word count range
{{LANGUAGES}}          — [sk, en] or [sk] or [en]
{{ASSETS}}             — paths to available images / brand assets / past-campaign reference
{{OUTPUT_PATH}}        — where to write the finished draft
```

---

## Edge cases

- **No draft-ready tickets** — "Nothing queued for drafting." End.
- **Type unclassifiable** — flag, ask user `type:X TICKET-ID`, do NOT guess.
- **Must-include fact is internal (e.g. revenue figure)** — reject; drafts are customer-facing, can't cite internal-only data.
- **Bilingual required but SK version fails avoid-ai-writing check** — still file both, flag the SK for revision via `/review`.
- **Asset referenced doesn't exist at path** — flag for campaign team, file draft with placeholder marker.
- **Draft length way off target** — don't auto-truncate; flag for user.
- **Ad-hoc `/draft [brief]` without ticket** — create retroactive ticket (owner = Claude, category best-guess, source = "ad-hoc draft"), then run.
- **Request covers multiple types** (e.g. "draft the launch" = blog + email + social) — split into multiple tickets + deliverables, one agent per type.
- **Revision requested on existing draft** — read old draft, produce v2 in same folder (`v1.md`, `v2.md`, etc.), do NOT overwrite.

## Integration with other operator-loop skills

- **`/research`** — findings often trigger a draft (research says "do X", draft creates X)
- **`/review`** — ALL drafts pass through here before publishing (governance gate)
- **`/questions`** — answer to "which version?" can dispatch a revision via `/draft`
- **`/tidy`** — counts draft velocity separately from research velocity; helps spot bottlenecks
- **`avoid-ai-writing`** — applied at quality-gate stage
- **`wrap-session`** — captures draft session state

## Governance: hard rule

**No draft auto-publishes.** Per universal pattern, the marketing/governance approval step is always mediated by the project's `/review` or CEO explicit approval. `/draft` produces finished, review-ready artifacts; the ship/no-ship decision is a separate approval.

This applies even when the source ticket owner is `Claude (post-approval)`. The post-approval owner simply means Claude executes the publish step AFTER approval. The draft itself still needs explicit green-light in `/review`.

## Related

- `~/.claude/skills/OPERATOR-LOOP.md` — family overview
- `~/.claude/skills/research/` — finds the facts this skill turns into copy
- `~/.claude/skills/review/` — approves/rejects drafts
- `avoid-ai-writing` skill — quality filter
- Project `CLAUDE.md` — governance + brand rules
