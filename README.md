# ig-dm-filter

Read your Instagram DMs without using the Instagram app, and get a push notification only for messages you actually care about.

Instagram's app is built to keep you scrolling. If you only want it to chat with friends, you still have to open the feed to see a message. This project moves your DMs into a private Matrix server you control, filters out the noise (reels, shared posts) and sends the rest to your phone as a plain notification. You reply from Instagram on the web (Safari on iPhone) when you want to.

```
Instagram -> bridge -> your Matrix server -> filter bot -> ntfy -> your iPhone
```

## Status

Early, working locally. The bridge and Matrix server run in Docker and your Instagram DMs flow in. The filter bot and notifications are **not built yet**. See [docs/roadmap.md](docs/roadmap.md).

> Running an Instagram bridge breaks Instagram's terms of service and can get an account challenged or locked. Read [docs/security.md](docs/security.md) and [docs/decisions/0003-host-at-home.md](docs/decisions/0003-host-at-home.md) before using your main account.

## Quickstart (local)

Requirements: Docker, `docker-compose`, your user in the `docker` group, Firefox.

Already set up (the normal case after a reboot):

```
sudo systemctl start docker
docker-compose up -d
```

First-time setup needs generated configs, secrets and an Instagram login, so follow the [runbook](docs/runbook.md). The files in `config-examples/` show what to change; they are not drop-in replacements because their secrets are `CHANGE_ME`.

## Where things are

| Path | What |
|---|---|
| `docs/` | Human documentation: architecture, runbook, roadmap, security, decisions |
| `knowledge/` | Terse facts for AI agents (see `AGENTS.md`) |
| `docker-compose.yml` | Synapse, the Instagram bridge, Element Web |
| `config-examples/` | Sanitized configs, safe to commit |
| `element/config.json` | Element Web config (no secrets) |
| `bridge/`, `synapse/`, `samples/`, `.env` | **Secrets and private data. Git-ignored. Never share.** |

## Documentation

- [Architecture](docs/architecture.md) - how the pieces fit and why there are databases
- [Runbook](docs/runbook.md) - starting, logging in, fixing, updating
- [Roadmap](docs/roadmap.md) - phases and what is next
- [Security](docs/security.md) - what was exposed, what to do before deploying
- [Decisions](docs/decisions/) - why things are the way they are
