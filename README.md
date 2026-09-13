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
| Feeders | `MetaFeederAniList`, `MetaFeederArxiv`, `MetaFeederEuropePMC`, `MetaFeederGiphy`, `MetaFeederGutenberg`, `MetaFeederInternetArchive`, `MetaFeederJamendo`, `MetaFeederMusicBrainz`, `MetaFeederOpenSubtitles`, `MetaFeederProwlarr`, `MetaFeederSciHub`, `MetaFeederSuwayomi`, `MetaFeederTMDB`, `MetaFeederTribler`, `MetaFeederUsenet`, `MetaFeederWikiCommons`, `MetaFeederYouTube` |
| Plugins | `MetaPluginAnimeDetector`, `MetaPluginFFmpeg`, `MetaPluginFileInfo`, `MetaPluginFilenameParser`, `MetaPluginFullHash`, `MetaPluginJellyfinNFO`, `MetaPluginLanguage`, `MetaPluginOpenSubtitles`, `MetaPluginStillExtractor`, `MetaPluginSubtitle`, `MetaPluginSubtitleExtractor`, `MetaPluginTMDB` — headless enrichment workers, one per `metamesh-plugin-*` image |

`MetaCore` is the dependency of every other app — install it first.

A **MetaPlugin** app is headless: no dashboard, the tile opens the plugin's
`/manifest`, and it holds no credentials of its own (the caller configures it).
MetaSort does **not** consume these — it starts its own plugin containers from
`plugins.yml` and has no slot for a plugin URL. Today the consumers that do take
a URL are the MetaFeeder sidecar slots for `filename-parser`, `tmdb` and
`opensubtitles`; each listing says where it stands.

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
