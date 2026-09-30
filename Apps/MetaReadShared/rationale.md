# MetaRead (shared backend) — Rationale

## What deviation / exception is being requested

1. `metaread-app` is **publicly routed with no AppShield sidecar**. It carries the
   `caddy_*` labels itself.
2. `metaread-search` runs `user: 0:0` and publishes host port `4006/tcp` (libp2p).
3. The compose top-level `name:` is `metaread`, the same project id as the self-contained
   `MetaRead` app in this store.

## Why it is necessary

1. meta-read has its **own** authentication: a secp256k1 keypair proven by signing a
   challenge, with invite-only signup. Its router gates deny-by-default: a route not in
   `open_routes()` answers 401. The open set is `/health`, `/api/config`, the
   challenge/token pair, `/api/invite/check/:token` and `/api/image/:cid`. That is the "app's
   own built-in auth" alternative the security checklist accepts. AppShield in front added a
   second login and an OIDC redirect on every cover fetch, because `<img src>` cannot carry
   an Authorization header. It also left the service worker's offline cache at the mercy of
   a session cookie.
2. libp2p is not HTTP. Caddy cannot proxy it, and remote peers dial the advertised
   multiaddr directly, so the port must be published and reachable inbound. The app carries
   the reserved `needs-public-ip` tag.
3. The two MetaRead variants are alternatives, not companions. They share the project id
   and the same `/DATA/AppData/metaread/` paths, so installing one over the other reuses the
   same data. Only one may be installed at a time.

## Security mitigations in place

- meta-read's signature-challenge login is on by default, and signup is invite-only, so a
  stranger reaching the public hostname cannot create an account.
- The `metaread-search` dashboard *is* behind an AppShield sidecar
  (`ghcr.io/yundera/appshield:2.0.9`) with Authelia SSO; only the reader itself uses its own
  gate.
- `metaread-app` runs unprivileged as `1000:1000` with a 512 MB hard memory cap and swap
  disabled.
- The only published port is raw libp2p TCP; no HTTP port is published to the host.
- No user directories are mounted: no `/DATA/Documents`, `/DATA/Downloads`, `/DATA/Media`
  or `/DATA/Gallery`.
- `cpu_shares` on every service; image tags pinned.

## Alternatives considered and rejected

- **AppShield in front of `metaread-app`.** Two identities for one user, plus the cover and
  service-worker costs above.
- **Bundle a private MetaCore/MetaShare.** That is the self-contained `MetaRead` app. On a
  box that already runs them, it doubles the memory footprint. It also gives the reader an
  identity, a User Data Layer and a catalogue separate from MetaWatch's on the same box.
- **A distinct project id so both variants coexist.** They would fight over the same public
  hostnames, the same `/DATA/AppData/metaread/` paths and host port 4006.

## Data protection

My List, Continue Reading, the session key and the invite book persist in
`/DATA/AppData/metaread/data`. The recovered cover blockstore and language prefs persist in
`/DATA/AppData/metaread/search-data`. Both are declared in `x-compose-app.folders` and
archived by Maison on uninstall. Because both MetaRead variants use those paths, switching
between them keeps the user's data. The MetaCore and MetaShare state this variant reads
belongs to those apps and is archived with them.
