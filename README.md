# Claude Operator-Loop Skills

A portable set of [Claude Code](https://claude.com/claude-code) skills that turn a project into a
self-running "operator loop": a single Notion source of truth, a weekly cadence of research →
review → tidy → draft, and clean session start/stop rituals. Industry-agnostic — the examples are
generic; point it at any project.

## What's inside

| Skill | What it does |
|---|---|
| `deploy-project` | Scaffold a new or existing project onto the operator-loop system (folders, CLAUDE.md, MEMORY.md, wiki, Notion workspace, governance). |
| `research` | Scan the task tracker for research tickets, fan out parallel agents, auto-file outputs. |
| `review` | Structured session that surfaces everything awaiting a decision and applies the outcomes. |
| `questions` | Capture and work an open-questions list. |
| `tidy` | Dedupe / cancel / rebalance the task backlog against calendar + velocity. |
| `draft` | Draft content (blog, email, social, outreach, one-pager) from type templates. |
| `wrap-session` | Persist project state at the end of a working session. |
| `resume-session` | Reload the latest project state at the start of a session. |
| `weekly-social` | Batch-prepare the week's social posts from a posting calendar. |
| `OPERATOR-LOOP.md` | The framework doc tying the loop together. |
| `_AGENTS.md`, `_CLAUDE.md` | Shared agent / config templates. |
| `_tools/` | Reference library of integration docs + CLI helpers (analytics, CRM, SEO, email, etc.). |

## Install

Copy the folders into your Claude Code skills directory:

```bash
git clone <this-repo> claude-operator-loop
cp -R claude-operator-loop/skills/* ~/.claude/skills/
```

Then invoke any skill in Claude Code with its slash command, e.g. `/resume-session`, `/research`,
`/tidy`. Start with `/deploy-project` to wire a project up from scratch.

## Notes

- Examples use placeholder names (`Acme Co`, `ACME-RE`, `your-workspace`) — swap in your own.
- `_tools/` documents how to call third-party APIs; it reads credentials from environment variables,
  never hardcoded. Add your own keys via your shell env.
- These skills assume a Notion workspace as the task/wiki source of truth (configurable).

## License

MIT — see [LICENSE](LICENSE).
