# LiDAR map project - handoff for Claude

Written 2026-09-20 by Kun, the assistant that made the changes described here.
Read this file first, then CHANGES.md (every change in detail), then AGENTS.md
(working rules). The original handoff in ../handoff/ is still the baseline for
everything that was true on the morning of 2026-09-20, but several of its
numbers are now superseded (see section 5).

## 1. What this project is

A LiDAR prospecting map for Luxembourg and Wallonia. High resolution shaded
relief next to aerial imagery behind a draggable swipe divider. The owner uses
it to find abandoned and ruined structures, especially under forest canopy. It
is not a web app: each build is a single self-contained HTML file, no server,
no build step, no keys. Leaflet 1.9.4 from cdnjs is the only library.

## 1b. Naming (owner request, 2026-09-21)

English-facing text says **Wallonia**, not "Wallonie": page titles, the region
menu ("Wallonia", "Luxembourg + Wallonia"), search hit labels, the landing
page and the README. Deliberately left as "Wallonie": the French UI strings,
the agency's official name "Service public de Wallonie", the domain
`geoservices.wallonie.be`, and the file names (renaming them would break the
live URLs). German keeps "Wallonien". See CHANGES.md C6.

## 2. Where the files are

The owner's PC, absolute paths (everything lives under one folder):

```
C:\Users\User\.kun\default_workspace\Plans\
  tests\          the builds (the only copies) and helpers
    luxembourg-wallonie-lidar-v2.html      2,344,433 bytes, sha256 f75b407a...
    luxembourg-wallonie-lidar-mobile.html  2,354,504 bytes, sha256 4a403b8e...
    serve-windows.cmd       double-click LAN server for phone testing
    allow-port-8000.cmd     one-time firewall helper (TCP 8000, private)
    RUINS-EXPANSION.md      change set A notes (totals superseded)
  claude\         this documentation folder (HANDOFF, CHANGES, AGENTS)
  site\           the published GitHub Pages site, a git repo (see section 11)
                  index.html + COPIES of both builds + README + docs\
  sync-site.cmd   re-copies the builds and docs from tests\ and claude\
                  into site\, then prints the commit and push commands
  work\           scratch: raw Overpass pulls, build scripts, test dumps
                  (safe to delete; the changelog references the scripts)
  handoff\        the previous agent's docs, kept for reference
```

