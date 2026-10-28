# Stack facts

## Services (`docker-compose.yml`)

| Service | Image | Ports | Notes |
|---|---|---|---|
| `synapse` | `matrixdotorg/synapse:latest` (1.162.0 when written) | `127.0.0.1:8008` | healthcheck on `/health` |
| `bridge` | `dock.mau.dev/mautrix/meta:ig-latest` | internal `29330` only | binary `/usr/bin/mautrix-meta` reports itself as `mautrix-instagram v26.09` |
| `element` | `vectorim/element-web:latest` | `127.0.0.1:8080` | config from `element/config.json` |

- `synapse` and `bridge` run as `${LOCAL_UID}:${LOCAL_GID}` from `.env` (copy `.env.example`). Otherwise they cannot read/write the user-owned files.
- `bridge` waits for `synapse` to be healthy.
- Everything binds to `127.0.0.1`. Nothing is exposed.
- **Not present yet:** `ntfy`, filter bot.

## Identities

| Thing | Value |
|---|---|
| Matrix server name | `localhost` |
| User | `@admin:localhost` (bridge admin permission) |
| Bridge bot | `@instagrambot:localhost` |
| Instagram ghosts | `@instagram_<igid>:localhost` |
| Bridge command prefix | `!ig` (not needed in the management room) |

## Storage

| What | Path (relative to project root) |
|---|---|
| Synapse DB (SQLite) | `synapse/homeserver.db` |
| Bridge DB (SQLite; IG session lives here) | `bridge/mautrix-meta.db` |
| Media files | `synapse/media_store/local_content/<id[0:2]>/<id[2:4]>/<id[4:]>` (no extension) |
| Appservice registration (source) | `bridge/registration.yaml` |
| Appservice registration (Synapse's copy) | `synapse/registration-instagram.yaml` (mode 644) |
| Old Messenger-bridge leftovers | `bridge/old-facebook/` (empty, safe to delete) |

After regenerating `bridge/registration.yaml`: copy it to `synapse/registration-instagram.yaml`, `chmod 644`, restart Synapse and the bridge.

## Config deltas from defaults

Synapse (`synapse/homeserver.yaml`):
- `enable_registration: false`
- `federation_domain_whitelist: []` (no federation)
- `app_service_config_files: [/data/registration-instagram.yaml]`

Bridge (`bridge/config.yaml`):
- `database.type: sqlite3-fk-wal`, `uri: file:/data/mautrix-meta.db?_txlock=immediate`
- `homeserver.address: http://synapse:8008`, `domain: localhost`
- `appservice.address: http://bridge:29330`, `hostname: 0.0.0.0`
- `bridge.permissions`: `localhost: user`, `@admin:localhost: admin` (no `*: relay`)

Instagram build defaults worth knowing (unchanged): `network.thread_backfill.batch_count: 0` (no history backfill), `network.disable_xma_backfill: true`, `id: instagram`, `username_template: instagram_{{.}}`.

Sanitized copies: `config-examples/`. Secret keys there are `"CHANGE_ME"`.
