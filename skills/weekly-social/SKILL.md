---
name: weekly-social
description: When the user wants to batch-prepare social media posts for the week ahead. Also use when the user mentions "weekly social batch," "Sunday social prep," "Monday morning posts," "plan the week," "social calendar walk-through," "batch the week ahead," "schedule next week's posts," or "go through the posting calendar with me." Walks slot-by-slot through the upcoming 7 days of the project's posting calendar, drafts copy, proposes assets, audits last-week catch-up debt, and outputs a single batch document ready for board review + scheduling. Cross-project — requires a project override at `.claude/weekly-social.md` in the project root.
---

# Weekly social media batch

A walkthrough of one week of social posts in a single session. Drafts every caption, queues every asset, surfaces approval gates, hands off a batch doc the user (or board) approves before scheduling.

## When to use

- **Sunday evening** or **Monday morning** — full week-ahead batch (the intended cadence)
- **Mid-week if behind** — catch-up plan + remaining slots
- **Before a campaign/holiday week** — prep all moments in advance
- **After a missed week** — triage what to backfill vs accept-loss + re-plan forward

## Step 0 — Bootstrap project override

Read `.claude/weekly-social.md` in the project root. It defines:
- Project name + voice rules location
- Active social channels + handles
- Posting playbook (slots, peak times)
- Posting calendar file path
- Asset folder convention + generation policy
- Brand do/don't rules
- Approval gates
- Bilingual / multilingual conventions
- Posting mechanism (manual via Business Suite, programmatic via MCPs, etc)

**If override missing:** stop and offer to create one using the template at the bottom of this skill. Don't proceed without it.

## Step 1 — Anchor the week

Compute the window:
- **Today is Sun or Mon** → window = next 7 days (Mon-Sun, or Sun-Sat depending on locale)
- **Mid-week (Tue-Sat)** → window = today through next Sunday + flag the current-week-remaining slots as catch-up territory

State the window explicitly at the start: "Planning posts from [date] through [date] ([N] active channel-slots per playbook)."

## Step 2 — Load the posting calendar

Read the calendar file specified in the override. Extract every dated slot in the window. Each slot has: date, day, time, platform, planned topic, asset reference (if any).

If the calendar file doesn't exist or is empty for the window, ask the user:
- Should we extrapolate from the playbook default cadence (Tue/Wed/Fri/Sat etc)?
- Or should we draft a fresh calendar for this window first?

## Step 3 — Audit prior week (catch-up debt)

For each active channel, pull what actually posted in the **last 7 days** vs what was planned:

- **Facebook / Instagram:** `mcp__meta-insights__get_fb_page_posts` and `mcp__meta-insights__get_ig_media_insights` for the date range
- **X / Twitter:** manual or via X MCP if available
- **GBP:** manual (no MCP)

Output a compact audit:

```
Channel | Planned | Posted | Missed slots (dates)
FB      | X       | Y      | [list]
IG      | X       | Y      | [list]
GBP     | X       | Y      | [list]
```

If catch-up debt exists, ask the user explicitly:
1. **Backfill which missed slots?** (some misses are recoverable, some are gone — event-driven posts after the event = gone)
2. **Accept loss** on the rest and focus on the week ahead?

Default recommendation: backfill medal/evergreen content; accept-loss on event-driven (Mother's Day after Mother's Day, etc).

## Step 4 — Walk each upcoming slot

For each slot in the window, do this in order:

1. **Confirm the topic.** Read the calendar entry. Cross-check with reality:
   - Is the topic still relevant?
   - **Are any events/coordination dependencies real?** A calendar mention of an event ≠ confirmation the project is participating. Ask before building build-up posts around an event.
   - Has anything changed since the calendar was built (new awards, product launches, news)?

2. **Draft caption(s)** per platform convention from the override.
   - Single multilingual post (one caption SK+EN with divider) — used by some FB pages
   - Per-platform separate captions (when FB caption flopped on IG, write a different one)
   - Stories, Reels: format differs (Stories = short + visual; Reels = hook + voice translation)

3. **Asset.** One of:
   - **Library reuse:** point to existing file path. Note: include the path in the batch doc.
   - **AI gen via Higgsfield:** write the prompt + model + mode. Prefer `higgsfield-product-photoshoot` for branded product visuals, `higgsfield-generate` for everything else, `higgsfield-marketplace-cards` for listing images.
   - **Shoot needed:** flag as a blocker. Propose a fallback (existing photo, Higgsfield gen) so the slot isn't lost.

4. **Posting time + format.** From the playbook. Note both day-of-week and exact time. If the playbook says "audience peak 15:00-20:00," default to that range, not lunch slots.

