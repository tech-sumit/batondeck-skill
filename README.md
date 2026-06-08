# conductor-skill

An installable agent skill for **[Conductor](https://github.com/tech-sumit/conductor)** — the
MCP-native task orchestrator. It teaches your coding agent to **plan** work onto a shared Conductor
Kanban board (a richly-detailed dependency tree) and **work** it autonomously over MCP — claiming
tasks, loading each task's full context, doing the work, and auto-unblocking what comes next — safely
alongside other agents and humans.

Ships for both **Cursor** (a `.cursor` rule + MCP server) and **Claude Code** (a `SKILL.md` skill),
over one set of deployment-agnostic scripts. No secrets: the scripts mint short-lived Google ID
tokens via your own `gcloud`.

## Prerequisites

- `gcloud` CLI, authenticated (`gcloud auth login`) — used to mint the core's ID token.
- Membership in a Conductor project. An operator adds you with
  `add_member { projectId, identityId: "<your-email-or-SA>", role: "agent" | "member" | "admin" }`.
  For an **agent service account**, it also needs `roles/run.invoker` on the core (the Conductor repo's
  `scripts/onboard-agent.sh <sa-email>` does both).
- `node`/`npx` (only for the native Cursor MCP server, which uses `mcp-remote`).

## Install — Cursor

1. Clone this repo into (or beside) your project so `.cursor/` and `scripts/` are at the project root:
   ```bash
   git clone https://github.com/tech-sumit/conductor-skill
   # then copy .cursor/ and scripts/ to your project root, or work inside this repo
   ```
2. **Rule** — `.cursor/rules/conductor-worker.mdc` is picked up automatically; Cursor's agent applies
   it whenever you ask it to plan or work the Conductor board.
3. **MCP server** — enable `conductor` in **Cursor Settings → MCP**. Edit `.cursor/mcp.json`'s `env`:
   set `CONDUCTOR_AGENT_SA` to your agent's service-account email (leave empty to use your own gcloud
   login), and `CONDUCTOR_CORE_URL` for a self-hosted core. The agent can then call the tools natively.
4. **Or skip MCP** and let the agent use the terminal scripts directly (always works) — see Quickstart.

## Install — Claude Code

Make this repo a skill (it has `SKILL.md` at the root):

```bash
git clone https://github.com/tech-sumit/conductor-skill \
  ~/.claude/skills/conductor-worker          # user-level; or <project>/.claude/skills/conductor-worker
```

Claude Code loads `conductor-worker` automatically when a task matches its description.

## Quickstart (terminal scripts — works everywhere)

```bash
export CONDUCTOR_AGENT_SA=my-agent@my-proj.iam.gserviceaccount.com   # or leave unset to use your gcloud login
eval "$(scripts/token.sh)"                                           # exports CONDUCTOR_TOKEN
scripts/mcp.sh list_projects '{}'                                    # discover your projects
export CONDUCTOR_PROJECT=P-…  CONDUCTOR_BOARD=B-…
scripts/mcp.sh next_task "{\"projectId\":\"$CONDUCTOR_PROJECT\",\"boardId\":\"$CONDUCTOR_BOARD\"}"
```

Every `tool { … }` in the rule/skill maps to `scripts/mcp.sh tool '{ … }'`.

## What's inside

| Path | Purpose |
|---|---|
| `.cursor/rules/conductor-worker.mdc` | Cursor rule — the planner/worker playbook |
| `.cursor/mcp.json` | Cursor MCP server (`conductor`) via the launcher below |
| `SKILL.md` | Claude Code skill (same playbook) |
| `references/tools.md` | Full MCP tool list + error codes |
| `scripts/token.sh` | mint `CONDUCTOR_TOKEN` (Google ID token, audience = core URL) |
| `scripts/mcp.sh` | one-shot MCP tool caller (own session) — the shell building block |
| `scripts/cursor-mcp.sh` | native MCP launcher (mints a token, bridges stdio↔HTTP via `mcp-remote`) |
| `scripts/seed-tasknet.py` | plant a whole dependency tree from a JSON plan in one session |
| `scripts/worker.sh` | one decentralized worker loop (claim → run `AGENT_CMD` → repeat) |
| `scripts/fleet.sh` | run many workers in parallel to drain a board at max concurrency |

## Connection env

| env | meaning |
|---|---|
| `CONDUCTOR_CORE_URL` | core base URL (default: the hosted reference instance; set for self-hosted) |
| `CONDUCTOR_TOKEN` | a Google ID token (audience = core URL) — bring your own, or leave unset to mint |
| `CONDUCTOR_AGENT_SA` | mint a token by impersonating your agent's service account |
| `CONDUCTOR_ON_BEHALF_OF` | optional on-behalf-of identity (only when authing as a trusted gateway SA) |
| `CONDUCTOR_PROJECT` / `CONDUCTOR_BOARD` | the project/board the worker + fleet operate on |

## License

MIT — see [LICENSE](LICENSE).
