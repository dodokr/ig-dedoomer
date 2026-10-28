# Message formats (contract for the filter bot)

Observed in Synapse on 2026-10-09 from real bridged DMs. Event type is always `m.room.message`; discriminate on `content.msgtype`.

| Instagram thing | `msgtype` | Identifying fields | Notes |
|---|---|---|---|
| Plain text | `m.text` | `body` is the text; may have `format`/`formatted_body` | The only type that should show text in a push. |
| Shared **reel** | `m.video` | `external_url` contains `instagram.com/reel/`; `body` is a markdown link to it; `url` is an `mxc://` of the real mp4; `filename` ends `.mp4` | Video is already downloaded into Synapse (about 8 MB, 720x1280, H.264/AAC, ~31 s sample). |
| Shared **post** | `m.image` | `external_url` contains `instagram.com/p/`; `body` is a markdown link; `url` is a JPEG `mxc://` | Thumbnail image of the post. |
| Bridge/command output | `m.notice` | sent by the bridge bot | Ignore. |

Extra keys seen on bridged events: `com.beeper.disappearing_timer`, `fi.mau.double_puppet_source` (the event was sent by the user's own Instagram account via double puppeting), `m.mentions`, `info` (`mimetype`, `size`, `w`, `h`).

## Planned detection rule

```
shared_media = msgtype in {m.image, m.video} and re.search(r"instagram\.com/(reel|p|tv)/", content.get("external_url", ""))
```

- `shared_media` -> placeholder ("<sender> sent a reel/post"), no link, no preview.
- `m.text` -> push with sender + text.
- Ignore events whose sender is the user, `@instagrambot:localhost`, or the filter bot itself.
- Anything else -> generic placeholder.

## UNVERIFIED / unsampled

Plain photos sent by a friend, voice notes, stickers, story replies, GIFs, reactions, edits, deletions, group chats, disappearing/view-once messages. Sample each (types only, not content), then fill this table. Do not assume a real photo lacks `external_url`; check.

## Media access

- File on disk: see `stack.md` (Storage).
- HTTP: `GET /_matrix/client/v1/media/download/<server>/<media_id>` with `Authorization: Bearer <token>`. The old `/_matrix/media/v3/download` endpoint returns 404 (disabled).
- Synapse ignores `Range` headers on media (returns `200` with the full file). Chromium cannot seek in such videos; Firefox can.
