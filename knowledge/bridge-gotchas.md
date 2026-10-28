# Gotchas (things that already cost time)

| Symptom | Cause | Fix |
|---|---|---|
| `login` only offers Messenger flows; bot says "Facebook Messenger bridge" | Image tag `mautrix/meta:latest` is the **Messenger** bridge | Use `dock.mau.dev/mautrix/meta:ig-latest`. `dock.mau.dev/mautrix/instagram` does not exist. Regenerate config and registration with the same image. |
| Bridge starts but the log says `mautrix-facebook` | Same as above | Same |
| `docker compose` -> "unknown command" | Compose plugin not installed | Use standalone `docker-compose` (v5.x installed) |
| `failed to connect to the docker API ... no such file` | Docker daemon not running (e.g. after reboot) | User runs `! sudo systemctl start docker` |
| `permission denied ... docker.sock` | User not in `docker` group, or the session predates the group change | Add to group, then log out/in and restart Claude Code. Verify with `id -nG`. |
| Files in `bridge/` / `synapse/` unreadable | Created by a root container | `chown` via a throwaway alpine container (`commands.md`) |
| Synapse cannot read registration or write DB | Synapse image defaults to UID 991 | `user: "${LOCAL_UID}:${LOCAL_GID}"` in compose |
| Bot avatar download returns 403 in bridge log | Federation is off; avatar is on maunium.net | Cosmetic, ignore |
| Many invites from the bridge after login | No double puppeting yet | `login-matrix <token>` (`commands.md`) |
| Element (Chromium) cannot seek in a reel | Synapse ignores HTTP `Range` on media | Use Firefox, or download the file. Verified by test on 2026-10-09 (full 200 response to a range request). |
| Unauthenticated media URL 404 | Authenticated media is required | Use `/_matrix/client/v1/media/download/...` with a bearer token |

## Bridge facts

- The Instagram login mode is chosen at login time (`login instagram` | `android` | `instagram-password`); there is no `network.mode` in this version's config.
- Bridge secrets are auto-generated into `bridge/config.yaml` (`as_token`, `hs_token`, provisioning `shared_secret`, media `signing_key`, `pickle_key`). Never copy that file raw; use `config-examples/`.
- Instagram traffic can be proxied via `network.proxy` (HTTP/SOCKS5). Not used. NordVPN-style shared IPs are a poor choice (`../docs/decisions/0003-host-at-home.md`).
- Account-flag risk is inherent: running the bridge violates Instagram ToS. Keep activity low and the IP stable.
