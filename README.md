# ig-dedoomer

Chat with friends on Instagram without the Instagram app.

Instagram's app is built to keep you scrolling. If you only want it to talk to friends, you still have to pass the feed and reels to read a message. This project moves your DMs into a private Matrix server on your home PC. You read and reply in **Element X** on your iPhone, a plain chat app with no feed. Reels your friends send still show up in the chat. Instagram on the web is only a fallback.

```
Instagram <-> bridge <-> your Matrix server (home PC) <-> Tailscale <-> Element X on your iPhone
```

## Status

Working. The stack runs on the home PC, Element X on the iPhone reads and replies. See [docs/roadmap.md](docs/roadmap.md) for what is next.

> Running an Instagram bridge breaks Instagram's terms of service and can get an account challenged or locked. Read [docs/security.md](docs/security.md) and [docs/decisions/0003-host-at-home.md](docs/decisions/0003-host-at-home.md) before using your main account.

## Quickstart

Requirements on the PC: Docker (enabled at boot), `docker-compose`, your user in the `docker` group, Tailscale with HTTPS enabled. On the phone: Tailscale and Element X.

Already set up (the normal case after a reboot): the containers restart on their own. To start by hand:

```
docker-compose up -d
```

First-time setup needs generated configs, secrets, Tailscale and an Instagram login, so follow the [runbook](docs/runbook.md). The files in `config-examples/` show what to change; they are not drop-in replacements because their secrets are `CHANGE_ME`.

## Where things are

| Path | What |
|---|---|
| `docs/` | Human documentation: architecture, runbook, roadmap, security, decisions |
| `knowledge/` | Terse facts for AI agents (see `AGENTS.md`) |
| `docker-compose.yml` | Synapse, the Instagram bridge, Element Web |
| `config-examples/` | Sanitized configs, safe to commit |
| `element/config.example.json` | Element Web config template (copy to `config.json`, set your server name) |
| `bridge/`, `synapse/`, `backups/`, `samples/`, `.env` | **Secrets and private data. Git-ignored. Never share.** |

## Documentation

- [Architecture](docs/architecture.md) - how the pieces fit and why there are databases
- [Runbook](docs/runbook.md) - starting, logging in, connecting the phone, fixing, updating
- [Roadmap](docs/roadmap.md) - phases and what is next
- [Security](docs/security.md) - what was exposed, what is sensitive
- [Decisions](docs/decisions/) - why things are the way they are
