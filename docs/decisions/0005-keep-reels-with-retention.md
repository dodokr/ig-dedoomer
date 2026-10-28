# 0005. Keep reels in Synapse, expire them with media retention

Date: 2026-10-28 - Status: accepted, applied 2026-10-28 (`local_media_lifetime: 30d`)

## Context

The bridge downloads shared reels (about 8 MB each) and photos into Synapse's media store. By default Synapse keeps all media forever. At 20 reels a day that is roughly 160 MB a day, about 58 GB a year, which would fill a home PC's disk over time.

We looked at downloading reels separately with `yt-dlp` and decided against it: the bridge already stored the real mp4, and an extra scraper adds account risk for no gain.

## Decision

Keep reels where the bridge puts them, and add a Synapse `media_retention` setting at deployment, for example:

```yaml
media_retention:
  local_media_lifetime: 30d
```

Lifetime counts from the last access, or from creation if the file was never opened. Thumbnails go with the original. Do not enable message `retention`; text is tiny.

## Consequences

- Disk use is bounded (about 5 GB at the example rates).
- After expiry the message stays in the room but its video no longer loads; the original is still reachable through the `external_url` link to Instagram.
- A reel you watched recently stays; one nobody opened expires.
- Protected or quarantined media is exempt from purging.
- Applied on the home PC when Synapse was recreated ([0006](0006-element-x-over-tailscale.md)); also in `config-examples/`.
