# Security notes

Things that were exposed or are sensitive, and what was done about them. Secret values are deliberately **not** written here.

## Exposed during setup (2026-10-09)

| What | Where it ended up | Status |
|---|---|---|
| Matrix access token for the old `@admin:localhost` (a `syt_...` token copied from an Element network request) | Pasted into the Claude Code chat; used for two read-only `GET`s against the media endpoint | **Resolved 2026-10-28.** Synapse was recreated with a new server name and database; the token no longer exists. |
| Instagram session cookies (sent to the bridge bot to log in) | Plain text in the bot DM room, therefore in Synapse's `homeserver.db` | The old database is in `backups/2026-10-28-localhost/`; delete it when no longer needed. For the new login (2026-10-28) the message should be removed in Element ("Remove"); Synapse keeps redacted content for ~7 days by default. |
| Instagram session / device state | `bridge/mautrix-meta.db` (and the old copy in `backups/`) | Keep out of backups that leave the machine; never commit. |

## Exposure through Tailscale

- Synapse is reachable at `https://<server-name>`, **only from devices in your tailnet**. Nothing is opened to the internet (Serve, not Funnel).
- HTTPS certificates are public: the machine name and tailnet name appear in Certificate Transparency logs. Do not put personal information into Tailscale machine names. The machine's earlier name may already be logged too.
- The repo never contains the real name: tracked files say `<server-name>`, and the real value lives in git-ignored files (`knowledge/local.md`, `element/config.json`, `synapse/`, `bridge/`).
- Anyone you share the tailnet with can reach Synapse's login page. Registration is closed, so they would still need your password.

## Sensitive files (all in `.gitignore`)

- `.env` - local UID/GID only, low risk
- `synapse/` - homeserver database (all bridged DMs), signing key, shared secrets in `homeserver.yaml`, media (friends' photos and reels)
- `bridge/` - bridge config, appservice tokens, Instagram session database
- `backups/` - the old setup, same sensitivity as the two above
- `samples/` - copies of friends' media used for testing

Do not commit, sync or share these folders. Check `git status` before every commit.

## Checklist

- [x] Fresh Matrix access tokens; old sessions gone (Synapse recreated 2026-10-28)
- [x] Fresh `registration_shared_secret`, `macaroon_secret_key`, `form_secret` (generated with the new config)
- [x] TLS in front of Synapse (Tailscale Serve); registration closed
- [x] Bridge port (29330) not exposed; only Synapse talks to it
- [ ] Cookie message in the bot DM removed
- [ ] Decide on backups: they contain private messages and the Instagram session
- [ ] Never paste tokens, cookies or cURL commands into chats, tickets or logs
