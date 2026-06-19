---
description: Execute the next N Claude-owned Todo tickets for the current project, in order
---

Execute the next Claude-owned, unblocked tickets from **the current project's** tracker, in priority order. Default **N = 3** unless I say otherwise (`/work 5`, `/work the M&A ones`, `/work ACME-001`).

1. **Workspace guard first** — same as `/monday` step 1: `notion-fetch` the `home_page` in `.claude/notion-registry.json`; **STOP** if it doesn't resolve (wrong workspace).

2. **Select.** Pull tickets where Owner = `Claude` or `Claude (post-approval)`, Status = `Todo`, with no open blocking dependency. Prefer P0 → P1. **Skip** anything owned by `CEO` or `External` — list those separately as "waiting on a human (you need to engage/decide)". This is the do-now vs blocked-on-a-hire split.

3. **Do each, in order:** set Status `In Progress` → do the work (research / draft / analysis on **non-PHI, synthetic data only**) → file the deliverable to the right `output/` folder → set Status `In Review` with a one-line result.

4. **Close-out ritual (every ticket):** fill `Actual Hours` + a one-line `Reflection`; append the key decision/finding to `context/decisions.md` (create it if missing) as `## [date] [TICKET-ID] — decision/finding (why)`.

5. **Stop at the gate.** Anything needing my approval or an external write (publish, send, contact a target/seller/patient, real-data access): prepare it, leave it `In Review`, do **not** execute.

6. **Report** a compact summary: ✅ done-to-review · ⏳ blocked-on-human (with who) · ⛔ skipped (why).

Governance: no PHI through general AI; nothing public without my approval; autonomy 4.
