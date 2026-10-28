# Commands

Run from `~/Projects/side-projects/ig-dm-filter`. Shell is fish. Use `docker-compose` (standalone), not `docker compose`.

## Docker

The agent has no `sudo`. Ask the user:

```
! sudo systemctl start docker
```

The user must be in the `docker` group, and Claude Code must have been started from a session that has that group (`id -nG` must list `docker`). Group changes need a fresh login session.

## Stack

```
docker-compose up -d                       # start
docker-compose ps --format '{{.Service}} {{.Status}}'
docker-compose logs --tail=40 bridge       # strip colors: | sed 's/\x1b\[[0-9;]*m//g'
docker-compose up -d --force-recreate synapse bridge   # after config or registration changes
docker-compose stop bridge
```

## Matrix user (user runs; it prompts for a password)

```
docker-compose exec synapse register_new_matrix_user -c /data/homeserver.yaml -u admin -a http://localhost:8008
```

Username must be `admin` (bridge grants admin to `@admin:localhost`). Element Web: http://localhost:8080. Use **Firefox** (Chromium cannot seek in videos, see `bridge-gotchas.md`).

## Instagram login (the user does this in Element, never in chat)

1. DM `@instagrambot:localhost`, send `login`. The bot lists the flows: `instagram` (cookies from instagram.com), `android` (username/password via Android API), `instagram-password` (username/password via web API).
2. The user used `login instagram`: in a private window log in on instagram.com, DevTools -> Network -> right-click a request to www.instagram.com -> Copy -> **Copy as cURL (bash)**, paste into the bot DM. Do not log out of that window (it kills the session). Then redact the message in Element.
3. Double puppeting (so the bridge joins rooms as the user, no invites): `login-matrix <access token>` in the bot DM. Prefer a token from the Matrix login API over the Element session token.

## Regenerate bridge config / registration

```
docker run --rm --user $(id -u):$(id -g) -v $PWD/bridge:/data dock.mau.dev/mautrix/meta:ig-latest /usr/bin/mautrix-meta -c /data/config.yaml -e            # default config
docker run --rm --user $(id -u):$(id -g) -v $PWD/bridge:/data dock.mau.dev/mautrix/meta:ig-latest /usr/bin/mautrix-meta -c /data/config.yaml -r /data/registration.yaml -g   # registration
```

## Fix root-owned files created by a container

```
docker run --rm -v $PWD:/work alpine chown -R $(id -u):$(id -g) /work/bridge /work/synapse
```
