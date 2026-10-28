# Roadmap

Status as of 2026-10-28.

## Done

- [x] **Phase 0: Setup.** Project folder, compose file, `.gitignore`.
- [x] **Phase 1: Core stack.** Synapse, the bridge and Element Web run in Docker, bound to localhost.
- [x] **Phase 2: Connect Instagram.** Logged in with cookies; double puppeting enabled.
- [x] **Phase 3: Discovery.** Reels, posts and text mapped to Matrix event types. Reel mp4s confirmed to be stored in Synapse.
- [x] **Docs and knowledge base** (this folder, `knowledge/`, `AGENTS.md`).

## Next

- [ ] **Phase 4a: Filter bot and ntfy.**
  - Add `ntfy` to the compose file.
  - Python bot (`matrix-nio`): plain text -> push; reel or post -> placeholder; unknown type -> generic placeholder; ignore own and bridge messages.
  - Tests built from real event shapes.
  - Output for the first version: an ntfy push plus a local log file.
- [ ] **Sample the remaining message types** (photo, voice note, story reply, sticker). Ask a friend to send them.
- [ ] **Phase 5: End-to-end local test.** A friend texts you and a push arrives. A friend sends a reel and only a placeholder arrives.

## Later

- [ ] **Phase 4b: Reel processing.** Pull frames and a transcript from the stored mp4 and summarize. Claude API versus local models is still open; this is the only step that sends anything outside the machine, so it needs an explicit decision.
- [ ] **Phase 4c: Digest or event extraction** ("things happening this week").
- [ ] **Home deployment** on your always-on home PC (see [decision 0003](decisions/0003-host-at-home.md)):
  - Docker enabled at boot, fixed LAN address, restart policies checked.
  - Tailscale for reaching it from outside without opening ports.
  - ntfy push to the iPhone via ntfy.sh's upstream relay (to be checked against the ntfy docs).
  - Synapse `media_retention` (for example 30 days) so reels do not fill the disk.
  - Backups, and rotating every secret listed in [security](security.md).
  - "Tap to watch": the bot saves reels to a folder served by a file server that supports range requests, and the ntfy notification links to the mp4 so Safari plays it with seeking.

## Not planned

- Element X on the phone. It needs a public HTTPS server. Revisit only if ntfy plus tap-to-watch is not enough.
- A Hetzner or other VPS. Datacenter IPs raise the chance Instagram challenges the account.
