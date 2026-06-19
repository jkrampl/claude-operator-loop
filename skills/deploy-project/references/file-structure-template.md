# File structure template — canonical project layout

Every project deployed by this runbook uses this structure. Standardize hard so that skills (research/review/questions/tidy/draft/wrap-session/resume-session) find everything in predictable paths.

## Project root layout

```
[project-root]/
├── CLAUDE.md                            # governance + brand + writing rules (always here)
├── STRUCTURE.md                         # index of this structure (optional but recommended)
├── README.md                            # public-facing project overview (if shared)
├── .claude/                             # Claude Code config
│   ├── settings.local.json              # permission allowlist
│   ├── research.md                      # project override for /research skill
│   ├── review.md                        # /review override
│   ├── questions.md                     # /questions override
│   ├── tidy.md                          # /tidy override
│   ├── draft.md                         # /draft override
│   ├── wrap-session.md                  # wrap-session override
│   └── [other skill overrides as needed]
├── context/                             # reference material, synthesized knowledge
│   ├── wiki/                            # Karpathy-style wiki (synthesized knowledge)
│   │   ├── index.md                     # master catalog of all wiki pages
│   │   ├── log.md                       # chronological changelog
│   │   ├── company/                     # about, team, finance, registration, goals
│   │   ├── customers/                   # segments, ICP, data
│   │   ├── market/                      # competitors, industry, trends
│   │   ├── channels/                    # marketing channels per platform
│   │   └── operations/                  # integrations, recurring-register, SOPs
│   ├── tracker/                         # legacy tracker (before Notion); keep for reference
│   │   ├── BACKLOG.md                   # deprecated once Notion is the source of truth
│   │   └── DONE.md
│   ├── intake/                          # Phase 1 intake artifacts
│   │   └── [YYYY-MM-DD] - intake interview.md
│   ├── audit/                           # Phase 2 audit artifacts
│   │   └── [YYYY-MM-DD] - existing-state audit.md
│   ├── sessions/                        # session-level state files
│   │   ├── [YYYY-MM-DD] - research sweep state.md
│   │   ├── [YYYY-MM-DD] - Review session.md
│   │   ├── [YYYY-MM-DD] - Questions upload.md
│   │   └── [YYYY-MM-DD] - Tidy session.md
│   ├── templates/                       # branded report template, email template, etc.
│   └── scripts/                         # project-specific scripts, builders
├── output/                              # generated deliverables
│   ├── README.md                        # folder structure guide
│   ├── reports/                         # final branded reports (HTML, MD)
│   │   └── [YYMMDD] - [TICKET-ID] - [slug].[ext]
│   ├── drafts/                          # work-in-progress, pre-approval
│   │   └── [YYMMDD] - [slug]/
│   ├── campaigns/                       # named campaigns with full asset packages
│   │   └── [YYMMDD] - [campaign-name]/
│   │       ├── README.md
│   │       ├── assets/
│   │       ├── blog-sk.md / blog-en.md
│   │       └── captions/
│   ├── digests/                         # recurring digest outputs (e.g. weekly market digest)
│   │   └── [source-name]/
│   ├── questions/                       # walk-friendly Q lists from /questions
│   │   └── [YYMMDD] - Walk questions.md
│   ├── board-reports/                   # board-reviewed reports archive
│   └── archive/                         # old content, retired
├── data/                                # structured data files (not content)
│   ├── [business-data-folder]/          # e.g. sales/, inventory/, awards/, financials/
│   ├── exports/                         # CSV exports from tools (analytics, CRM, etc.)
│   └── official-filings/                # public registry documents
└── experiments/                         # one-off analysis, exploration
    └── [YYMMDD] - [hypothesis]/
```

## User-level memory location

Memory files live OUTSIDE the project root, at:

```
~/.claude/projects/-[escaped-project-path]/memory/
├── MEMORY.md                            # primary memory index
├── project_[topic].md                   # topic memory files
├── feedback_[rule-name].md              # user preference rules
└── [project]_session_archive_[period].md  # archived Completed Work blocks
```

**Escaped path rule:** replace `/` with `-` in the project path.

Example: project at `/Users/jakub/projects/AlphaStudio` → memory at `~/.claude/projects/-Users-jakub-projects-AlphaStudio/memory/`.

## Naming conventions

### Files

- **Dated deliverables:** `[YYMMDD] - [Title].[ext]` (6-digit date, leading zero)
- **Dated + ticketed deliverables:** `[YYMMDD] - [TICKET-ID] - [Title].[ext]`
- **Session logs:** `[YYYY-MM-DD] - [type].md` (4-digit year for session logs)
- **Wiki pages:** lowercase-kebab-case.md
- **Templates:** `[name].template.[ext]`

### Folders

- **Dated campaign folders:** `[YYMMDD] - [campaign-name]/` (short title, no special chars)
- **Topic folders under context/:** lowercase
- **Raw asset folders inside a campaign:** `assets/`

### Slugs

- Lowercase, kebab-case, ASCII only
- Remove punctuation
- Target 3-5 words max
- Do NOT include dates in slugs (the folder already has them)

## Standard CLAUDE.md sections

