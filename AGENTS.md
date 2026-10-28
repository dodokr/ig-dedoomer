# ig-dedoomer - agent guide

(`CLAUDE.md` is a symlink to this file. Edit `AGENTS.md` only.)

## What this is

A self-hosted setup that lets the user chat on Instagram **without using the Instagram app** (no feed, no reels tab):

```
Instagram <-> mautrix-meta (Instagram build) <-> Synapse (Matrix) <-> Tailscale Serve (HTTPS) <-> Element X on iPhone
```

The user reads and replies in Element X. Reels shared by friends are shown in the chat (fine with the user). Safari on instagram.com is only a fallback for things the bridge cannot do; never Instagram on the laptop. Everything runs on the user's always-on home PC, so Instagram sees a residential IP.

Project root: `~/Projects/side-projects/ig-dedoomer` on the home PC (`ssh <pc>`, see `knowledge/local.md`). Its own git repo, **public on GitHub**: personal details go only into `knowledge/local.md` (git-ignored). The old laptop copy (`ig-dm-filter`) is retired: never start its stack (two bridges on one Instagram session).

## Start here

1. Read `knowledge/status.md` - current state and the next step. Read `knowledge/local.md` if it exists (real names, hosts).
2. Then read only what the task needs, using the index below.

| Need | Read |
|---|---|
| Services, ports, paths, identities | `knowledge/stack.md` |
| Exact commands (start, logs, login, fixes) | `knowledge/commands.md` |
| How text/reels/posts arrive in Matrix (contract for any bot) | `knowledge/message-formats.md` |
| Past mistakes to avoid | `knowledge/bridge-gotchas.md` |
| Inspecting Synapse data safely | `knowledge/sqlite-queries.md` |
| Hard rules | `knowledge/rules.md` |
| Why a decision was made | `docs/decisions/` |

## Hard rules (full list in `knowledge/rules.md`)

- Never print, log, echo or commit tokens, cookies, cURL commands, passwords or `synapse/`, `bridge/`, `backups/`, `samples/`, `.env`.
- Never print the text of friends' messages. Print message **types and structure** only.
- Use the Instagram image `dock.mau.dev/mautrix/meta:ig-latest`. The `latest` tag is the Messenger bridge.
- The agent has no `sudo`. Ask the user to run privileged commands with `! <command>` (or in a terminal on the PC).
- Confirm before anything destructive on Docker volumes or on `bridge/` / `synapse/` (they hold the Instagram session and all DMs).
- Bots never post into bridged rooms (it can reach friends).
- The repo is public. Never write the real Tailscale name, hostnames, SSH aliases, IPs, usernames or other personal details into a tracked file. Use placeholders (`<server-name>`, `<pc>`); real values are in `knowledge/local.md` (git-ignored).

## Conventions

- The PC's login shell is fish. Over SSH, wrap multi-line or `VAR=` scripts in `bash -s <<'EOF' ... EOF`.
- Compose is the standalone binary: `docker-compose`, not `docker compose`.
- Each fact lives in one place. `knowledge/` states facts tersely; `docs/` holds narrative and reasoning. Link, do not copy.
- At the end of every work session or finished phase, update `knowledge/status.md` (and tick `docs/roadmap.md`).
- English everywhere.
- Commit docs, compose file, `config-examples/` and `element/config.example.json`. Never commit anything ignored by `.gitignore`.
