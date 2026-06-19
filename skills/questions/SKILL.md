---
name: questions
description: Trigger this when the user says "/questions", "generate questions", "walk questions", "questions for walk", "voice questions", or wants a list of open questions they can voice-record responses to while away from the machine. Also triggers on "/questions upload [transcript]" to ingest a voice-recorded transcript, match answers to question IDs, classify each response, and apply decisions across the tracker. Purpose is to let the user spend walking time answering decisions instead of sitting at a desk.
---

# Questions — walk-friendly Q&A flow

Convert open decisions across the project into a numbered, phone-readable question list for voice-recorded responses. Then ingest a transcript, parse responses, classify each, and apply decisions automatically.

Two modes, same skill:
- **Generate mode**: produce the question list
- **Upload mode**: process the transcript of voice responses

Part of the **operator loop** skill family: `/research` → `/questions` → `/review` → `/tidy`. See `~/.claude/skills/OPERATOR-LOOP.md`.

## Run modes

### Generate mode

- **`/questions`** (default) — scan for all open decisions, produce a numbered list
- **`/questions category:[list]`** — filter by category
- **`/questions source:[tickets|research|memory|all]`** — scope the scan
- **`/questions max:N`** — cap list length (default 25, good for a ~30-min walk)
- **`/questions urgent`** — only decisions with hard deadlines

### Upload mode

- **`/questions upload`** — then paste transcript in the same or next message
- **`/questions upload [transcript-text]`** — inline
- **`/questions upload file:[path]`** — read from file
- **`/questions upload dry`** — show what WOULD be applied without actually applying

## Prerequisites

### Config discovery (standard pattern)

1. `~/.claude/skills/questions/SKILL.md` — this file
2. `.claude/questions.md` — project override
3. `~/.claude/projects/-[escaped-project-path]/memory/MEMORY.md`

### Required project config

`.claude/questions.md` must declare:

- **Tracker source** — Notion DB ID, tracker path, or GitHub Issues
- **"Needs input" status value(s)** — e.g. "Awaiting CEO", "Needs input", "Blocked on decision"
- **Research deliverable scan paths** — e.g. `output/reports/` (scan for "Flags for CEO review" sections)
- **Memory file path** — e.g. `~/.claude/projects/-[escaped]/memory/MEMORY.md` (scan for "Still open" items)
- **Wiki decision markers** — regex/patterns for inline `[NEEDS DECISION]` markers, if used
- **Team member list** — for owner re-assignment on new tickets created by answers
- **Categories** — project's categorization taxonomy

---

## Generate mode — run order

### Step 1 — Scan all question sources

Collect open decisions from:

1. **Tracker tickets** — status ∈ {Needs input, Awaiting CEO, Awaiting decision, Blocked on CEO}, owner = CEO
2. **Research deliverables** — parse files in project's report folders for sections named:
   - `## Flags for CEO review`
   - `## Open questions`
   - `## Questions for [owner]`
   - `## Decisions needed`
   Extract each bullet as a potential question.
3. **Memory file** — "Still open" / "Parked for decision" / "Next session" sections
4. **Wiki inline markers** — scan wiki files for `[NEEDS DECISION]`, `[DECIDE]`, or similar patterns
5. **Session log** — last 7 days of session-log files for "deferred" decisions

### Step 2 — Deduplicate + rank

- Fuzzy-dedupe similar questions (80%+ title similarity → merge with links to all source IDs)
- Rank by:
  1. Hard deadline present
  2. Blocks other work (dependency count)
  3. Age (oldest first)
  4. Priority inferred from source ticket

### Step 3 — Reformat for voice response

Each question must be:
- **Self-contained** — no need to look up context while walking
- **Binary or list-answerable** — avoid open-ended "what do you think"; prefer "A or B or C?" / "yes or no" / "which number"
- **One sentence** — max 25 words
- **Answerable in <30 seconds** out loud

If a sourced item is too complex to reframe, include it as a "deep" question with a note: "Q-X [deep, answer later]: [question]"

### Step 4 — Group by category

Use project's category taxonomy. Example grouping:
- Marketing (SEO, MKT, content, campaigns)
- Operations (OPS, procurement, team)
- Finance (FIN, budget, pricing)
- Product (product decisions, roadmap)
- Legal (compliance, contracts, policies)
- Strategic (positioning, partnerships, long-term)

### Step 5 — Output

File: `output/questions/[YYMMDD] - Walk questions.md`

Format:

```markdown
# Walk Questions — [YYYY-MM-DD]

[N] questions across [K] categories. Estimated ~[T] min walking time at a conversational pace.

**How to answer:**
- Say "question [N]" before each answer so transcription can match it
- For yes/no: "yes", "no", "skip", "pass"
- For numbers: say the number clearly
- For open answers: explain, then say "next" when done
- For deferral: "park this" or "defer to [timeframe]"

---

## Marketing ([N] questions)

### Q-01
AWC Vienna 2026 — research recommends 9 samples total, 6 paid + 3 free. Budget 750 to 850 euros. Yes, no, or different number?

### Q-02
European Red and Rosé launch day — Monday April 20 or Tuesday April 21?

...

## Operations ([N] questions)

### Q-[NN]
...

---

## Deep questions (answer at desk, not on walk)

### Q-[NN] [deep]
[Longer question that needs context / data]

---

**Source references** (for post-walk filing):
- Q-01 → ACME-MKT-071 + research deliverable at [path]
- Q-02 → ACME-MKT-044 + campaign plan at [path]
...
```