Every project's CLAUDE.md contains these sections (in this order):

1. **Project identity** — name, legal entity, industry, one-sentence what-we-do
2. **Content Quality: Avoid AI Writing Patterns** — full no-AI-isms list + industry-specific bans
3. **Governance: [Project]-specific approval gates** — what needs approval, by whom
4. **Governance: Tool-write approval** — MCP writes that need per-use approval
5. **Link / URL conventions** — site structure, locale prefixes, CDN paths
6. **Wiki maintenance** — the "read wiki first, update on change, log every change" rule
7. **Branded reports standard** — if the project produces branded deliverables
8. **Industry-specific content restrictions** — e.g. medical health claims, alcohol age-gates, financial advice disclaimers, food allergen rules
9. **Bilingual / multilingual conventions** (if applicable)

## Standard MEMORY.md sections

Every project's MEMORY.md contains these sections (in this order):

1. **Company** — 5-10 bullet company facts (legal name, registration, ownership)
2. **Digital State** — current stack, live integrations, recent metrics
3. **[Industry integration]** — e.g. Shopify integration for e-commerce, EMR integration for medical
4. **MCP Integrations** — table of connected MCPs with status
5. **Project tracker** — Notion DB ID + data source ID, ticket count summary
6. **Active Projects** — in-flight initiatives with file paths
7. **Priorities** — top 3-5 for current quarter
8. **[Domain-specific active campaign]** — e.g. April 2026 paid ads, Q2 launch
9. **Governance** — summary of approval gates (full detail in CLAUDE.md)
10. **Key IDs & Accounts** — tracking IDs for tools
11. **Domain & DNS** (if relevant)
12. **EN Link Rules** (if bilingual site)
13. **Team/Agents** — roster + Paperclip agent handles if used
14. **Email Marketing** (if relevant)
15. **Session handoff docs** — list of in-flight session docs that should be read first
16. **Topic Memory Files** — one-line index of all `project_*.md`, `feedback_*.md`, integration guides
17. **Completed Work** — dated blocks of what shipped (archive older blocks to topic files)
18. **[Domain sections]** — e.g. "SEO & GEO — Single Source of Truth"
19. **Key Files** — one-line index of most important paths

**Rule:** MEMORY.md stays under 400 lines. Archive older Completed Work blocks to `[project]_session_archive_[period].md` when it grows past.

## Standard wiki structure

```
context/wiki/
├── index.md                # master catalog
├── log.md                  # chronological changelog
├── company/
│   ├── about.md
│   ├── registration.md     # legal entity details
│   ├── finance.md          # financial overview (last-filed year or more)
│   ├── team.md
│   ├── goals.md
│   └── [awards.md, history.md, etc.]
├── customers/
│   ├── segments.md
│   ├── icp.md
│   └── [data.md]
├── market/
│   ├── industry-overview.md
│   ├── competitors.md
│   ├── trends.md
│   └── [regulatory.md]
├── channels/               # per-channel playbook
│   ├── website.md
│   ├── social.md
│   ├── email.md
│   ├── paid-ads.md
│   └── [specific platforms]
└── operations/
    ├── integrations.md      # which MCPs + tools
    ├── recurring-register.md  # weekly/monthly/quarterly/annual tasks
    ├── reports.md           # report formats, templates
    └── [SOPs for common operations]
```

**Wiki rule:** under 300 lines per page. Synthesize, don't duplicate. Every factual number has a date stamp.

## .claude/ folder conventions

Every project's `.claude/` contains:

```
.claude/
├── settings.local.json              # MCP permissions, tool allowlists
├── research.md                      # project config for /research
├── review.md                        # /review config
├── questions.md                     # /questions config
├── tidy.md                          # /tidy config
├── draft.md                         # /draft config
├── wrap-session.md                  # wrap-session config
├── launch.json                      # (optional) Claude Code launch preferences
├── plans/                           # (optional) multi-step plans the agent is executing
└── skills/                          # (optional) project-local skills that shouldn't be user-level
```

## output/ folder conventions

- **Nothing gets deleted from `output/`.** Move to `output/archive/` if superseded.
- **Every deliverable has a ticket.** If you can't find the ticket for a file, either create one or archive the file.
- **Branded HTML reports** follow the template at `context/templates/branded_report_builder.py` (or equivalent).
- **Drafts use `_internal` prefix** if they contain non-publishable info (triggers public-content check).

## data/ folder conventions

- Structured files only (CSV, XLSX, JSON, YAML) — no prose.
- Files that may contain PII, client data, or regulated data go in `data/_restricted/` and are gitignored if using git.
- Raw exports from tools go in `data/exports/[source-name]/[YYMMDD].[ext]`.
- Preserved snapshots of official-record filings (company registry, regulatory, legal) go in `data/official-filings/` — immutable.

## Git (optional)

If the project uses git:

- `.gitignore` excludes: `data/_restricted/`, `output/drafts/_internal/`, any `.env`, any credentials JSON
- Session-log files in `context/sessions/` — committed or not per project preference
- Notion DB IDs are OK to commit (they're workspace identifiers, not secrets)
- Per-MCP tokens are NEVER committed
