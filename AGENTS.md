# ig-dm-filter - agent guide

(`CLAUDE.md` is a symlink to this file. Edit `AGENTS.md` only.)

## What this is

A self-hosted pipeline that lets the user read Instagram DMs **without using the Instagram app**:

```
Instagram -> mautrix-meta (Instagram build) -> Synapse (Matrix) -> [filter bot, planned] -> ntfy push -> iPhone
```

Goal: only plain text messages notify the user. Reels and posts become a placeholder ("X sent a reel"). The user replies by hand in Safari on Instagram web (never on the laptop). Hosting target: the user's always-on home PC, so Instagram sees a residential IP.

Project root: `~/Projects/side-projects/ig-dm-filter` (its own git repo, **not** inside `my-chat-with-ai`).

## Start here

1. Read `knowledge/status.md` - current state and the next step.
2. Then read only what the task needs, using the index below.

| Need | Read |
|---|---|
| Services, ports, paths, identities | `knowledge/stack.md` |
| Exact commands (start, logs, login, fixes) | `knowledge/commands.md` |
| How text/reels/posts arrive in Matrix (the filter's contract) | `knowledge/message-formats.md` |
| Past mistakes to avoid | `knowledge/bridge-gotchas.md` |
| Inspecting Synapse data safely | `knowledge/sqlite-queries.md` |
| Hard rules | `knowledge/rules.md` |
| Why a decision was made | `docs/decisions/` |

## Hard rules (full list in `knowledge/rules.md`)

- Never print, log, echo or commit tokens, cookies, cURL commands, passwords or `synapse/`, `bridge/`, `samples/`, `.env`.
- Never print the text of friends' messages. Print message **types and structure** only.
- Use the Instagram image `dock.mau.dev/mautrix/meta:ig-latest`. The `latest` tag is the Messenger bridge.
- The agent has no `sudo`. Ask the user to run privileged commands with `! <command>`.
- Confirm before anything destructive on Docker volumes or on `bridge/` / `synapse/` (they hold the Instagram session and all DMs).

## Conventions

- Shell is fish. Compose is the standalone binary: `docker-compose`, not `docker compose`.
- Each fact lives in one place. `knowledge/` states facts tersely; `docs/` holds narrative and reasoning. Link, do not copy.
- At the end of every work session or finished phase, update `knowledge/status.md` (and tick `docs/roadmap.md`).
- English everywhere.
- Commit docs, compose file, `config-examples/` and `element/config.json`. Never commit anything ignored by `.gitignore`.
