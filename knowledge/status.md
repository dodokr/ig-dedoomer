# Status

**Last updated: 2026-10-28.** Update this file at the end of every session.

## State

- Phases 0-3 done (see `../docs/roadmap.md`): stack ran locally, Instagram logged in via cookies, double puppeting enabled, message formats mapped.
- Phase 4a (filter bot + ntfy) **not started**. No filter code and no ntfy container exist yet.
- Docker daemon was **not running** on 2026-10-28 (machine rebooted). User must run `! sudo systemctl start docker`.
- Git repo exists, **no commits yet**.
- Last live check of the stack: 2026-10-12 (not re-verified since).

## Decisions made

- Instagram bridge image: `dock.mau.dev/mautrix/meta:ig-latest` (`docs/decisions/0001`).
- SQLite for Synapse and bridge while local (`0002`).
- **Host on the user's home PC**, not a VPS, to keep a residential IP (`0003`).
- Notifications via ntfy + filter bot; Element X optional (`0004`).
- Reels are kept; plan to add Synapse `media_retention` (e.g. 30d) at deployment (`0005`). Not yet applied.
- Reels/posts: placeholder notification for the first version. No LLM step yet.

## Open items

- [ ] Verify the cookie cURL message in the bot DM was redacted (it holds Instagram session cookies).
- [ ] An admin Matrix access token was pasted in a chat on 2026-10-09. Log out that session / rotate before any non-local deployment (`../docs/security.md`).
- [ ] Ask a friend to send a plain photo, a voice note, a story reply, a sticker -> sample them, fill `message-formats.md`.
- [ ] Make the first git commit (docs, compose, config-examples, element/config.json).

## Next step

Phase 4a: add `ntfy` to `docker-compose.yml`; write a Python (`matrix-nio`) bot that logs in as its own Matrix user, watches bridged rooms, and:
- plain `m.text` -> push with sender + text
- reel/post (see `message-formats.md`) -> placeholder push
- unknown types -> generic placeholder
- ignore own messages and bridge-bot messages
Unit tests built from real event shapes. Open questions still unanswered by the user: LLM for reel summaries (Claude API vs local) is deferred to Phase 4b; output for v1 = ntfy push + local log file.

## Later (home deployment)

Docker enabled at boot, fixed LAN address, Tailscale for remote access, ntfy upstream via ntfy.sh for iPhone push (UNVERIFIED, check ntfy docs), `media_retention`, backups.
