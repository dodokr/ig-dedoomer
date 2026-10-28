# Architecture

## The big picture

```
 Friends on Instagram
        |
        v
 Instagram  <-------------------------------+
        |                                    |  (you reply here, by hand,
        v                                    |   in Safari on instagram.com)
 +-------------+      +-----------+      +--------------+      +-------+
 |   bridge    | ---> |  Synapse  | ---> |  filter bot  | ---> | ntfy  | ---> iPhone
 | mautrix-meta|      |  (Matrix) |      |  (planned)   |      | push  |
 +-------------+      +-----------+      +--------------+      +-------+
     keeps your            |
     IG login              +--> Element Web (optional viewer in the browser)
```

Everything left of the iPhone runs in Docker on one machine (your home PC when deployed).

## The parts

**Bridge (`mautrix-meta`, Instagram build).** Logs in to Instagram as you and copies conversations into Matrix rooms, one room per chat. It also copies what you send from Matrix back to Instagram. It is the only component that talks to Instagram.

**Synapse.** The Matrix server. It stores every bridged message, the rooms and your account. Think of it as a private chat server that happens to mirror your Instagram.

**Element Web.** A viewer for Synapse, in the browser. Optional: the filter bot does not need it. Use Firefox: Chromium cannot seek inside videos (see [findings](#things-we-learned)).

**Filter bot (planned).** A small Python program that logs in to Synapse as its own user, watches the bridged rooms, and decides what is worth a notification.

**ntfy (planned).** A tiny push-notification server. The filter bot posts to it, and the ntfy app on your iPhone shows the notification.

## Why there are databases

A program that restarts forgets everything unless it saves to disk. Two of the parts need to remember things:

- **Synapse** stores all messages, rooms, accounts and read markers. Without that there would be no history.
- **The bridge** stores your Instagram login session and a map "this Instagram chat is that Matrix room, this message is that event". Without it, a restart would lose the session and could duplicate messages.

The filter bot needs no database. Both databases are SQLite files for now, which is fine for one user. See [decision 0002](decisions/0002-sqlite-first.md).

## How a message flows

1. A friend sends you a text on Instagram.
2. The bridge receives it and writes it into the matching Matrix room.
3. The filter bot sees the new event.
4. If it is plain text, you get a push with the sender and text. If it is a reel or post, you get a placeholder. Anything else gets a generic notice.
5. You read the notification. If you want to answer, you open Instagram in Safari.

## Things we learned

- How each Instagram thing arrives in Matrix (reels as video events with the original link attached, posts as image events, text as text) is documented for the bot in `knowledge/message-formats.md`.
- The bridge already downloads reels into Synapse as real mp4 files. No scraping is needed to get them.
- Synapse does not support HTTP range requests on media, so Chromium cannot jump around in a video. Firefox can.

## Where the data lives

All under the project folder and all git-ignored: `synapse/` (database, media, keys), `bridge/` (config, Instagram session). Treat both as private. See [security](security.md).
