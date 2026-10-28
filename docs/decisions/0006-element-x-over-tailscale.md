# 0006. Element X over Tailscale is the main app; server name is the Tailscale name

Date: 2026-10-28 - Status: accepted. Supersedes [0004](0004-ntfy-over-element-x.md).

## Context

[0004](0004-ntfy-over-element-x.md) assumed Element X needs a public HTTPS server, and planned a filter bot plus ntfy, with replies by hand in Safari. Two things changed:

- The user wants to read **and reply** inside one Matrix app on the iPhone, and is fine with seeing reels. Safari on instagram.com is only a fallback for things the bridge cannot do.
- Tailscale Serve gives Synapse a private HTTPS address (`https://<machine>.<tailnet>.ts.net`) with a publicly trusted Let's Encrypt certificate, reachable only from devices in the user's tailnet. No ports are opened at home, which keeps [0003](0003-host-at-home.md) intact.

Synapse 1.164 supports native sliding sync (`org.matrix.simplified_msc3575`) and password login, which is what Element X needs.

The Matrix server name cannot change once Synapse's database exists. The first setup used `localhost`, so IDs looked like `@admin:localhost` and the phone had to be told the server URL by hand.

## Decision

- **Element X on the iPhone is the main client** for reading and replying. Its own push notifications replace ntfy.
- Synapse is exposed only through `tailscale serve --bg 8008` on the home PC, as `https://<server-name>`.
- Synapse was **recreated** with that name as `server_name`, plus `public_baseurl` and `serve_client_wellknown: true`, so Element X discovers the server from the name alone. The old `localhost` data is kept in `backups/2026-10-28-localhost/` (git-ignored).
- The bridge was recreated for the new domain and logged in again with cookies from an already logged-in browser session (reuses that session; no new Instagram login).

## Consequences

- One app for everything; the planned filter bot is no longer needed for notifications. A smaller "feed" bot that posts "<friend> sent a reel" into a **separate, non-bridged** room is still an option (see [roadmap](../roadmap.md)).
- The phone must have Tailscale switched on to load chats. Push still arrives without it (it goes through Element's push gateway and Apple).
- The machine name `<machine>` and tailnet name are in public Certificate Transparency logs. Do not put anything sensitive into Tailscale machine names. (The earlier name may already be logged too.) The repo uses the placeholder `<server-name>`; the real value stays in git-ignored files.
- Recreating Synapse invalidated every old access token, including the one exposed on 2026-10-09 ([security](../security.md)).
- Matrix history from before 2026-10-28 is only in the backup. The bridge backfills recent messages into new rooms.
- Synapse now makes outbound requests to the push gateway when you get a message. UNVERIFIED whether Element X registers its pusher with event-id-only payloads (message text not sent through the gateway).
