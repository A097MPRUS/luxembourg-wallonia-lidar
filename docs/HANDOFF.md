# LiDAR map project - handoff for Claude

Written 2026-09-20 by Kun, the assistant that made change sets A and B.
Fully revised 2026-09-21 by Claude (change sets C, D and E: publishing, naming,
the switch to government aerial photos, and dropping Luxembourg's 1967 photo).

Read this file first, then CHANGES.md (every change in detail), then AGENTS.md
(working rules), then `../handoff/ARCHITECTURE.md` (code map). The original
handoff in `../handoff/` is the baseline for what was true on the morning of
2026-09-20; several of its facts are now superseded (see section 5).

## 1. What this project is

A LiDAR prospecting map for Luxembourg and Wallonia. High resolution shaded
relief next to the governments' own aerial photos, behind a draggable swipe
divider. The owner uses it to find abandoned and ruined structures, especially
under forest canopy. It is not a web app: each build is a single
self-contained HTML file, no server, no build step, no keys. Leaflet 1.9.4
from cdnjs is the only library.

**Every map layer now comes from a public body or from OpenStreetMap.** Esri
World Imagery was removed on 2026-09-21 at the owner's request and replaced by
ACT and SPW orthophotos (section 8). Do not add a commercial imagery layer back.

## 2. Naming (owner request, 2026-09-21)

English-facing text says **Wallonia**, not "Wallonie": page titles, the region
menu ("Wallonia", "Luxembourg + Wallonia"), search hit labels, the landing
page and the README. Deliberately left as "Wallonie": the French UI strings,
the agency's official name "Service public de Wallonie", the domain
`geoservices.wallonie.be`, and the file names (renaming them would break the
live URLs). German keeps "Wallonien". See CHANGES.md C6.

## 3. Where the files are

The owner's PC, absolute paths (everything lives under one folder):

```
C:\Users\User\.kun\default_workspace\Plans\
  tests\          the builds (the working copies) and helpers
    luxembourg-wallonie-lidar-v2.html      2,348,671 bytes, sha256 92fe2b43...
    luxembourg-wallonie-lidar-mobile.html  2,358,744 bytes, sha256 16020f91...
    serve-windows.cmd       double-click LAN server for phone testing
    allow-port-8000.cmd     one-time firewall helper (TCP 8000, private)
    RUINS-EXPANSION.md      change set A notes (totals superseded)
  claude\         this documentation folder (HANDOFF, CHANGES, AGENTS)
  site\           the published GitHub Pages site, a git repo (section 12)
                  index.html + COPIES of both builds + README + docs\
  sync-site.cmd   re-copies the builds and docs from tests\, claude\ and
                  handoff\ARCHITECTURE.md into site\, then prints the commit
                  and push commands
  work\           scratch: raw Overpass pulls, build scripts, test dumps
                  (safe to delete; the changelog references the scripts).
                  work\make_alpha.py produced the orthophoto change (change
                  set D); work\drop_1967.py removed Luxembourg's 1967 photo
                  (change set E).
  handoff\        the previous agent's docs, kept for reference
```

`site\` holds COPIES. `tests\` is the only place the builds are edited.
After changing a build, run `sync-site.cmd`, then commit and push from `site\`,
or the live site silently serves the old version.

**Backups of earlier builds** live in the `site\` git history. The last builds
with Esri imagery are in commit `4ffc6d9`, the commit just before change set D.

History (2026-09-20, owner request): the older build files that used to sit in
`Plans\` were DELETED, along with the macOS-only `serve.command` and
`stop.command`. Do not delete or overwrite the builds in `tests\` without asking.

## 4. What the builds contain beyond the originals

1. **Ruins layer expanded, Luxembourg only.** 8,250 -> 8,478 points.
   - +222 from change set A: archaeological sites (79), city walls (65),
     forts (23), mine adits (51), historic mines (2), city gates (2).
   - +6 from change set B: bunker sites (Monument Bunker Hondsbesch,
     Bunker an der Runtschelt, Friedboesch Bunker, and three OSM points
     literally named "Bunker"/"bunker").
   - Wallonia dots unchanged at 6,316.
2. **Second overlay: Bike trails.** Dark brown (#5b3a1e) lines of cycleways
   and signed bicycle routes from OpenStreetMap, both territories.
   23,764 lines (~6,147 km simplified length), LU 7,458 / WAL 16,306.
   Toggle "Bike trails" sits in the Overlays group of the panel (both builds),
   default ON. See section 9 for the data schema and the removal recipe.
3. **Drag fix.** The hard pan limit (`maxBounds`) is gone. In the combined
   region the map could previously be dragged left/right only within a narrow
   slack at some zooms; now panning is free everywhere. Details in CHANGES.md.
4. **Legal copy updated** to the current figures (8,478 points, 1,026 bunkers,
   1,570 duplicates), a "Bike trails" row, and the orthophoto sources.
5. **Layers panel scroll fix.** The panel caps at the window height and its
   body scrolls. The phone panel got the same treatment for iOS Safari
   (`min-height:0`, entry B10 in CHANGES.md).
6. **Government aerial photos replace Esri** (2026-09-21, change set D), with an
   "Aerial photo" picker: four options in Wallonia, three in Luxembourg
   (1967 removed, change set E). Section 8.
7. **"Wallonia" in English-facing text** (section 2).

## 5. Superseded facts in older docs

- `handoff/HANDOFF.md` and `tests/RUINS-EXPANSION.md` carry older figures
  (8,250 and 8,472, "1,020 bunkers", "1,568 duplicates"; "139 mine shafts" is
  still right).
- `handoff/HANDOFF.md` section 5 lists Esri World Imagery as the imagery
  source. **That is superseded**: the row is marked REMOVED there, and the
  sources are now in section 8 below.
- The build files themselves are the truth. If a doc conflicts with the build,
  the build wins.

## 6. Current data facts

Ruins points by kind (both territories, 8,478 total):

| kind | meaning | count |
| --- | --- | --- |
| 0 | Ruins (incl. archaeo sites, walls, forts in LU) | 1,348 |
| 1 | Bunkers | 1,026 |
| 2 | Abandoned | 2,341 |
| 3 | Disused | 3,153 |
| 4 | Mine shafts (incl. adits in LU) | 139 |
| 5 | Brownfield | 471 |

By territory: Luxembourg 2,162, Wallonia 6,316. Raw pull 10,046 -> 8,478,
so the copy says 1,570 duplicates collapsed. All OSM, pulled 2026-09-20 with
Overpass (`out center tags` / `out geom`, real User-Agent required).
Also embedded: 4,583 place names, 2,550 condition outlines, 23,764 bike lines.
All counts re-read from the shipped data blocks on 2026-09-21.

## 7. Colour chart (owner-confirmed on 2026-09-20)

| Colour | Hex | Meaning |
| --- | --- | --- |
| Red | #d1495b | Ruins |
| Green | #2f6f4e | Bunkers |
| Orange | #e07a1f | Abandoned |
| Yellow | #e8c33a | Disused (off by default) |
| Near-black grey | #33383d | Mine shafts |
| Olive-brown | #8a7a1f | Brownfield (off by default) |
| Dark brown | #5b3a1e | Bike trails overlay (lines, not dots) |

Do not reuse any of these for something else.

## 8. Aerial photos (government orthophotos)

Replaced Esri World Imagery on 2026-09-21 (owner confirmed the alpha, then
asked for it everywhere). Change set D in CHANGES.md has the full evidence.

### Sources and licences

| Region | Service | Licence, as checked 2026-09-21 |
| --- | --- | --- |
| Luxembourg | ACT, `wmts{1-4}.geoportail.lu/opendata/wmts/<layer>/GLOBAL_WEBMERCATOR_4_V3/{z}/{x}/{y}.jpeg` | Only CC0 (public domain) editions are used: 2001, 2025 summer, 2025 winter, each `cc-zero` on data.public.lu. The 1967 edition (`ortho_1967`) has licence "notspecified" there and was **removed** at the owner's request (change set E). Do not add it back. |
| Wallonia | SPW, `geoservices.wallonie.be/arcgis/services/IMAGERIE/<service>/MapServer/WMSServer` (WMS 1.3.0, layer `0`, EPSG:3857) | SPW "Conditions d'accès et d'utilisation des services web géographiques de visualisation" v1.1 (2016): free for any user; keep the source credited, do not alter the images, do not overload the servers. Same terms as the relief layers. |

Attribution in the map reads "Relief, orthophoto : © ACT / geoportail.lu" and
"Relief, orthophoto : © SPW / geoportail.wallonie.be". Keep it; SPW's terms
forbid removing the source mention.

### The four options (`IMAGERY` array in the IIFE)

| Option (id) | Luxembourg layer | Wallonia service |
| --- | --- | --- |
| Summer (`summer`, default) | `ortho_2025` (2025 été) | `ORTHO_2023_ETE` (2023) |
| Winter / spring (`leafoff`) | `ortho_2025_winter` (2025 hiver) | `ORTHO_2026_PRINTEMPS` (2026) |
| Around 2001 (`y2001`) | `ortho_2001` | `ORTHO_2001_2003` |
| Around 1970 (`y1970`) | none (`lu: null`) | `ORTHO_1971` |

Years come from each service's own title. The year column shows the Luxembourg
year, the Walloon year, or both ("2025 / 2023") in the combined region.

**Options without a Luxembourg photo** (`lu: null`, currently only `y1970`):
`imgAvail(def)` hides them in the Luxembourg region; in the combined region
they show Wallonia only, year column "WAL 1971", and Luxembourg has no photo
under the swipe. If the chosen option is not available after a region switch,
`setImagery` falls back to Summer and ticks its radio.

**Coverage was verified before choosing**, because some SPW campaigns are
partial. Tested at 10 Walloon towns (Tournai, Mons, Namur, Liège, Eupen,
Dinant, Bastogne, Arlon, Chimay, Wavre) and 7 Luxembourg towns:
- `ORTHO_2025_ETE` had photo at **none** of the 10 Walloon points, so it is
  not used. `ORTHO_2024` misses Tournai, Arlon and Chimay.
- Every campaign in the table above covers all test points.
- SPW's `ORTHO_LAST` ("latest campaign") is pixel-identical to
  `ORTHO_2023_ETE` at three test points. The dated service is used on purpose
  so the year label cannot go stale when SPW moves `ORTHO_LAST` on.

### How it works

- Pane `sat` (z 350) as before; the swipe clip is unchanged.
- `luOrtho(id)` is a `Wmts` layer (JPEG, subdomains 1-4, `maxNativeZoom` 20).
- `walOrtho(svc, png)` is an `L.tileLayer.wms`, JPEG in the Wallonia region.
- In the combined region both are stacked in a layer group: Luxembourg below,
  Wallonia above as **transparent PNG**. SPW returns transparent pixels where
  it has no photo, so Luxembourg shows through; ACT returns white outside
  Luxembourg, which Wallonia's photo covers. Verified pixel by pixel at four
  border tiles: 0% white, 0% empty after stacking.
- `setImagery(id)` swaps the layer; it runs at boot and from `setRegion`.
  `buildImagery()` renders the picker; it runs at boot, from `setRegion` and
  from `setLang`. The chosen option persists across region switches.
- New ids: `imgGrp` (the picker container, both builds, right after
  `layerGrp`) and the generated radios `im-summer`, `im-leafoff`, `im-y2001`,
  `im-y1970`. New translation keys in all four languages: `aerial`,
  `imgSummer`, `imgLeafoff`, `img2001`, `img1970`.

### Known quirk

In the combined "Luxembourg + Wallonia" view, SPW's photos run about 1 to 2 km
past the Walloon border (fully transparent by lon 5.94 near Steinfort, within
about 1 km at Martelange). A thin strip on the Luxembourg side therefore shows
the Walloon photo, which may be a different year. No gaps or blank areas.
Single-region views are unaffected. A fix would clip SPW tiles to the Walloon
polygon; not done, owner not yet asked.

### To add another year

Check coverage first (the Walloon service must have photo across the region,
not just at one point), read the year from the service title, add a row to
`IMAGERY` in both builds, add the year to the "Aerial photos" rows of the Data
sources table in both builds, and update the table above.

## 9. The bike overlay: how it works, how to remove it

- Data lives in `<script type="application/json" id="bikedata">`, one line.
  Schema per line: `[region, minLat, minLon, maxLat, maxLon, [flat ints]]`.
  Flat ints are consecutive (dLat, dLon) offsets in 1e-5 degree units from
  (minLat, minLon); decode exactly like `ringOf()` does for shapes.
  Lines are ordered route-members-first so the 3000-lines-per-viewport cap
  keeps the signed routes when zoomed out.
- Code: one self-contained block in the IIFE right after the `drawShapes`
  wiring, marked with a comment. Pane `bike` z-index 430 (under the mask),
  canvas renderer, colour #5b3a1e, weight 2.5, opacity 0.85.
- Toggle: checkbox `#bikes`, label `#bikesLbl`, hint `#bikeHint`, in the
  Overlays group of both builds; translations `bikelayer`/`bikehint` in all
  four languages; `drawBike()` is called from `setRegion`, boot and the map
  events.
- **To remove it completely** (the owner may ask): delete the `bikedata`
  script tag, the bike JS block, the three `drawBike()` calls, the two
  `setLang` lines, the markup label + hint in both builds, the four
  translation entries, and the "Bike trails" row in the sources table.
  Nothing else depends on it.

## 10. Sources checked

- OpenStreetMap via Overpass: used for all points, outlines and bike lines.
- geoportail.lu: open services are raster only (ortho, topo, cadastre,
  buildings, lidar). No rescue/archaeology/mine vector layer exists. Its open
  WMTS lists orthophotos from 1967 to 2025, plus winter 2019 and 2025 and an
  infrared set.
- geoservices.wallonie.be: `IMAGERIE` folder lists orthophotos from 1971 to
  spring 2026, several of them partial (section 8).
- data.public.lu: INSPIRE "Protected Sites - Cultural Monuments" (1,371
  features) was found and deliberately excluded: names are internal codes
  like `LUX_HOL_esc_62_MN`, the set mixes buildings with trees and gardens,
  and it has no ruin/abandoned classification.
- Esri World Imagery: removed 2026-09-21. Not open data; the map loaded its
  tiles without an ArcGIS account, which Esri's terms do not clearly allow.
- Apple Maps / Google Maps: not usable. Terms forbid bulk extraction and
  there is no keyless access; the project stays keyless and open-data only.
- Leaflet: the rendering library, not a data source.

## 11. Open items (things the owner may come back on)

- Phone build is untested on real hardware. It boots clean in headless
  testing and at 375 px in a browser, but nobody has touched it with fingers.
- The combined-view border strip (section 8, known quirk).
- Bike overlay removal recipe (section 9) is one request away.
- "Mine shafts" chip text now also covers adits; renaming is possible.
- Casemates (Pétrusse, Bock) were treated as fortifications, not bunkers.
- "Burgruine Berbourg" (49.7318, 6.3917) still has no ruins tag in OSM and is
  therefore not a dot; add by hand if the owner wants it.
- Wallonia dots were not expanded beyond the bike overlay (owner asked for
  Luxembourg-only dot additions).

## 12. GitHub Pages site: LIVE

- **URL**: https://a097mprus.github.io/luxembourg-wallonie-lidar/
- **Repo**: https://github.com/A097MPRUS/luxembourg-wallonie-lidar, public,
  branch `main`, Pages serving from `main` at `/`.
- **Local repo**: `Plans\site\`. Its git identity is set on that repo only,
  `A097MPRUS <314761645+A097MPRUS@users.noreply.github.com>`, so the owner's
  real email stays out of the public commit history. No global git config was
  touched.
- **Contents**: `index.html` (landing page), copies of both builds,
  `README.md`, `.nojekyll` (stops Pages processing the single-line data
  blocks), `.gitattributes` with `* -text` (keeps the builds byte exact on
  checkout), and `docs\` with HANDOFF, ARCHITECTURE, CHANGES and AGENTS.
- **Landing page behaviour**: one primary "Open the map" action that points at
  the mobile build when `matchMedia("(pointer:coarse)")` and the viewport is
  under 900 px, otherwise the v2 build, plus explicit links to both. It is a
  link change, not a redirect; landing straight in the map is a two line
  change if the owner wants it.
- **History**: published 2026-09-20 (C1 to C5). Briefly made private and taken
  offline on 2026-09-21 (C7), which was a misreading: the owner had asked for
  that only IF the Esri imagery made it illegal. Restored the same day (C8).
  Esri was then replaced entirely (change set D), which settles the question.

### To update the live site after a build change

```
Plans\sync-site.cmd
cd site
git add -A
git commit -m "..."
git push
```

Pages rebuilds in well under a minute.

### Still open for the owner to decide

- **Nominatim** address search is reachable by the public. Fine at the
  current scale; it needs a plan if the site gets real traffic.
- **A public repo makes the dataset and the code public.** That was accepted.
- **Custom domain** is supported and free to point at Pages. Only the domain
  registration costs anything: `.org` roughly EUR 10-15/year, `.lu` roughly
  EUR 25-40/year through an accredited registrar and it needs an EU or
  Luxembourg link.

## 13. Standing instruction

The owner's rule: **every change updates CHANGES.md with a detailed entry, and
this HANDOFF if state changed, in the same pass.** Small changes count. A
change log that lags is worth nothing. The owner may ask for an alpha first
("do not update the handoff until I confirm"); then build in a separate folder,
leave `tests\`, the site and the docs alone, and document everything in the
same pass as the promotion.
