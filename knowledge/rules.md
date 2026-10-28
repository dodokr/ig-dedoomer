# Rules for agents

## Secrets
- Never print, echo, log, summarize or commit: Matrix access tokens (`syt_...`), Instagram cookies or cURL commands, passwords, `as_token` / `hs_token`, anything in `.env`, `bridge/`, `synapse/`.
- If the user pastes a secret into the chat: use it only for what they asked (and say so), then tell them how to rotate it and add it to `docs/security.md`. Never write the value to a file.
- When inspecting configs, mask values (`sed -E 's/(:\s*).*/\1<hidden>/'`).
- Example configs go through the sanitize-and-verify step: replace the values, then assert that none of the originals remain in the output.

## Privacy
- Messages in Synapse are friends' private DMs. Print types, structure, lengths. Never print `m.text` content, sender display names or phone numbers.
- Media in `synapse/media_store/` and `samples/` is friends' content. Do not send it to any external service (LLM, upload, paste) without the user's explicit OK for that action.

## Safety
- Docker volumes and `bridge/`, `synapse/` hold the Instagram session and all DMs. Ask before deleting, recreating or `chown`-ing them, and before removing images that other work depends on.
- Do not run the bridge with credentials from a datacenter IP or add proxies/VPNs without asking: account-flag risk is the main danger (`../docs/decisions/0003-host-at-home.md`).
- No automated Instagram requests other than the bridge itself (no `yt-dlp`, no scraping, no extra logins). The bridge already stores reel mp4s in Synapse.
- Do not post anything into bridged rooms from a bot: it can reach friends. Output goes to ntfy and logs.
- The agent has no `sudo`; ask the user to run privileged commands with `! <command>`.

## Git
- Track: `README.md`, `AGENTS.md`, `CLAUDE.md` (symlink), `docs/`, `knowledge/`, `docker-compose.yml`, `config-examples/`, `element/config.json`, `.env.example`, `.gitignore`, future filter-bot code.
- Never track what `.gitignore` lists. Check `git status` before any commit.
- Do not commit or push unless the user asks.

## Housekeeping
- Update `status.md` when a session ends or a phase completes.
- Use the scratchpad directory for temporary files, not the project tree.
