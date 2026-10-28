# Stack facts

Host: home PC, `ssh <pc>` (alias in `local.md`), project `~/Projects/side-projects/ig-dedoomer`.

## Services (`docker-compose.yml`)

| Service | Image | Ports | Notes |
|---|---|---|---|
| `synapse` | `matrixdotorg/synapse:latest` (1.164.0 at last check) | `127.0.0.1:8008` | healthcheck on `/health` |
| `bridge` | `dock.mau.dev/mautrix/meta:ig-latest` | internal `29330` only | binary `/usr/bin/mautrix-meta` reports itself as `mautrix-instagram v26.09` |
| `element` | `vectorim/element-web:latest` | `127.0.0.1:8080` | config from `element/config.json` |

- `synapse` and `bridge` run as `${LOCAL_UID}:${LOCAL_GID}` from `.env` (copy `.env.example`). Otherwise they cannot read/write the user-owned files.
- `bridge` waits for `synapse` to be healthy. All services `restart: unless-stopped`.
- Everything binds to `127.0.0.1`. The only way in from outside the PC is Tailscale Serve (below).

## Tailscale

`<server-name>` in all tracked files = the PC's Tailscale name, `<machine>.<tailnet>.ts.net`. The real value is in `local.md` (git-ignored, this folder). Never write it into a tracked file.

| Thing | Value |
|---|---|
| Machine name / MagicDNS | `<server-name>` |
| Serve | `tailscale serve --bg 8008` -> `https://<server-name>` -> `http://127.0.0.1:8008`, tailnet only |
| Certificate | Let's Encrypt via Tailscale; name is in public CT logs |
| Stale entry | old machine name still listed until `tailscale serve reset` (see `local.md`) |

## Identities

| Thing | Value |
|---|---|
| Matrix server name | `<server-name>` (permanent for this DB) |
| User | `@admin:<server-name>` (bridge admin permission) |
| Bridge bot | `@instagrambot:<server-name>` |
| Instagram ghosts | `@instagram_<igid>:<server-name>` |
| Bridge command prefix | `!ig` (not needed in the management room) |

## Storage

| What | Path (relative to project root) |
|---|---|
| Synapse DB (SQLite) | `synapse/homeserver.db` |
| Bridge DB (SQLite; IG session lives here) | `bridge/mautrix-meta.db` |
| Media files | `synapse/media_store/local_content/<id[0:2]>/<id[2:4]>/<id[4:]>` (no extension) |
| Appservice registration (source) | `bridge/registration.yaml` |
| Appservice registration (Synapse's copy) | `synapse/registration-instagram.yaml` (mode 644) |
| Old `localhost` setup (Synapse, bridge, Element config) | `backups/2026-10-28-localhost/` |

After regenerating `bridge/registration.yaml`: copy it to `synapse/registration-instagram.yaml`, `chmod 644`, restart Synapse and the bridge.

## Config deltas from defaults

Synapse (`synapse/homeserver.yaml`, appended block):
- `public_baseurl: "https://<server-name>/"`
- `serve_client_wellknown: true`
- `enable_registration: false`
- `federation_domain_whitelist: []` (no federation)
- `app_service_config_files: [/data/registration-instagram.yaml]`
- `media_retention.local_media_lifetime: 30d`

Bridge (`bridge/config.yaml`):
- `database.type: sqlite3-fk-wal`, `uri: file:/data/mautrix-meta.db?_txlock=immediate`
- `homeserver.address: http://synapse:8008`, `domain: <server-name>`
- `appservice.address: http://bridge:29330`, `hostname: 0.0.0.0`
- `backfill.enabled: true` (2026-10-28): new rooms get up to 50 recent messages, up to 500 missed messages caught up after restart. Existing rooms get no older history (standard Synapse cannot insert past events). `network.thread_backfill.batch_count` stays `0`.
- `bridge.permissions`: `<server-name>: user`, `@admin:<server-name>: admin` (no `*: relay`)

Instagram build defaults worth knowing (unchanged): `network.thread_backfill.batch_count: 0` (no history backfill), `network.disable_xma_backfill: true`, `id: instagram`, `username_template: instagram_{{.}}`.

Element Web (`element/config.json`, git-ignored; template `element/config.example.json`): `base_url: https://<server-name>`.

Sanitized copies: `config-examples/`. Secret keys there are `"CHANGE_ME"`.
