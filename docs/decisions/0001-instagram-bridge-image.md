# 0001. Use the Instagram build of mautrix-meta (`ig-latest`)

Date: 2026-10-09 - Status: accepted

## Context

We want Instagram DMs inside Matrix. Options considered: scraping the web inbox with a headless browser, an unofficial API library, or an existing Matrix bridge.

The repository `mautrix/meta` contains **two bridges**: one for Facebook Messenger and one for Instagram DMs. They are published as different Docker tags of the same image name. We first pulled `latest`, which is the Messenger one; the login only offered Messenger flows and the startup log said `mautrix-facebook`.

## Decision

Use the maintained bridge, with the Instagram image `dock.mau.dev/mautrix/meta:ig-latest`. Do not scrape.

## Consequences

- We get message history in a real chat database, reel and photo downloads, double puppeting and typing/read state for free, and the bridge is updated when Instagram changes.
- Risk is the same family as scraping: the bridge uses an unofficial API against Instagram's terms of service, so the account can be challenged or locked.
- The image name is easy to get wrong. `dock.mau.dev/mautrix/instagram` does not exist; `latest` is Messenger. This is recorded in `knowledge/bridge-gotchas.md`.
- Login modes are chosen at login time (`instagram` cookies, `android`, `instagram-password`), not in the config.
