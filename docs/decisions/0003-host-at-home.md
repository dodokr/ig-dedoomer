# 0003. Host on the home PC, not on a VPS

Date: 2026-10-28 - Status: accepted

## Context

The bridge keeps a permanent connection to Instagram from wherever it runs. Instagram looks at the IP type, sudden location or IP changes, and behaviour. Datacenter ranges (Hetzner and the like) and shared VPN ranges are treated as more suspicious than residential connections, and "confirm it's you" challenges or locks are the usual consequence. Nobody can promise a safe setup: running the bridge breaks Instagram's terms of service on any host.

Options compared:

| Option | Account risk | Cost | Notes |
|---|---|---|---|
| Home PC (residential IP) | lowest | none | depends on home power and internet |
| VPS + static ISP/residential proxy (via the bridge's `network.proxy`) | medium | VPS + proxy | extra moving part |
| VPS directly (e.g. Hetzner) | highest | low | likely to hit a verification once |
| VPS + consumer VPN (e.g. NordVPN) | high | low | shared, often already-flagged IPs; rotation looks like new logins |

## Decision

Run everything on the user's always-on home PC, so Instagram sees the same residential IP the account normally uses. No ports are opened at home.

## Consequences

- Remote access goes through Tailscale. The phone uses Element X against Synapse's Tailscale HTTPS address; see [0006](0006-element-x-over-tailscale.md).
- The PC must stay on, and Docker must start at boot. If the home IP changes (ISP), expect Instagram to notice; keep it stable where possible.
- Keep activity low: read-mostly, no extra scrapers, no repeated logins.
- If the PC cannot be always on later, fall back to a VPS plus a static residential proxy, not a plain VPS or a VPN.