`site\` holds COPIES. `tests\` is still the only place the builds are edited.
After changing a build, run `sync-site.cmd`, then commit and push from `site\`,
or the live site silently serves the old version.

History (2026-09-20, owner request): the older build files that used to sit in
`Plans\` were DELETED, along with the macOS-only `serve.command` and
`stop.command`. The two files in `tests\` are now the only builds, and they
are the ones the owner uses. Do not delete or overwrite them without asking.

## 3. What the test builds contain beyond the originals

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
   default ON. See section 7 for the data schema and the removal recipe.
3. **Drag fix.** The hard pan limit (`maxBounds`) is gone. In the combined
   region the map could previously be dragged left/right only within a narrow
   slack at some zooms; now panning is free everywhere. Details in CHANGES.md.
4. **Legal copy updated** to the new figures (8,478 points, 1,026 bunkers,
   1,570 duplicates) plus a "Bike trails" row in the Data sources table.
5. **Layers panel scroll fix.** The panel now caps at the window height and
   its body scrolls; before this, the bottom of the panel (the Sites chips)
   could sit below the screen with no way to scroll to it. The phone panel
   got the same treatment for iOS Safari (`min-height:0`, entry B10 in
   CHANGES.md).

## 4. Current data facts

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

## 5. Superseded numbers

`handoff/HANDOFF.md` and `tests/RUINS-EXPANSION.md` carry the older figures
(8,250 and 8,472, "1,020 bunkers", "1,568 duplicates", "139 mine shafts" is
still right). The build files themselves are the truth. If a doc number
conflicts with the build, the build wins.

## 6. Colour chart (owner-confirmed on 2026-09-20)

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

## 7. The bike overlay: how it works, how to remove it

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

## 8. Sources checked for the expansions

- OpenStreetMap via Overpass: used for everything.
- geoportail.lu: open services are raster only (ortho, topo, cadastre,
  buildings, lidar). No rescue/archaeology/mine vector layer exists.
- data.public.lu: INSPIRE "Protected Sites - Cultural Monuments" (1,371
  features) was found and deliberately excluded: names are internal codes
  like `LUX_HOL_esc_62_MN`, the set mixes buildings with trees and gardens,
  and it has no ruin/abandoned classification.
- Apple Maps / Google Maps: not usable. Terms forbid bulk extraction and
  there is no keyless access; the project stays keyless and open-data only.
- Leaflet: the rendering library, not a data source.

## 9. Open items (things the owner may come back on)

- Phone build is untested on real hardware. It boots clean with the same
  dataset in headless testing, but nobody has touched it with fingers.
- Bike overlay removal recipe above is one request away.
- "Mine shafts" chip text now also covers adits; renaming is possible.
- Casemates (Pétrusse, Bock) were treated as fortifications, not bunkers.
- "Burgruine Berbourg" (49.7318, 6.3917) still has no ruins tag in OSM and is
  therefore not a dot; add by hand if the owner wants it.
- Wallonia dots were not expanded beyond the bike overlay (owner asked for
  Luxembourg-only dot additions).

## 10. Standing instruction

The owner's rule, extended to this folder: **every change updates CHANGES.md
with a detailed entry, and this HANDOFF if state changed, in the same pass.**
Small changes count. A change log that lags is worth nothing.

## 11. GitHub Pages: TAKEN OFFLINE 2026-09-21

**Current state:** the repo `A097MPRUS/luxembourg-wallonie-lidar` is PRIVATE
and GitHub Pages is OFF. The owner asked for this on 2026-09-21 out of concern
about the Esri World Imagery licence. The URL below no longer serves the site
(fresh requests 404; GitHub's CDN may serve a cached front page for a few
minutes after). Do NOT make the repo public or re-enable Pages without the
owner asking. Before any republish, resolve the imagery question (swap Esri
for the open geoportail.lu ortho layers, or confirm Esri's terms fit). The
history below is kept for reference. See CHANGES.md C7.

### History: published 2026-09-20

The owner green-lit publishing on 2026-09-20 and chose GitHub Pages, public,
under the A097MPRUS account. It is live.

- **URL**: https://a097mprus.github.io/luxembourg-wallonie-lidar/
- **Repo**: https://github.com/A097MPRUS/luxembourg-wallonie-lidar, public,
  branch `main`, Pages serving from `main` at `/`.
- **Local repo**: `Plans\site\`. Its git identity is set on that repo only,
  `A097MPRUS <314761645+A097MPRUS@users.noreply.github.com>`, so the owner's
  real email stays out of the public commit history. No global git config was
  touched.
- **Contents**: `index.html` (new landing page), copies of both builds,
  `README.md`, `.nojekyll` (stops Pages processing the single-line data
  blocks), `.gitattributes` with `* -text` (keeps the builds byte exact on
  checkout), and `docs\` with HANDOFF, ARCHITECTURE, CHANGES and AGENTS.
- **Landing page behaviour**: one primary "Open the map" action that points at
  the mobile build when `matchMedia("(pointer:coarse)")` and the viewport is
  under 900 px, otherwise the v2 build, plus explicit links to both so either
  is always reachable. It is a link change, not a redirect. If the owner wants
  the URL to land straight in the map, that is a two line change.
- **Verified live**: Pages build succeeded in 22.2 s; the landing page and the
  v2 build both load over HTTPS in a real browser, 48 of 48 tiles, no console
  errors, no mixed content.

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

- **Esri World Imagery is not open data.** The site is public and serves that
  layer. This is a licensing question, not a technical one, and it was flagged
  before publishing rather than changed unilaterally. The open alternative is
  the geoportail.lu ortho layers.
- **Nominatim** address search is now reachable by the public. Fine at the
  current scale; it needs a plan if the site gets real traffic.
- **A public repo makes the dataset and the code public.** That was accepted.
- **Custom domain** is supported and free to point at Pages. Only the domain
  registration costs anything: `.org` roughly EUR 10-15/year, `.lu` roughly
  EUR 25-40/year through an accredited registrar and it needs an EU or
  Luxembourg link.
