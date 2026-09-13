# MetaAppStore

CasaOS / Yundera 3rd-party app store for the **MetaMesh** application family.

Split out of [Worph/AppStore](https://github.com/Worph/AppStore) so the MetaMesh
apps ship on their own release cadence, independent of the general-purpose app
collection.

## Adding the store

In Yundera / CasaOS → App Store → *Add source*:

```
https://github.com/Worph/MetaAppStore/archive/refs/heads/main.zip
```

## What's in here

| Tier | Apps |
|------|------|
| Core | `MetaCore` (leader election, Redis, WebDAV) |
| Library | `MetaSort`, `MetaFuse`, `MetaStremio` |
| Network | `MetaShare` (content transport), `MetaGateway`, `MetaGatewayShared` |
| Clients | `MetaWatch`, `MetaWatchShared`, `MetaRead` |
| Feeders | `MetaFeederAniList`, `MetaFeederArxiv`, `MetaFeederEuropePMC`, `MetaFeederGiphy`, `MetaFeederGutenberg`, `MetaFeederOpenSubtitles`, `MetaFeederProwlarr`, `MetaFeederSciHub`, `MetaFeederSuwayomi`, `MetaFeederTMDB`, `MetaFeederTribler`, `MetaFeederUsenet`, `MetaFeederWikiCommons` |

`MetaCore` is the dependency of every other app — install it first.

Some apps pair with non-Meta apps that remain in
[Worph/AppStore](https://github.com/Worph/AppStore): **Suwayomi** (the chapter
tier `MetaFeederSuwayomi` reads) and **SegmentPlayer**.

## Assets

Icons and screenshots are served from jsDelivr:

```
https://cdn.jsdelivr.net/gh/Worph/MetaAppStore@main/Apps/{AppName}/
```

`psd-source/meta-icons-v2/` holds the procedural icon generator for the family —
see its README to regenerate an app icon.

## Upstream source

These compose files are the deployable face of the MetaMesh services built in
[MetaMesh/meta-root-v2](https://github.com/worph). This repo is vendored there
as the `packages/MetaAppStore` submodule.
