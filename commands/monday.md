---
description: Run the Monday operator-loop sequence for the current project (guard → resume → review → research)
---

Run the standard Monday operating sequence for **the current project** (auto-detect from the working directory + its `.claude/` config). Do them in order:

1. **Workspace guard.** Read `.claude/notion-registry.json` in the project root. `notion-fetch` the `home_page` ID. If it resolves to the expected `home_title`, the Notion connector is on the right workspace — proceed. If it errors or returns a different page, **STOP** and tell me: "Notion connector is on the wrong workspace — switch it to **[project]** before I touch the tracker." Do not write to Notion until this passes. (If there is no `notion-registry.json`, say so and fall back to local trackers.)

2. **resume-session** — load state (changelog, MEMORY, open queue). Give me a 3-line "where we are".

3. **/review** — surface everything awaiting my decision (In Review / Awaiting CEO + any new ticket comments). Walk them **one by one**, oldest first (decision-debt first). Apply each decision to the tracker as I answer.

4. **/research** — dispatch the ready Claude-owned research tickets so they run while I'm away; file results and queue them for next review.

I'll answer in shorthand — honor it: `go`/`approve` · `skip` · `defer [period]` · `reject — reason` · `comment N: …` · batch like `approve 1,3; defer 2; reject 4 — reason`.

Governance: nothing public or sent without my approval; no patient/PHI data through general AI; autonomy 4 — you prepare, I decide.
