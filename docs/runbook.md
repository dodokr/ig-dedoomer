# Runbook

Run commands from the project root. The shell here is fish.

## After a reboot

1. Start Docker (needs `sudo`): `sudo systemctl start docker`
2. `docker-compose up -d`
3. Check: `docker-compose ps`. Synapse should be `healthy`.
4. Open http://localhost:8080 in **Firefox** and sign in.

If Docker says `permission denied`, your user is not in the `docker` group, or you need a fresh login after adding it. Check with `id -nG`.

## First-time setup from scratch

1. **Env:** `cp .env.example .env`. Set the values to your own `id -u` and `id -g`.
2. **Synapse config.** Generate it:
   ```
   docker run --rm -v $PWD/synapse:/data -e SYNAPSE_SERVER_NAME=localhost -e SYNAPSE_REPORT_STATS=no matrixdotorg/synapse:latest generate
   ```
   Then apply the changes shown in `config-examples/synapse.homeserver.example.yaml` (registration off, federation off, appservice registration file).
3. **Bridge config.** Generate the default with the Instagram image `dock.mau.dev/mautrix/meta:ig-latest` and compare with `config-examples/bridge.config.example.yaml` for the settings to change (database, addresses, permissions).
4. **Registration.** Generate `bridge/registration.yaml`, copy it to `synapse/registration-instagram.yaml` (mode 644).
5. Fix file ownership if a container created root-owned files. Commands for steps 3 to 5 and the ownership fix are in `knowledge/commands.md`.
6. `docker-compose up -d`
7. **Create your Matrix user** (it asks for a password):
   ```
   docker-compose exec synapse register_new_matrix_user -c /data/homeserver.yaml -u admin -a http://localhost:8008
   ```
   Use the username `admin`.

## Logging in to Instagram

1. In Element, start a DM with `@instagrambot:localhost` and send `login`. It lists three flows: `instagram` (cookies), `android` and `instagram-password`.
2. For the cookie flow send `login instagram`. In a **private browser window**, log in on instagram.com. Open DevTools, go to the Network tab, right-click a request to `www.instagram.com`, choose Copy then **Copy as cURL (bash)**, and paste it into the bot DM.
3. Close the private window **without logging out**. Logging out ends the session and disconnects the bridge.
4. In Element, **remove (redact) the message** you pasted. It contains your session cookies.
5. Your chats appear as rooms. To stop getting invites and have the bridge join automatically, enable *double puppeting*: get an access token for your Matrix user and send `login-matrix <token>` to the bot.

Never paste cookies, cURL commands or tokens anywhere else (chats, issues, notes).

## Common fixes

| Problem | Fix |
|---|---|
| Login only offers Messenger | You are on the wrong image. Use `dock.mau.dev/mautrix/meta:ig-latest`. |
| Bridge disconnected | Check `docker-compose logs --tail=40 bridge`. If the session expired, log in again. |
| Config or registration changed | `docker-compose up -d --force-recreate synapse bridge` |
| Files unreadable or root-owned | `docker run --rm -v $PWD:/work alpine chown -R $(id -u):$(id -g) /work/bridge /work/synapse` |
| Cannot seek in a video | Use Firefox, or download the file. |

## Looking at a reel

Reels are stored as plain mp4 files in `synapse/media_store/`. The path pattern and how to copy one out are in `knowledge/stack.md` and `knowledge/sqlite-queries.md`. Play a copy with `mpv`; it seeks fine.

## Updating

```
docker-compose pull
docker-compose up -d
```

Read the bridge release notes first. The bridge talks to an unofficial API, so updates sometimes matter for staying connected.

## Backups

`synapse/` and `bridge/` contain all your messages and the Instagram session. If you back them up, encrypt the backup and keep it off shared drives. Stop the stack before copying the SQLite files.
