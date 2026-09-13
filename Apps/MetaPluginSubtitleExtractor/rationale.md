# MetaPlugin · Subtitle Extractor — Rationale

## What deviation / exception is being requested

`metapluginsubtitleextractor-plugin` runs `user: 0:0`.

## Why it is necessary

The `metamesh-plugin-*` images declare no `USER` and have always run as root, both
when MetaSort spawns them and when a MetaFeeder bundles them as an enrichment
sidecar. This listing is a packaging change — the same image, started by Maison
instead of by MetaSort — so it keeps the runtime user the image was built and
validated with.

## Security mitigations in place

- Public ingress terminates at the AppShield sidecar (`ghcr.io/yundera/appshield:2.0.9`),
  which gates every path behind the PCS's Authelia SSO over OIDC. The plugin carries no
  Caddy labels and publishes no host port, so `/process` and `/configure` are reachable
  only from the `pcs` Docker network.
- The container mounts **no user directories** — not `/DATA/Documents`, `/DATA/Downloads`,
  `/DATA/Media` or `/DATA/Gallery`.
  Its only bind is its own cache under `/DATA/AppData/metapluginsubtitleextractor/`, declared in
  `x-compose-app.folders` and owned `$PUID:$PGID`. It reads the files it works on over
  MetaCore's WebDAV, never from a bind mount.
- It holds **no credentials of its own**: keys arrive per-caller over `/configure` and
  are never written to this app's storage.
- `cpu_shares` and a hard `mem_limit` (swap disabled) set on every service; image tags
  pinned; no `:latest`.

## Alternatives considered and rejected

- **Run as `$PUID:$PGID`.** Not validated for these images; a uid change belongs in the
  plugin image, not in a store listing that is otherwise a pure repackaging.
- **No AppShield gate, no public route at all.** Rejected: the tile would have no way to
  show whether the plugin is alive, and `/manifest` is the only status surface a headless
  plugin has.
- **Expose it unauthenticated.** Rejected outright — `/process` takes URLs and does work
  on request, and `/configure` accepts credentials.

## Data protection

The plugin holds no user media and no user account. Its only state is a rebuildable cache under
`/DATA/AppData/metapluginsubtitleextractor/`, which is the unit Maison archives on uninstall.