## Step 5 — Asset queue

Group all assets across the week into three lists:
- **Reuse from library** (paths) — fastest, already approved on-brand
- **Higgsfield gen queue** — ordered prompts to run sequentially
- **Shoot needed** — items where no fallback works; flag as a blocker

If Higgsfield items exist and the user has the CLI installed (per memory or `which higgsfield`), offer to run the queue inline before scheduling.

## Step 6 — Output the batch document

Save to: `output/campaigns/[YYMMDD] - week of [date] social batch.md`

Structure:
```markdown
# Social batch — week of [date]

## Catch-up debt audit
[3-row table from Step 3]
[Decisions on backfill vs accept-loss]

## Slot-by-slot plan
### [Day, Date]
- [Platform] [Time]
- Topic:
- Caption (SK):
- Caption (EN):
- Asset: [path or gen prompt]

(repeat for each slot)

## Asset queue
- Reuse: [paths]
- Generate: [Higgsfield prompts]
- Shoot needed: [blockers]

## Open questions for approval
- [list anything blocking publish]

## Scheduling plan
- [manual via Business Suite / programmatic / hybrid]
```

## Step 7 — Approval + schedule

Per the override:
- **Manual mode (default for high-control projects):** present the batch doc, wait for board approval, then user schedules in Meta Business Suite (or equivalent) directly.
- **Programmatic mode:** queue posts via platform MCPs (Meta, X, etc) AFTER per-post approval per the project's approval policy.

Never auto-publish unless the override explicitly permits.

## Hard rules

- **Approval gates first.** Most projects require board approval before publish. This skill drafts and queues; it does not auto-publish.
- **Verify event participation before drafting build-up content.** Calendar entry ≠ confirmation. Ask once before drafting 4 dependent posts.
- **Match the project voice.** Read the brand rules in the override. If the rules say "no hashtags," there are no hashtags in the output, period.
- **Bilingual = different captions per platform when needed.** If a previous identical-caption FB+IG attempt flopped on one platform, write a different version this time. Note the change in the batch doc.
- **Asset gen policy.** Use Higgsfield (or other gen AI) only if the override allows. Some projects mandate human photography only.
- **Mid-week mode.** If triggered mid-week with catch-up debt, prioritise: (1) today's slot, (2) tomorrow's slot, (3) easy-win backfills, (4) the rest. Don't try to backfill distant misses unless evergreen.
- **Do not delete the previous batch doc.** Each week gets its own dated file. Past batches are reference + accountability.

## Project override template

Save this as `.claude/weekly-social.md` in a new project root:

```markdown
---
project_name: [project name]
voice_rules: [path to brand/voice doc, e.g. CLAUDE.md in the project root]
---

## Active channels

- **Facebook:** [page name] (page id [id], fans [N]). Tools: `mcp__meta-insights__get_fb_page_posts`, `get_fb_page_insights`.
- **Instagram:** @handle (ig account id [id], followers [N]). Tools: `mcp__meta-insights__get_ig_media_insights`, `get_ig_account_insights`.
- **GBP:** [location]. No MCP, manual check.
- **X / Twitter:** @handle. [Opportunistic / regular] use.
- **Other platforms:** [list]

## Posting playbook

Calendar path: [path to current month's posting calendar.md]
Source playbook: [path to playbook doc, e.g. ACME-OPS-026]

Slots:
| Day | Slots |
|---|---|
| Mon | [list times + channels] |
| Tue | [list] |
| ... | ... |

Audience peak: [time range, e.g. 15:00–20:00 CET]

## Voice + brand rules

Source: [voice rules doc]
Highlights (the must-remember):
- [no-go phrases]
- [punctuation rules, e.g. no em-dashes]
- [hashtag policy]
- [bilingual conventions per platform]
- [forbidden terms specific to the brand]
- [accurate terminology requirements]

## Asset policy

- Library path: [path]
- Higgsfield: [ENABLED/DISABLED]. Preferred sub-skills: [list]
- Shoot policy: [conditions for needing a shoot]
- Approved reuse hero assets: [list with paths + relevance]

## Approval gates

- Board approval before publish: [YES/NO + who]
- Per-platform write approval: [policy]
- Scheduling method: [manual via Business Suite / via MCP / hybrid]

## Catch-up triage rules

- Keep: [categories of post that are still worth backfilling]
- Drop: [categories of post that are time-dead and not worth backfilling]
- Approve fresh: [categories that should be drafted from scratch each week]

## Known sub-skills available

- [list any skills this project uses for asset gen, voice audit, etc]

## Output destination

`output/campaigns/[YYMMDD] - week of [date] social batch.md`
```
