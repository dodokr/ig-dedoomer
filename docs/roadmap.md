# Roadmap

Status as of 2026-10-28.

## Done

- [x] **Phase 0: Setup.** Project folder, compose file, `.gitignore`.
- [x] **Phase 1: Core stack.** Synapse, the bridge and Element Web run in Docker, bound to localhost.
- [x] **Phase 2: Connect Instagram.** Logged in with cookies; double puppeting enabled.
- [x] **Phase 3: Discovery.** Reels, posts and text mapped to Matrix event types. Reel mp4s confirmed to be stored in Synapse.
- [x] **Docs and knowledge base** (this folder, `knowledge/`, `AGENTS.md`).
- [x] **Home deployment.** Moved to the home PC; Tailscale Serve gives Synapse private HTTPS; Synapse recreated with the Tailscale name as server name; media retention 30 days; bridge backfill on.
- [x] **Element X on the iPhone** for reading and replying ([decision 0006](decisions/0006-element-x-over-tailscale.md)).

## Next

- [ ] **Docker at boot** on the PC (`sudo systemctl enable docker`), and check the containers come back after a reboot.
- [ ] **Remove the stale Tailscale Serve entry** for the old machine name (`tailscale serve reset`, then `tailscale serve --bg 8008`).
- [ ] **Sample the remaining message types** (photo, voice note, story reply, sticker). Check how each one looks in Element X.
- [ ] **Backups**: encrypted copy of `synapse/` and `bridge/`, kept on the PC or an encrypted drive.

## Later

- [ ] **Feed bot (optional).** Posts "<friend> sent a reel" plus the same video into a **separate, non-bridged** room, reusing the stored file. Never posts into bridged rooms. Only if Element X's own notifications are not enough.
- [ ] **Reel processing.** Pull frames and a transcript from the stored mp4 and summarize. Claude API versus local models is still open; this would send friends' content outside the machine, so it needs an explicit decision.
- [ ] **Digest or event extraction** ("things happening this week").

## Not planned

- ntfy and the filter bot from [decision 0004](decisions/0004-ntfy-over-element-x.md). Replaced by Element X.
- A Hetzner or other VPS. Datacenter IPs raise the chance Instagram challenges the account.
- Running the stack on the laptop again. Two bridges on one Instagram session look suspicious.
