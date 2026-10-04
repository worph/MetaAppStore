# MetaFeeder · Usenet — Rationale

## What deviation / exception is being requested

1. `metafeederusenet-feeder` runs `user: 0:0`.

(The NNTmux indexer this feeder used to bundle — and its own exceptions: a web UI on its own
login, seeded TMDB/TVDB keys — is now the separate **NNTmux** app in worph/AppStore, with its
own `rationale.md`.)

## Why it is necessary

1. The feeder publishes resolved records to the shared MetaCore and writes plugin artwork
   through its WebDAV endpoint; those targets are created and owned by MetaCore's
   root-running container.

## Security mitigations in place

- No web UI and no Caddy route: the feeder's configuration page is reached only through the
  MetaGateway dashboard's proxy, behind the gateway's login.
- It talks to NNTmux only over NNTmux's Newznab API, with the key of a dedicated
  `metamesh` user — it holds no database credential and mounts no NNTmux storage.
- The API key and the indexer keys are secrets on the feeder's config page (write-only,
  never echoed back); none has an env seed or a default.
- `cpu_shares` and memory caps on all three services; every image pinned.

## Alternatives considered and rejected

- **Run the feeder as `$PUID:$PGID`.** Writes into MetaCore's root-owned tree fail.

## Data protection

All state — the feeder config, its redb cache, the per-release `.nzb` cache, the TMDB
cache — is under `/DATA/AppData/metafeederusenet/`. The `.nzb` cache is re-derivable from
NNTmux. No user media directory is mounted.
