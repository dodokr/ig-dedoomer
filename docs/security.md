# Security notes

Things that were exposed or are sensitive during local development, and what to do before this leaves the laptop. Secret values are deliberately **not** written here.

## Exposed during setup (2026-10-09)

| What | Where it ended up | Risk now | Action |
|---|---|---|---|
| Matrix access token for `@admin:localhost` (a `syt_...` token copied from an Element network request) | Pasted into the Claude Code chat; used for two read-only `GET`s against the media endpoint | Full access to the admin account, but Synapse listens on `127.0.0.1` only and federation is off | Log out of Element (invalidates it). If double puppeting breaks afterwards, re-run `login-matrix <new token>` with a fresh token. **Must be done before any public deployment.** |
| Instagram session cookies (cURL command sent to the bridge bot to log in) | Plain text in the bot DM room, therefore in Synapse's `homeserver.db` | Anyone with the file can act as the Instagram session | Redact the message in Element ("Remove"); Synapse keeps redacted content for ~7 days by default. Consider logging out of the Instagram session from the Instagram app once the bridge is stable and re-login later if needed. |
| Instagram session / device state | `bridge/mautrix-meta.db` | Same as above | Keep out of backups that leave the machine; never commit. |

## Sensitive files (all in `.gitignore`)

- `.env` - local UID/GID only, low risk
- `synapse/` - homeserver database (all bridged DMs), signing key, shared secrets in `homeserver.yaml`, media (friends' photos and reels)
- `bridge/` - bridge config, appservice tokens, Instagram session database
- `samples/` - copies of friends' media used for testing

Do not commit, sync or share these folders. If this directory ever becomes a git repo, check `git status` before the first commit.

## Before deploying to a VPS / home server

- [ ] Fresh Matrix access tokens; log out all existing sessions
- [ ] Rotate `registration_shared_secret`, `macaroon_secret_key`, `form_secret` in `homeserver.yaml`
- [ ] TLS in front of Synapse; keep registration closed
- [ ] Do not expose the bridge port (29330); only Synapse talks to it
- [ ] Never paste tokens, cookies or cURL commands into chats, tickets or logs
- [ ] Decide on backups: they contain private messages and the Instagram session
