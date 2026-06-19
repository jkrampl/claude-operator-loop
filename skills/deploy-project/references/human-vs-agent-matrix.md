# Human vs agent / LLM decision matrix

The framework for deciding who does what. Used during Phase 7 (backlog generation) for every ticket and during Phase 8 (governance codification) for defining gates.

## The 5-item closed list (repeated from operator-loop conventions)

Every ticket's `Owner` MUST be one of these. No exceptions.

| Value | Who | When |
|---|---|---|
| `Claude` | Autonomous LLM + tools, no gate | Pure research, drafting, analysis, read-only tool use, memory updates |
| `Claude (post-approval)` | LLM + tools after one-line CEO approval | Tool writes (Shopify, Notion batch, email drafts queued), anything touching external state |
| `CEO` | Principal / founder / project lead | Manual UI work, human meetings, board decisions, judgment calls, relationship touches |
| `External` | Dev / designer / vendor / contractor / law firm / accountant / auditor | Specialist work requiring a human expert outside the core team |
| `[Team member name]` | Named person on the team | Specific operational tasks tied to a role (sales, fulfillment, clinical, warehouse) |

## Decision tree

Use this sequence for every task:

```
┌────────────────────────────────────────────────────┐
│ 1. Does this task require a human judgment call    │
│    (business strategy, relationship, ethics,       │
│    subjective aesthetic, high-stakes decision)?    │
└────────────────────────────────────────────────────┘
        │
     YES│
        ▼
    ┌─────────────────────────────────┐
    │ Does it need SPECIFIC SPECIALIST│
    │ (dev, designer, lawyer, doctor, │
    │ accountant, engineer)?          │
    └─────────────────────────────────┘
          │
       YES│→ External
          │
        NO│
          ▼
    ┌─────────────────────────┐
    │ Does it need principal   │
    │ (CEO / founder / partner)│
    │ judgment specifically?   │
    └─────────────────────────┘
          │
       YES│→ CEO
          │
        NO│→ [Team member name]

        NO│ (not a human-only task)
          ▼
    ┌───────────────────────────────────┐
    │ 2. Does this task WRITE / CHANGE  │
    │ external state (publish, send,    │
    │ modify a paid platform, Shopify,  │
    │ email platform, ad account)?      │
    └───────────────────────────────────┘
          │
       YES│
          ▼
    ┌───────────────────────────────────┐
    │ Is it governance-gated            │
    │ (per CLAUDE.md marketing approval,│
    │ regulatory, financial, PHI, etc.)?│
    └───────────────────────────────────┘
          │
       YES│→ Claude (post-approval)
          │   (CEO approves per use)
          │
        NO│
          ▼
    ┌─────────────────────────────┐
    │ Pre-approved automation     │
    │ (SEO metadata, analytics    │
    │ reports, internal docs)?    │
    └─────────────────────────────┘
          │
       YES│→ Claude
          │
        NO│→ Claude (post-approval) (default to caution)

        NO│ (read-only or pure analysis)
          ▼
    ┌──────────────┐
    │ → Claude     │
    └──────────────┘
```

## Common task-to-owner patterns (cheat sheet)

### Research tasks
| Task | Owner | Why |
|---|---|---|
| Market research, competitor tracking | Claude | Pure research, no writes |
| Public-record financials lookup | Claude | Public data, read-only |
| Customer interview analysis (if transcripts anonymized) | Claude | Processing existing data |
| Draft a strategic recommendation | Claude | Draft; decision still CEO's |
| Deep-dive on a named customer account | CEO or team lead | Requires relationship context |

### Content tasks
| Task | Owner | Why |
|---|---|---|
| Draft blog post | Claude | Drafting is research's cousin |
| Approve blog post to publish | CEO | Governance gate per CLAUDE.md |
| Publish approved blog to CMS (Shopify write) | Claude (post-approval) | Tool write, post-approval |
| Write product description | Claude | Drafting |
| Ship product description to live | Claude (post-approval) | Shopify write |
| Schedule social post | Claude (post-approval) | If platform API approved for use |
| Reply to customer DM | CEO or Community Manager | Relationship |
| Create ad creative copy | Claude | Drafting |
| Launch ad spend | CEO | Budget approval |

### Operational tasks
| Task | Owner | Why |
|---|---|---|
| Check analytics, pull a report | Claude | Read-only |
| Update a pricing spreadsheet | Claude (post-approval) | Modifies source of truth |
| Change a Shopify product price | CEO | Pricing is strategic |
| Close a ticket after work done | Claude | Administrative |
| Archive old files | Claude | Housekeeping |
| Install a new integration | CEO or Claude (post-approval) | Depends on auth flow — OAuth may need CEO |
| Respond to a supplier inquiry | Team member or CEO | Relationship |
| Negotiate a contract | CEO + External legal | Judgment + legal |
| Approve an invoice for payment | CEO | Cash disbursement |
| Pay an invoice | Team member or automated | Operational |

