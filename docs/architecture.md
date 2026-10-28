# Architecture

## The big picture

```
 Friends on Instagram
        ^
        |
        v
 +-------------+      +-----------+      +-------------------+      +------------------+
 |   bridge    | <--> |  Synapse  | <--> |  Tailscale Serve  | <--> | Element X        |
 | mautrix-meta|      |  (Matrix) |      |  HTTPS, tailnet   |      | on iPhone        |
 +-------------+      +-----------+      |  only             |      | (Tailscale on)   |
     keeps your            |   |         +-------------------+      +------------------+
     IG login              |   |                                             ^
                           |   +--> push gateway --> Apple push ------------+
                           +--> Element Web (browser on the PC, optional)

 Fallback: Safari on instagram.com, only for things the bridge cannot do.
```

Everything except the phone runs on the home PC: the three containers in Docker, Tailscale as a system service.

## The parts

**Bridge (`mautrix-meta`, Instagram build).** Logs in to Instagram as you and copies conversations into Matrix rooms, one room per chat. It also sends what you write in Matrix back to Instagram. It is the only component that talks to Instagram.

**Synapse.** The Matrix server. It stores every bridged message, the rooms and your account. Think of it as a private chat server that happens to mirror your Instagram. Its server name is the Tailscale address `<server-name>`, so your Matrix ID is `@admin:<server-name>`.

**Tailscale Serve.** Puts Synapse behind HTTPS with a real certificate, reachable only from your own Tailscale devices. Nothing is opened to the internet. See [decision 0006](decisions/0006-element-x-over-tailscale.md).

**Element X.** The chat app on the iPhone. Reads, replies, plays reels, shows push notifications.

**Element Web.** The same idea in a browser on the PC. Optional. Use Firefox: Chromium cannot seek inside videos (see [findings](#things-we-learned)).

## Why there are databases

A program that restarts forgets everything unless it saves to disk. Two of the parts need to remember things:

- **Synapse** stores all messages, rooms, accounts and read markers. Without that there would be no history.
- **The bridge** stores your Instagram login session and a map "this Instagram chat is that Matrix room, this message is that event". Without it, a restart would lose the session and could duplicate messages.

Both databases are SQLite files, which is fine for one user. See [decision 0002](decisions/0002-sqlite-first.md).

## How a message flows

1. A friend sends you a text or a reel on Instagram.
2. The bridge receives it and writes it into the matching Matrix room. A reel arrives as a real video file stored in Synapse.
3. Synapse asks the push gateway to wake up Element X; you get a notification.
4. You open the chat in Element X (Tailscale must be on) and reply.
5. The bridge sends your reply to Instagram as you.

## Things we learned

- How each Instagram thing arrives in Matrix (reels as video events with the original link attached, posts as image events, text as text) is documented in `knowledge/message-formats.md`.
- The bridge already downloads reels into Synapse as real mp4 files. No scraping is needed to get them. They expire 30 days after last viewing ([decision 0005](decisions/0005-keep-reels-with-retention.md)).
- Synapse does not support HTTP range requests on media, so Chromium cannot jump around in a video. Firefox can.
- The Matrix server name is permanent. Changing it means a fresh Synapse and a fresh bridge login.

## Where the data lives

All under the project folder and all git-ignored: `synapse/` (database, media, keys), `bridge/` (config, Instagram session), `backups/` (the old `localhost` setup). Treat them as private. See [security](security.md).
