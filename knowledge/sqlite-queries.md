# Inspecting Synapse data (read-only, privacy-safe)

Always open the DB **read-only** so a running Synapse is never affected, and **never print `m.text` bodies** (they are friends' private messages). Print types and structure.

```python
# run with: python3 -I script.py   (from the project root)
import sqlite3, json
c = sqlite3.connect('file:synapse/homeserver.db?mode=ro', uri=True)
rows = c.execute("""select e.stream_ordering, j.json from events e join event_json j using(event_id)
                    where e.type='m.room.message' order by e.stream_ordering desc limit 60""").fetchall()
for so, js in reversed(rows):
    ct = json.loads(js)['content']
    mt, body = ct.get('msgtype'), ct.get('body') or ''
    shown = f"<text, {len(body)} chars>" if mt == 'm.text' else body[:90]   # do NOT print text bodies
    print(so, mt, shown, sorted(k for k in ct if k not in ('msgtype', 'body')))
```

Careful with `m.text` events whose body is a cURL command or other secret (the Instagram login paste is one): they are `m.text`, so the snippet above does not print them in full. Still, never widen the preview for `m.text`.

## Tables used

| Table | Use |
|---|---|
| `events` | `stream_ordering`, `type`, `room_id`, `sender`, `event_id` |
| `event_json` | full event JSON by `event_id` |
| `local_media_repository` | `media_id`, `media_type`, `media_length` for media uploaded to this server |

## Media id -> file

`mxc://<server-name>/<media_id>` -> `synapse/media_store/local_content/<media_id[0:2]>/<media_id[2:4]>/<media_id[4:]>`. Check `media_type` in `local_media_repository`, and copy to `samples/<name>.mp4` (ignored by git) to play it with `mpv`.