### Strategic tasks
| Task | Owner | Why |
|---|---|---|
| Pick annual OKRs | CEO | Strategic judgment |
| Decide hiring plan | CEO | Human decision |
| Interview a candidate | CEO + maybe External | Relationship |
| Close a candidate | CEO | Offer decision |
| Board-deck first draft | Claude | Drafting |
| Present board deck | CEO | In-person |
| Approve final deck | CEO | Governance |
| Decide to enter a new market | CEO + board | Strategic |

### Technical tasks
| Task | Owner | Why |
|---|---|---|
| Review a GitHub PR | External (or dev team) | Code review |
| Merge a PR | External | Governance |
| Deploy to production | External or Claude (post-approval) | Depends on pipeline |
| Troubleshoot a bug report | Claude or External | Claude for triage, External for fix |
| Write code | External | Specialist work |
| Write SEO meta tags | Claude | Pre-approved per CLAUDE.md SEO exception |
| Update robots.txt / sitemap | Claude (post-approval) | Writes to theme |

### Legal, compliance, finance
| Task | Owner | Why |
|---|---|---|
| Draft a privacy policy | Claude | Draft; requires lawyer review |
| Approve privacy policy for publish | CEO + External (lawyer) | Liability gate |
| Update company registration | CEO + External | Regulatory filing |
| File statutory accounts | External (accountant) | Specialist |
| Tax filing | External | Specialist |
| Publish financial press release | CEO | Governance gate |
| Customer contract signature | CEO | Authority |
| Employment offer letter | CEO + External (HR/legal) | Judgment + legal |

## Industry-specific owner adjustments

### Medical
- Any patient-facing content → needs clinical sign-off (External or clinician Team member)
- Any health claim → Claude drafts but CEO cannot unilaterally approve; needs medical advisory
- PHI handling → never Claude, ever

### Finance / investment
- Investment recommendations → only External (licensed advisor)
- Performance reporting → Claude drafts + CEO approves + External reviews for compliance
- Marketing materials → triple gate (Claude + CEO + External compliance)

### Family office
- Default: if in doubt, CEO
- Nothing goes Claude autonomous without explicit written scope
- All communications mentioning family → principal approval

### Food & bev + alcohol
- Allergen labels → External (regulatory specialist) approves, Claude drafts
- Recipe changes → Chef/Winemaker (Team member) approves
- Age-gate compliance → CEO (liability)

### Manufacturing
- ECO (engineering change order) → External (engineering) + Team member (quality)
- Supplier substitutions → CEO or COO (depends on threshold)
- Product claims → External (compliance) approves

## Approval cadence patterns

### Pre-approved (Claude autonomous)
Actions the CEO has given blanket approval for:
- SEO meta tags + alt text + schema markup
- Internal-only documents and reports
- Data pulls and analysis
- Research deliverables
- First-draft content (stays in `output/drafts/` until approved)

### Per-use approved (Claude post-approval)
Actions that need one-line "go" each time:
- Shopify writes (product, page, article, redirect, discount)
- Email campaign send
- Social post publish
- Ad campaign launch
- Public content publish
- CRM bulk update

### CEO-only
Actions the CEO never delegates:
- Pricing decisions above threshold
- Hiring / firing
- Strategic pivots
- Contract signatures
- Board communications
- Material financial commitments

## Governance decision tree for ambiguous cases

When unclear, ask these in order:

1. **Reversibility?** — Can this be undone in <1 hour at <€100 cost? If yes → lean autonomous.
2. **Liability?** — Does this create a legal, regulatory, or reputational risk? If yes → CEO or External.
3. **Relationship impact?** — Does this touch a customer, supplier, or partner relationship? If yes → human owner.
4. **Data sensitivity?** — Does this process PII, PHI, financial, or legal data? If yes → escalate.
5. **Budget impact?** — Above the CEO's pre-approved threshold? If yes → CEO (or board).
6. **Precedent?** — Have we done this exact thing before via Claude with good outcome? If yes → lean autonomous.

## Escalation rules

- **Claude encounters ambiguity** → pauses, creates a ticket for CEO clarification, does NOT improvise
- **Team member encounters decision beyond their scope** → escalates to CEO with context
- **External specialist flags a risk** → CEO owns the decision; External documents their opinion
- **Conflicting signals from governance + research** → governance always wins; update research

## Annual review

Every year during the Q4 `/tidy` cycle, review the owner distribution:

- Are too many items going to CEO? (Overload — delegate more to Claude or team)
- Are things going to Claude that should be CEO? (Governance drift — re-tighten)
- Are External gates slowing work? (Contract more capacity)
- New team members — update the `[Team member name]` column?
