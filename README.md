# Estudio Tile Picker

A one-time tool so designers can pick which tiles should be ordered as
physical samples, across all 9 Estudio Cerámico catalogs on file (406 named
designs).

**Send designers this link — nothing to unzip, nothing to install:**

**https://varundixit-gif.github.io/TileSelection/**

It's hosted as a plain static site on GitHub Pages (repo:
https://github.com/varundixit-gif/TileSelection), so it's a normal webpage
like any other — no Claude account, no sandbox. Designers open the link in
their own browser (Edge/Chrome on Windows), pick tiles, and the Download
and Save-as-PDF buttons both work as real one-click actions. Verified live:
page loads, all 388 catalog images return 200 (GitHub Pages serves from a
case-sensitive filesystem, unlike macOS locally, so this was worth
checking), and `isStandalone` detection correctly enables both buttons.

To update the page later: edit the `catalog-data/*.json` files in this
folder, run `python3 build.py`, then copy `artifact.html` to `index.html`
and `images/` into a clone of the `TileSelection` repo, commit, and push to
`main` — GitHub Pages redeploys automatically in under a minute.

**Alternative: zip and share the folder directly**, if you'd rather not use
the hosted link (e.g. offline designers). Zip this `tile-selector` folder
(`artifact.html` + `images/`) and share it (OneDrive/Drive link, WhatsApp
Desktop — not email, it's ~150MB). They unzip and double-click
`artifact.html`; same no-sandbox behavior as the hosted link, since neither
is a Claude Artifact.

The Claude Artifact link is still useful as **your own quick preview** —
https://claude.ai/artifact/KhnVBFgnJSiUsdcXpCPxim — but Claude Artifacts
sandbox every visitor without a Claude account on the link out of all
file-save paths (not just the `downloads` capability — native browser
download and the print dialog are blocked too, confirmed by testing live
against the published link signed out). Since none of the designers have
Claude accounts, the Artifact link can never give them a real download.
That's why neither the Artifact link nor a claude.ai-hosted copy is what
gets sent to designers — the GitHub Pages link above is.

## How it works

Designers open `artifact.html` (from the unzipped folder), type their name,
search/filter by catalog, check every tile they want sampled (with an
optional quantity per tile), and add any remarks. Clicking **Generate my
list** produces a summary grouped by catalog. From there:
- **Download selection sheet** — a self-contained HTML file with each tile's
  actual image, quantity, catalog file, and page, ready to attach to an
  email. This is the one-click "must have" download — it only works when the
  page is opened locally like this, not through the Claude Artifact link.
- **Save as PDF** — runs the browser's native print dialog on a clean,
  image-included layout; "Save as PDF" as the destination gives the same
  result as the download button, as a backup if download is ever blocked by
  a corporate browser policy.
- **Copy to clipboard** — plain text (name/qty/catalog/page, no images), for
  pasting into WhatsApp/email instead.

There's no shared backend: each designer sends their own file/list back to
you, and you compile the combined order yourself. (A live auto-aggregating
version was considered, but Claude Artifacts can't accept writes from
visitors without a Claude account, and can't call out to an external service
like Google Sheets from inside the sandbox — so "generate locally and send"
is what actually works for designers with no Claude login.)

The page detects which context it's running in (`typeof claude ===
"undefined"` → opened locally, real downloads work; otherwise → running
inside the Claude Artifact sandbox, only Copy to clipboard reliably works)
and shows the right buttons automatically — no manual switching needed.

## Files

- `artifact.html` — the published page (source of truth; `build.py`
  regenerates it).
- `catalog-data/*.json` — one file per catalog: `{catalogId, catalogLabel,
  catalogFile, entries: [{image, page, tiles: [names]}]}`.
- `images/<catalogId>/*.jpg` — swatch thumbnails cropped/rendered from each
  catalog PDF.
- `build.py` — regenerates `artifact.html` from the `catalog-data/*.json`
  files. Run `python3 build.py` from this folder after editing any catalog
  JSON (e.g. fixing a tile name), then republish with the Artifact tool
  (`url` = the link above, `file_path` = `artifact.html`).

## Catalogs covered

| Catalog | Source file | Designs |
|---|---|---|
| Motif Collection | GOLD MOROCCON.pdf | 27 |
| Tropical Jewel Collection | Tropical Estudio .pdf | 13 |
| Moroccan Collection (20x20cm) | Estudio 20x20 cm.pdf | 51 |
| Live Life the Moroccan Way (60x60cm) | Estudio 60x60 Morrocon Catalouge Jan 24.pdf | 15 |
| 300×300mm Moroccan Collection | estudio_300x300mm_morroccan_ecatalogue_sep_26.pdf | 26 |
| Trend Collection 60×120cm | Trend Collectio 60x120 Catalogue 11 .pdf | 11 |
| Fullbody Decor | Full body Estudio .pdf | 11 |
| e decor — Ceramic Wall Tiles Collection 2024 | ESTUDIO SPECIAL e -DECOR.pdf | 61 |
| Estudio Gallery — 30 Panel Collection | estudio 30 panel Estudio Gallary.pdf | 191 |

## Known quirks worth knowing about

- **Gallery — 30 Panel Collection overlaps heavily with the other 8
  catalogs.** It's a multi-line sampler PDF that re-prints many designs from
  Motif, Trend 60x120, and Tropical Jewel alongside its own. Same tile name
  can appear under two different catalogs with two different crops — that's
  expected, not a bug. If it's more confusing than useful, consider hiding
  that catalog pill's section, or just tell designers to ignore duplicates.
- **Image style isn't consistent across catalogs.** Some catalogs (Motif,
  Tropical, Trend, Fullbody, 300x300, Special Decor) have a clean swatch-only
  index page per design. A couple (20x20cm, 60x60cm Moroccan) only have
  combined pages — name + swatch + a lifestyle room photo baked into one
  image — because that's how those catalog PDFs are laid out; there was no
  cleaner page to crop.
- One tile in the 20x20cm catalog, **"Chiffon Tray"**, has a font
  ligature-rendering bug baked into the source PDF — the image itself shows
  "Chi on Tray" even though the checkbox/label correctly reads "Chiffon
  Tray". Cosmetic only, not worth custom-fixing the image for one tile.
- In the Special Decor catalog, "**Gold Metallic Base**" and "**Silver
  Metallic Base**" each appear 2-3 times (different pages, different plain
  swatch images) — kept as distinct entries rather than merged, since each
  has its own catalog page reference.
- A few tile names were corrected from an obvious source-PDF OCR/font glitch
  (e.g. "Chi on Tray" → "Chiffon Tray" in the JSON's `tiles` field, though not
  in the baked-in image). If you spot the vendor's official name differs,
  edit the relevant `catalog-data/*.json` and rerun `build.py`.

## Adding more catalogs later

Follow the same pattern: render each catalog's clean swatch/index pages as
JPEGs into `images/<new-catalog-id>/`, write a
`catalog-data/<new-catalog-id>.json` matching the schema above, add its id to
the `ORDER` list at the top of `build.py`, then run `python3 build.py` and
republish.
