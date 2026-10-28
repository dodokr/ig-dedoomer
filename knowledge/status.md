# Status

**Last updated: 2026-10-28.** Update this file at the end of every session.

## State

- Runs on the home PC (`ssh <pc>`, project `~/Projects/side-projects/ig-dedoomer`). Real server name and host details: `local.md` (git-ignored). The laptop copy is retired; its bridge DB was deleted.
- Synapse **recreated 2026-10-28** with `server_name: <server-name>` (was `localhost`). Old data in `backups/2026-10-28-localhost/` (git-ignored).
- `tailscale serve --bg 8008` -> `https://<server-name>` (tailnet only). `/.well-known/matrix/client` served by Synapse. Native sliding sync and password login advertised.
- Bridge re-logged in with cookies; double puppeting set; 16 portals at last check. Backfill enabled (see `stack.md`).
- User reads and replies in **Element X on iPhone** - works (user-confirmed 2026-10-28).
- `media_retention.local_media_lifetime: 30d` applied.
- Recurring harmless bridge log errors: `last read message not found` (read receipts for events the bridge never saw), `cannot change members for DM`.
- Git: first commit 089b60a; this session's doc changes uncommitted.

## Decisions made

- Instagram bridge image: `dock.mau.dev/mautrix/meta:ig-latest` (`docs/decisions/0001`).
- SQLite for Synapse and bridge (`0002`).
- **Host on the user's home PC**, not a VPS, to keep a residential IP (`0003`).
- ~~ntfy + filter bot~~ (`0004`, superseded).
- Reels are kept, expire via media retention (`0005`).
- **Element X over Tailscale is the main app; server name = Tailscale name** (`0006`).

## Open items

- [ ] Remove the stale serve entry for the old machine name (user, sudo): `tailscale serve reset && tailscale serve --bg 8008`.
- [ ] `sudo systemctl enable docker` (and `tailscaled`) on the PC; verify after a reboot.
- [ ] User confirms the cookie message in the bot DM was removed.
- [ ] Ask a friend to send a plain photo, a voice note, a story reply, a sticker -> sample them, fill `message-formats.md`.
- [ ] Decide when to delete `backups/2026-10-28-localhost/` (holds old DMs and the old IG session cookies message).
- [ ] UNVERIFIED: Element X pusher payload format (event-id-only or full content via the push gateway).

## Next step

Open items above. After that, the optional feed bot (`../docs/roadmap.md`, Later): posts "<friend> sent <type>" plus the media into a separate non-bridged room. The user has not yet confirmed they want it in addition to Element X's notifications.
