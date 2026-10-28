# 0004. Notify through a filter bot and ntfy; Element X is optional

Date: 2026-10-09 - Status: **superseded** by [0006](0006-element-x-over-tailscale.md) on 2026-10-28. Kept for history; the assumption that Element X needs a public HTTPS server turned out wrong (Tailscale Serve provides private HTTPS).

## Context

Element X on iOS can show the bridged chats, but it cannot filter by message type: a reel arrives as an ordinary message, and Element X would notify for it. Matrix push rules can match on fields such as `msgtype`, but we do not control how every Instagram item is represented, and Element X also needs a Synapse reachable over public HTTPS, which conflicts with hosting at home ([0003](0003-host-at-home.md)).

## Decision

Build a small filter bot that reads the bridged rooms and sends only chosen messages to **ntfy**, a lightweight push service with an iOS app. The user replies by hand in Safari on Instagram web.

## Consequences

- Full control over what counts as noteworthy; the same bot can later add summaries or an event digest.
- The phone never needs access to Synapse. Notifications are the only thing that reaches it.
- Replying is still a manual step in Safari. That fits the goal (open Instagram only when you want to write).
- Element Web stays available in a desktop browser for reading whole chats (use Firefox).
- One more component to run and test.
