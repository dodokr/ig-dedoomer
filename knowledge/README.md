# knowledge/ - index for AI agents

Terse, factual, one topic per file. Humans: read `../docs/` instead.

| File | Read when |
|---|---|
| `status.md` | **Always first.** Current state, open items, next step. |
| `stack.md` | You need ports, paths, image tags, usernames, config deltas. |
| `commands.md` | You need to start/stop/inspect the stack or walk the user through login. |
| `message-formats.md` | You are writing or testing a bot that reads bridged messages. |
| `bridge-gotchas.md` | Something behaves oddly (wrong login flow, permission errors, no Docker access). |
| `sqlite-queries.md` | You need to look at events or media in Synapse. |
| `rules.md` | Before touching secrets, message content, Docker volumes or git. |
| `local.md` | You need the real server name or host details (git-ignored; may be missing on a fresh clone). |

Rules for editing this folder:
- One fact, one place. Reasoning belongs in `../docs/decisions/`; link to it.
- Mark anything not verified as `UNVERIFIED`.
- Date the `status.md` header every time it changes.
