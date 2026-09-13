# MetaFeeder · YouTube — Rationale

## What deviation / exception is being requested

`metafeederyoutube-feeder` runs `user: 0:0`.

## Why it is necessary

Carried over unchanged from the MetaFeeder listings this app is a sibling of, so the
feeder behaves exactly as they do. The MetaFeeder images write their state and redb
cache as the container user and some reach MetaCore's root-owned tree; a uid-1000
process is refused.

## Security mitigations in place

- Public ingress terminates at the AppShield sidecar (`ghcr.io/yundera/appshield:2.0.9`),
  which gates every path behind the PCS's Authelia SSO over OIDC. The feeder carries no
  Caddy labels and publishes no host port, so it is reachable only from the `pcs` Docker
  network.
- The container mounts **no user directories** — not `/DATA/Documents`, `/DATA/Downloads`,
  `/DATA/Media` or `/DATA/Gallery`. Its only bind is its own state under
  `/DATA/AppData/metafeederyoutube/`, declared in `x-compose-app.folders` and owned `$PUID:$PGID`.
- `cpu_shares` and a hard `mem_limit` (swap disabled) set on every service; image tags
  pinned; no `:latest`.

## Alternatives considered and rejected

- **Run as `$PUID:$PGID`.** Not validated for this image; the sibling MetaFeeder apps all
  run `0:0`, and changing the runtime user here would make this listing the odd one out
  for no measured benefit.

## Data protection

The feeder holds no user media. Its entire state lives under `/DATA/AppData/metafeederyoutube/`, which
is the unit Maison archives on uninstall and restores from backup.
