# MetaMesh icon family v2

Procedural replacement for the lost v1 icon sources. One mesh sphere, generated
from code (`gen_mesh.py`: fibonacci sphere → convex hull → front-facing edges,
teal → blue → purple gradient), plus a per-app badge (`meta_icons.py`): dark disc,
family-coloured ring, white glyph, auto-fitted label.

- **Neutral** (no badge): MetaCore.
- **Ring colours:** teal = MetaShare, orange = MetaGateway and every MetaFeeder,
  gold = MetaSort, green = MetaFuse, purple = MetaStremio, **blue = every
  MetaPlugin** (the headless enrichment workers).
- A glyph may be reused across two families — the ring colour and the label
  separate them (`photo` is WikiCommons in orange and Still Extractor in blue).
  Within one colour, never reuse a glyph.
- MetaWatch / MetaRead (client apps) keep their half-moon icons; MetaMCP keeps its own.

Regenerate (needs `python3`, `rsvg-convert`, ImageMagick `convert` + DejaVu Sans Bold):

```bash
python3 meta_icons.py out            # all apps → out/<App>.svg + out/<App>.png (1024²)
python3 meta_icons.py out MetaFeederTMDB
```

Then copy `out/<App>.png` to `Apps/<App>/icon.png` (and `thumbnail.png` where the
listing references one). Add a new app by adding one line to `APPS` (and a glyph to
`G` if none fits). `svg/` holds the rendered masters.
