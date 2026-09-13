# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**MetaAppStore** is the CasaOS / Yundera 3rd-party app store for the **MetaMesh**
application family only. It was split out of `Worph/AppStore` — the
general-purpose fork — so the MetaMesh apps release on their own cadence. Apps
that are *not* MetaMesh (Jellyfin, Nextcloud, MetaMCP — third-party
`metatool-ai` software despite the name, …) stay in `Worph/AppStore` and must
not be added here.

⚠ **Asset URLs are repo-scoped.** Everything in this repo must reference
`https://cdn.jsdelivr.net/gh/Worph/MetaAppStore@main/Apps/{AppName}/`. A file
copied over from `Worph/AppStore` will carry the old
`.../gh/Worph/AppStore@main/...` URL and its icon will silently 404 in the store
listing. Grep after any import:

```bash
grep -rn 'gh/Worph/AppStore@main' Apps/   # must be empty
```

## Structure

- `Apps/{AppName}/` — one dir per app:
  - `docker-compose.yml` — compose + `x-casaos` metadata (the store entry)
  - `icon.png`, `thumbnail.png`, `screenshot-{1,2,3}.png`
  - optional `rationale.md`, `README.md`, `pre-install/`
- `category-list.json` — `Media` (every MetaFeeder and client app) and
  `Developer` (the MetaPlugin family)
- `recommend-list.json`, `featured-apps.json` — store front placement
- `psd-source/meta-icons-v2/` — procedural icon generator for the family
  (`gen_mesh.py` mesh sphere + `meta_icons.py` per-app badge); see its README

## Conventions

- **Storage**: `/DATA/AppData/{appname}/`; media under `/DATA/Media/`, `/DATA/Downloads/`
- **Networking**: `expose`, not `ports`; main services join the external `pcs`
  network for Caddy routing. Three caddy labels per public service:
  `caddy_0` gateway domain (`appname-${APP_DOMAIN}`, `gateway_tls` import),
  `caddy_1` nip.io, `caddy_2` sslip.io (Let's Encrypt, no import)
- **`store_app_id`** is the lowercased app name (`MetaFeederTMDB` → `metafeedertmdb`)
- **Images**: `ghcr.io/worph/meta*` pinned to `:latest` on purpose; the Yundera
  perimeter images (`nginx-hash-lock`, `appshield`) stay version-pinned
- **Install order**: `MetaCore` first — every other app depends on it
- **Multi-language**: descriptions/taglines carry `en_us`, `fr_fr`, `es_es`, `zh_cn`, `ko_kr`, `de_de`

## The three app families

| Family | What it is | Shape | Category |
|---|---|---|---|
| `Meta*` | The services — Core, Sort, Fuse, Share, Gateway, Watch, Read, Stremio | backend + AppShield gate, real dashboard | Media |
| `MetaFeeder<Provider>` | One upstream, one app (see the provider-naming rule below) | `<sid>-feeder` + AppShield gate, `index: /config` | Media |
| `MetaPlugin<Name>` | One `metamesh-plugin-*` enrichment worker, standalone | `<sid>-plugin` + AppShield gate, `index: /manifest` | Developer |

**MetaFeeder apps are named by PROVIDER**, one provider per app — role/bundle
names (Card, Indexer, Comic, Book, Common, Paper, Subtitle) are retired. Gateway
feeder keys follow `<provider>-feeder`; the feeder's `upstream_id()` in its crate
is the authority for the provider spelling.

### MetaPlugin: what these apps can and cannot do

A `metamesh-plugin-*` image is a **headless** worker speaking four verbs —
`/manifest`, `/health`, `/configure`, `/process`. It has no UI, so the tile opens
`/manifest`, and it holds **no credentials**: the *caller* POSTs `/configure` with
its own TMDB token / OpenSubtitles account before use. Two consumers sharing one
instance are last-writer-wins on that config.

⚠ **MetaSort cannot consume these apps.** `ContainerPluginConfig`
(`packages/meta-sort/.../container-plugins/types.ts`) takes an `image:` and
MetaSort spawns the container itself; there is no `url:` field. Until one exists,
installing a MetaPlugin app does **not** add that plugin to MetaSort.

⚠ **Only three plugins have a consumer that takes a URL today** — the feeder
SDK's sidecar slots, `FILENAME_PARSER_URL`, `TMDB_PLUGIN_URL` and
`OPENSUBTITLES_PLUGIN_URL` (`meta-feeder-sdk/src/enrich.rs`). The other nine
MetaPlugin listings are packaging ahead of the consumer, and each one says so in
its own description and `tips` — keep that wording honest if the situation
changes.

⚠ **`MetaPluginFilenameParser` cannot be shared between a feeder and MetaSort.**
`EMIT_TITLE_TAGS` is process-level and the two want opposite values (a feeder has
no real file and wants title guesses; MetaSort probes the real stream and must
not have them). Install a second copy rather than flipping it.

Note the SDK **derives** a sidecar URL from the feeder's own hostname when the
variable is unset (`derive_sidecar_url`), which is why pointing a feeder at a
shared MetaPlugin app requires setting the variable explicitly.

## Cross-repo notes

- Apps here are the deployable face of the services in `MetaMesh/meta-root-v2`;
  this repo is vendored there as `packages/MetaAppStore`. A compose change made
  directly on a box (e.g. `holyhorse`, `watch.nsl.sh`) is overwritten by a store
  reinstall — port anything load-bearing back here.
- Non-Meta companions live in `Worph/AppStore`: **Suwayomi** (chapter tier for
  `MetaFeederSuwayomi`) and **SegmentPlayer**. Cross-references between the two
  stores are prose-only; don't add hard dependencies.

## Validation

```bash
docker compose -f Apps/{AppName}/docker-compose.yml config   # compose validity
jq . category-list.json recommend-list.json featured-apps.json
grep -rn 'gh/Worph/AppStore@main' Apps/                       # must be empty
```
