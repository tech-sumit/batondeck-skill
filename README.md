# conductor-skill

An installable agent skill for **[Conductor](https://github.com/tech-sumit/conductor)** — the
MCP-native task orchestrator. It teaches your coding agent to **plan** work onto a shared Conductor
Kanban board (a richly-detailed dependency tree) and **work** it autonomously over MCP — claiming
tasks, loading each task's full context, doing the work, and auto-unblocking what comes next — safely
alongside other agents and humans.

Ships for both **Cursor** (a `.cursor` rule + MCP server) and **Claude Code** (a `SKILL.md` skill).
Interactive use needs **no secrets and no `gcloud`**: the MCP endpoint speaks OAuth 2.1, so your MCP
client (Cursor / `mcp-remote`) signs you in through a normal Google browser flow. A `gcloud`-based
script path is kept as an optional advanced/headless option.

## Prerequisites

- `node`/`npx` — the Cursor MCP server uses `mcp-remote`, which drives the OAuth browser flow.
- A Google account. On first connect you'll get a browser sign-in (Google account picker) — that's it.
- Membership in a Conductor project. An operator adds you with
  `add_member { projectId, identityId: "<your-email-or-SA>", role: "agent" | "member" | "admin" }`.
  (A valid sign-in with no membership sees nothing — authorization is by project membership.)
- *(Advanced / headless only)* `gcloud` authenticated (`gcloud auth login`), for the optional
  token-minting script path. For an **agent service account** add it with `add_member` and grant it
  `roles/run.invoker` on the core (the Conductor repo's `scripts/onboard-agent.sh <sa-email>` does both).

## Install — Cursor

1. Clone this repo into (or beside) your project so `.cursor/` and `scripts/` are at the project root:
   ```bash
   git clone https://github.com/tech-sumit/conductor-skill
   # then copy .cursor/ and scripts/ to your project root, or work inside this repo
   ```
2. **Rule** — `.cursor/rules/conductor-worker.mdc` is picked up automatically; Cursor's agent applies
   it whenever you ask it to plan or work the Conductor board.
3. **MCP server** — enable `conductor` in **Cursor Settings → MCP**. `.cursor/mcp.json` already points
   `mcp-remote` at the hosted MCP endpoint (`https://conductor-mcp-…/mcp`); on first use Cursor opens a
   browser to sign in with Google, then the tools are available natively. For a self-hosted core, change
   the URL in `.cursor/mcp.json` to your own mcp-gateway `/mcp`.
4. **Or skip MCP** and let the agent use the terminal scripts directly — see the advanced Quickstart.

## Install — Claude Code

Make this repo a skill (it has `SKILL.md` at the root):

```bash
git clone https://github.com/tech-sumit/conductor-skill \
  ~/.claude/skills/conductor-worker          # user-level; or <project>/.claude/skills/conductor-worker
```

Claude Code loads `conductor-worker` automatically when a task matches its description.

## Quickstart — advanced (terminal scripts + gcloud, for headless/CI)

For interactive use prefer the MCP server above (browser OAuth, no gcloud). The scripts below are the
optional headless path: they mint a Google ID token with your own `gcloud` and call the core directly.

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
| `.cursor/mcp.json` | Cursor MCP server (`conductor`) — `mcp-remote` → the mcp-gateway `/mcp` (OAuth browser flow) |
| `SKILL.md` | Claude Code skill (same playbook) |
| `references/tools.md` | Full MCP tool list + error codes |
| `scripts/token.sh` | *(advanced)* mint `CONDUCTOR_TOKEN` (Google ID token) via `gcloud` |
| `scripts/mcp.sh` | *(advanced)* one-shot MCP tool caller (own session) — the headless shell building block |
| `scripts/cursor-mcp.sh` | *(advanced)* gcloud-token launcher that bridges stdio↔HTTP via `mcp-remote` |
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
