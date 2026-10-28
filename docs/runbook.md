# Runbook

Run commands on the home PC from the project root (`~/Projects/side-projects/ig-dedoomer`). The shell is fish.

## After a reboot

With Docker and Tailscale enabled at boot, nothing is needed: the containers and `tailscale serve --bg` come back on their own. To check:

1. `docker-compose ps`. Synapse should be `healthy`. If Docker is not running: `sudo systemctl start docker`, then `docker-compose up -d`.
2. `tailscale serve status` should list `https://<server-name>` -> `127.0.0.1:8008`.
3. Open Element X on the phone (Tailscale on). On the PC you can also use http://localhost:8080 in **Firefox**.

If Docker says `permission denied`, your user is not in the `docker` group, or you need a fresh login after adding it. Check with `id -nG`.

## First-time setup from scratch

0. **Tailscale.** Install it, `sudo systemctl enable --now tailscaled`, `sudo tailscale up`. In the Tailscale admin console, give the machine a harmless name (it becomes public, see [security](security.md)) and enable MagicDNS and HTTPS certificates. Then `sudo tailscale serve --bg 8008`. The server name below is the resulting `<machine>.<tailnet>.ts.net`. **Choose it before step 2: it cannot be changed later** without starting over.
1. **Env:** `cp .env.example .env`. Set the values to your own `id -u` and `id -g`.
2. **Synapse config.** Generate it (fish syntax; in bash use `-u $(id -u):$(id -g)`):
   ```
   docker run --rm -u (id -u):(id -g) -v $PWD/synapse:/data -e SYNAPSE_SERVER_NAME=<server-name> -e SYNAPSE_REPORT_STATS=no matrixdotorg/synapse:latest generate
   ```
   Then append the block shown at the end of `config-examples/synapse.homeserver.example.yaml` (public URL, well-known, registration off, federation off, appservice registration file, media retention).
3. **Bridge config.** Generate the default with the Instagram image `dock.mau.dev/mautrix/meta:ig-latest` and compare with `config-examples/bridge.config.example.yaml` for the settings to change (database, addresses, domain, permissions, backfill).
4. **Registration.** Generate `bridge/registration.yaml`, copy it to `synapse/registration-instagram.yaml` (mode 644).
5. Fix file ownership if a container created root-owned files. Commands for steps 3 to 5 and the ownership fix are in `knowledge/commands.md`.
6. `docker-compose up -d`
7. **Create your Matrix user** (it asks for a password):
   ```
   docker-compose exec synapse register_new_matrix_user -c /data/homeserver.yaml -u admin -a http://localhost:8008
   ```
   Use the username `admin`.

## Connecting the iPhone

1. Install **Tailscale** from the App Store, sign in to the same account and switch it on. It can stay on.
2. Install **Element X**, tap **Sign in manually** (may be labelled "Change account provider").
3. Enter `<server-name>`, then username `admin` and your password.
4. Allow notifications.

If Tailscale is off, notifications still arrive but chats do not load.

## Logging in to Instagram

1. In Element X or Element Web, start a chat with `@instagrambot:<server-name>` and send `login`. It lists three flows: `instagram` (cookies), `android` and `instagram-password`.
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
| Cannot seek in a video (Element Web) | Use Firefox, or download the file. |
| Element X cannot reach the server | Tailscale on the phone switched on? On the PC: `tailscale serve status`, `docker-compose ps`. |
| Old machine name still in `tailscale serve status` | `sudo tailscale serve reset`, then `sudo tailscale serve --bg 8008`. |

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

`backups/2026-10-28-localhost/` is the old setup from before the server was renamed. It is not needed to run anything; delete it once you are sure you do not want the old Matrix history.