Source references at the bottom are used later by upload mode to map answers to correct destinations.

### Step 6 — Output companion PDF (optional)

If user will read from phone, also generate a plain-text version easy on mobile:
- `output/questions/[YYMMDD] - Walk questions - mobile.txt`
- No markdown syntax, just numbered questions, larger spacing

### Step 7 — Summary to user

```
Generated [N] walk questions at [path]
Estimated walk time: ~[T] min
Source references: stored in file for upload-mode matching
[Mobile-friendly version also at: path]
```

---

## Upload mode — run order

### Step 1 — Accept transcript

Transcript may be:
- Inline text in the message
- File path (read it)
- Pasted with or without question numbering
- Paragraph-form or structured

### Step 2 — Load the question file

Find the most recent `output/questions/[YYMMDD] - Walk questions.md` file (or use explicit file if user specified). Load the question list with source references.

### Step 3 — Parse answers

For each question, find the matching answer in the transcript using:

1. **Explicit Q-ID anchors** — `Q-01`, `question 1`, `one dot`, `first question`, numeric markers
2. **Sequential order fallback** — if no explicit IDs, assume answers are in question order (warn the user)
3. **Fuzzy content matching** — if an answer references the question content ("yes to the AWC Vienna thing"), match on content similarity

If an answer cannot be confidently matched to a question, collect it as an "unmatched comment" to surface at the end.

### Step 4 — Classify each response

Each answer falls into one of these classes:

| Class | Signal | Action |
|-------|--------|--------|
| **Direct decision** | yes/no, specific number, specific name, "A" or "B" | Apply to source ticket: status change, field update, or approval |
| **New task/direction** | "we should also do X", "start working on Y" | Create new ticket with classified owner |
| **Clarification request** | "I need to know X first", "find out Y" | Update ticket: add dependency note, status → Blocked |
| **Deferral** | "park this", "next quarter", "defer to [date]" | Update ticket: status → Deferred, new due date |
| **Rant / context** | Long explanation, no clear instruction | Save to memory observations, do NOT modify ticket |
| **Out of scope** | Answer doesn't relate to question (mis-numbered) | Flag for user review, don't apply |

### Step 5 — Owner classification for new tickets

Any new ticket created from an answer MUST be assigned one of:
- `Claude` — autonomous
- `Claude (post-approval)` — needs CEO approval then Claude executes
- `CEO` — manual
- `External` — dev, designer, vendor
- `[Team member name]` — from project config team list

Classify based on task nature:
- Pure research / drafting → Claude
- Tool execution (Shopify write, GSC submit, etc.) → Claude (post-approval)
- UI click / human conversation / board decision → CEO
- Code / design / vendor work → External
- Specific-person operational task → team member name

### Step 6 — Apply in batch

For each parsed answer, execute its action:
- Tracker update (Notion MCP or equivalent)
- New ticket creation — body must be **self-contained** per `feedback_notion_task_self_contained`: include the answer text verbatim, the question that prompted it, the source ticket / report it relates to, the intended next step, and any explicit constraints from the user. **Link** via "Depends on" / "Project link" / the Campaigns dual relation (named "Campaign" or "Linked Tasks") to anything connected (the question source ticket; the parent campaign or project).
- Dependency note
- Deferral with new due date
- Memory append

Collect results: what succeeded, what failed, what needed clarification.

### Step 7 — Summary

```
Questions upload processed — [timestamp]

Questions answered:    [N] / [total]
Answers parsed:
  — [X] direct decisions applied
  — [Y] new tickets created (classified: [breakdown by owner])
  — [Z] clarifications requested
  — [W] deferrals
  — [V] memory observations saved
  — [U] unmatched (surfaced below)

Unmatched comments (placement needed):
[list of comments that didn't clearly match any question]

Questions skipped by user:
Q-[X], Q-[Y] — remain open for next /questions round

Follow-up suggested:
/review  — to act on pending outputs from clarifications
```

### Step 8 — Audit trail

Append session log to `context/sessions/[YYYY-MM-DD] - Questions upload.md`:
- Original transcript
- Parsed answers
- Applied actions + outcomes
- Unmatched comments
- Errors

## Edge cases

- **No open questions** — "No decisions pending. Clear walk!" End.
- **Transcript has fewer answers than questions** — apply what matches, leave rest open for next round.
- **Transcript has MORE answers than questions** — extra content goes to unmatched for review.
- **Ambiguous answer to a yes/no** (e.g. "well, kind of") — don't auto-apply, surface for clarification.
- **Answer contradicts earlier decision** — flag conflict, do NOT auto-apply.
- **No Q-IDs in transcript, sequence ambiguous** — warn user, offer to proceed with best-guess matching OR reject.
- **Transcript references new item not in question list** — add as unmatched new-ticket suggestion.
- **Mid-walk realization: "about Q-03, I change my mind to X"** — apply the update, override earlier answer if already parsed.

## Integration with other operator-loop skills

- **`/research`** — deliverables produced by research are scanned for "Flags for CEO review" sections
- **`/review`** — can surface pending clarifications created by `/questions upload`
- **`/tidy`** — runs after big upload batches to rebalance the tracker
- **`wrap-session`** — captures both the question file and the upload session log

## Related

- `~/.claude/skills/OPERATOR-LOOP.md` — family overview
- Project `CLAUDE.md` — constrains what can be applied automatically
