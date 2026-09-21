# Change log

Every change made to this project, newest last. Change sets A and B by Kun
(2026-09-20); C, D and E by Claude (2026-09-20 and 2026-09-21).
One entry per change: what, where, why, and the evidence it works.
Owner's rule: this file gets a detailed entry for EVERY change, in the same
pass as the change itself. Small changes count.

---

## Change set A - Luxembourg ruins layer expansion

**What.** The "Ruins and abandoned" dots for Luxembourg grew from 1,934 to
2,156 points (+222). New tag families, all OpenStreetMap via Overpass, pulled
2026-09-20:

| Tag family | Added | Shown under |
| --- | --- | --- |
| historic=archaeological_site | +79 | Ruins (red) |
| historic=citywalls | +65 | Ruins (red) |
| historic=fort | +23 | Ruins (red) |
| man_made=adit | +51 | Mine shafts (near-black) |
| historic=mine | +2 | Mine shafts (near-black) |
| historic=city_gate | +2 | Ruins (red) |

**Where.** Mapdata `ruins` array in both builds. Rules: skip anything within
35 m of an existing dot, collapse same-kind duplicates within 35 m keeping the
longest label, label = name or a short descriptor. Wallonia untouched.

**Why.** The owner asked for a richer ruins layer for Luxembourg, from OSM or
any usable LU/European database.

**Evidence.** Served and driven with headless Edge on the real finder:
Luxembourg City viewport 33 -> 103 dots (46 city walls); Minett view shows the
new adit dots. All 8,250 original points still present byte-for-byte. Legal
figures updated then (8,250 -> 8,472). Full notes: `tests/RUINS-EXPANSION.md`.

---

## Change set B - bunkers, bike overlay, drag fix, docs

### B1. Luxembourg bunker sweep merged (+6 points)

**What.** Queried every bunker-ish tag in OSM for Luxembourg (military=bunker,
building=bunker, bunker_type, casemate, air raid shelters, name~bunker).
15 elements returned; 7 skipped by rule (casemate attractions, streets, the
three already present). Added 6:

| Label | Coords | Source tag |
| --- | --- | --- |
| Monument Bunker Hondsbesch | 49.53288, 5.89099 | historic=memorial + wikipedia (the Niederkorn bunker museum site) |
| Bunker an der Runtschelt | 49.91564, 5.89758 | tourism=information, named bunker |
| Friedboesch Bunker | 49.95065, 6.04647 | building=hut + bunker_type=personnel_shelter |
| Bunker | 49.78511, 6.16531 | name=Bunker |
| bunker | 49.97433, 6.05316 | building=bunker |
| Bunker | 50.07583, 6.11538 | name=Bunker |

Dropped: "Bunker WW2" memorial node (2 m from the existing dot), "Hide Out
FEB/JUN 1944" (already in as a ruin), casemate attractions, Rue des Casemates.

**Why.** Owner: "look at a whole luxembourgish bunker database", dots must be
green. Reality: Luxembourg has very few bunker sites in OSM (that is the whole
dataset there is; no separate official coordinate database was found on
geoportail.lu or data.public.lu - see HANDOFF section 8).

**Evidence.** Merge log `work/bunker_merge.log`; totals 8,472 -> 8,478,
bunkers 1,020 -> 1,026, duplicates figure 1,568 -> 1,570. Rendered check:
at 49.5329, 5.8910 the green dot renders with title "Monument Bunker
Hondsbesch" (probe run dom_r3_honds3).

### B2. Region-both drag/pan fix (removed maxBounds)

**What.** Removed the three `map.setMaxBounds(...)` calls (setRegion and boot)
and the `maxBoundsViscosity: 1` option. The map can now be panned freely in
every region, including "Luxembourg + Wallonie".

**Why.** Owner: "when selecting the wallonia + luxembourg map dragging the map
to one side is impossible, it blocks me from dragging to the left or right
side". Cause: after each region switch the map applied a hard pan limit equal
to the region box +3%. In the combined region the fitted viewport is as wide
as the whole territory box, so horizontal dragging had only a tiny slack and
then dead-stopped. This is by design of Leaflet's maxBounds + viscosity 1.

**Evidence.** Reproduced on the pre-fix build at zoom 9: a 420 px horizontal
drag moved the view only 86 px, then stopped (run dom_r3pre_drag3). On the
fixed build the same drag moves the full amount with no limit (all post-fix
runs). Static check: zero occurrences of `setMaxBounds` in both builds; per
Leaflet's source, with no maxBounds every clamp path is a no-op.

**Side finding (recorded for honesty).** A pre-existing ordering quirk meant
the pan limit was only ever applied for normal-motion users: `fitBounds` runs
synchronously under `prefers-reduced-motion` and its moveend fires before the
`once("moveend")` that set the limits, so reduced-motion users never got the
clamp. Removing the limits unifies both paths. Headless test note: proving
this took several attempts because virtual time interacts badly with fly
animations; the clean reproduction above is with real (non-reduced) motion.

### B3. Bike trails overlay (new second overlay)

**What.** New line overlay in dark brown (#5b3a1e): cycleways and signed
bicycle routes from OpenStreetMap, Luxembourg and Wallonia. Toggle "Bike
trails" (`#bikes`) in the Overlays group of both builds, default ON. Data:
23,764 lines (~6,147 km simplified), LU 7,458 / WAL 16,306.

**How the data was made** (scripts in `work/`, reproducible):
1. Overpass pull `work/fetch_bike_geom.overpass`: all `highway=cycleway` in
   the map bbox plus member ways of all named `route=bicycle` relations
   (107 MB raw response, one query).
2. `work/process_bike.py`: simplify (Douglas-Peucker 0.00025 deg), clip to
   the two territory polygons (fast scanline grid), drop stubs
   (< 25 m, < 8 m for route members), quantize to 1e-5 deg offsets, tag by
   territory, order route members first.
3. Embedded as `<script type="application/json" id="bikedata">`, schema
   `[region, minLat, minLon, maxLat, maxLon, flat ints]`.

**Why.** Owner asked for a second overlay of bike trails, dark brown, toggle
in the right menu, both regions, and warned he may ask to remove it later
(HANDOFF section 7 has the removal recipe; the code block is self-contained
and commented).

**Evidence.** Probe run at the Hondsbesch view: checkbox default ON, bike pane
canvas draws 1,391 brown pixels; clicking the toggle flips the checkbox state
(off/on verified; pixel recheck after toggling is flaky under headless virtual
time because the canvas repaint waits on a frame, but the draw path is the
same function in all cases). Legend and translations present in all four
languages; the mobile build carries the same overlay.

### B4. Legal copy updated

**What.** Data sources: "8,478 tagged points", "1,026 bunkers", "which removed
1,570 duplicates", table cell "8,478 points", plus a new row
"Bike trails | OpenStreetMap contributors | 2026-09-20 | 23,764 lines".

**Why.** The project rule: every displayed figure must be traceable; stale
figures are worse than none.

**Evidence.** Rendered `#sources` page on the fixed build contains all new
strings and no stale ones (desktop and mobile checked).

### B5. Data block style restored

**What.** The mapdata JSON now sits on its own line between its script tags,
matching the originals' three-line shape (the change-set A patch had merged
opening tag and JSON onto one line, which was harmless but inconsistent).

**Why.** Keep the document shape as the ARCHITECTURE doc describes it.

**Evidence.** Line diff of round-2 vs round-3 builds shows the restored shape.

### B6. This claude folder created

**What.** `Plans/claude/` with HANDOFF.md (project state for the next agent),
CHANGES.md (this file), AGENTS.md (working rules).

**Why.** Owner will continue with Claude and wants every change documented
there, in the same style as the original handoff folder.

### B7. Bike line width increased (owner feedback: too thin)

**What.** Bike polyline `weight` 1.6 -> 2.5 in both builds. One number,
nothing else touched. Build sizes unchanged (same byte count).

**Why.** Owner saw the overlay and asked for a thicker line.

**Evidence.** Served + probe at the Hondsbesch view: the same bike canvas now
draws 1,865 sampled pixels versus 1,391 at weight 1.6; page still renders
normally, no JS errors. New build hashes (also updated in HANDOFF section 2):

```
80cbbec... -> f931cc948162f70d2bd7d35df846f641affb83c53236082cc19dd2f102de54e8  v2
285cd6c... -> 2b63bf1bb812ac154e15be2631767e616b3c5b841e45e55fb75149e226cfc28f  mobile
```

### B8. Layers panel could not scroll to Sites (owner: major bug)

**What.** Base CSS in both builds: `.panel` gained
`display:flex; flex-direction:column; max-height:calc(100vh - 24px)` and
`.panel-body` gained `overflow-y:auto; overscroll-behavior:contain`. The
mobile override (52vh cap, already-scrolling body) sits later in the
stylesheet and still wins, so mobile behaves as before.

**Why.** Owner reported being unable to reach the Sites section at the bottom
of the Layers panel. Root cause: the desktop panel had no height cap and no
scroll container at all, and the app page itself never scrolls, so once the
panel content was taller than the window its bottom was physically off-screen
and unreachable. The bike rows added about 90 px on top of an already tall
panel; with them it overflows on common laptop windows.

**Evidence.** Headless Edge, viewport 1256 x 628:

- before: panel bottom 1041 px, Sites chips 898..1028 px, overflowY visible,
  no scrollable ancestor, unreachable.
- after: panel capped at 604 px (`calc(100vh - 24px)`), body scrolls
  (scrollH 1050 vs clientH 568); scrolling to the bottom puts the Sites chips
  at 475..605 px inside the window, visible.
- tall window 1896 x 1108: panel fits (1029 < 1084), no scroll needed,
  Sites visible.
- mobile 430 x 932 equivalent: max-height still computes from 52vh,
  Find-section Sites reachable, no regression.

Hashes updated in HANDOFF section 2.

### B9. Phone testing tooling (owner: build does not load in iPhone Safari)

**What.**

- New helper `tests\allow-port-8000.cmd`: one-click, self-elevating script
  that adds an explicit inbound firewall rule for TCP 8000 (private
  networks). The removal command is in its comments.
- `tests\serve-windows.cmd`: added the "type the URL with the http:// part"
  tip, a pointer to the helper, and a guard that detects port 8000 already
  in use and says so instead of failing with a Python error.

**Why.** Owner reports the build does not load in Safari on the iPhone, no
matter what. PC-side checks on this machine: server listens on 0.0.0.0:8000,
both builds fetch over the LAN IP with HTTP 200, Windows Firewall already
has python.exe allow rules (Private profile), Wi-Fi network profile is
Private. No explicit port-8000 rule exists, and the agent session is not
admin, so a self-elevating helper was needed. The remaining suspects are on
the phone/network side (URL without http://, guest SSID, VPN).

**Evidence.** Ran the real batch file: with the port busy it prints the new
"Port 8000 is already in use" message and exits; with the port free it prints
the Phone URL (192.168.178.29) and serves the mobile build over the LAN
address (HTTP 200, 2,354,476 bytes). Build files unchanged in this entry -
no new hashes.

### B10. Phone panel could not scroll (owner: tabs stuck, cannot get down)

**What.** `min-height:0` added to `.panel-body`: once in the shared base rule
(both builds), once in the mobile override (mobile build). One property.

**Why.** Owner reports the phone panel cannot be scrolled down to its lower
content. On iOS Safari a scrollable flex item whose `min-height` stays `auto`
refuses to shrink below its content height; when the panel content is taller
than the 52vh cap, the body neither shrinks nor scrolls and the bottom of the
panel (legend chips, hints) stays clipped and unreachable. Chromium shrinks
such items per spec, so the earlier tests (which all fit within the panel)
did not catch it. `min-height:0` is the canonical cross-browser fix.

**Evidence.** Headless Edge, mobile build, forced-overflow window 430x700
(52vh = 364 px < content 418 px):

- after: body clientH 316 vs scrollH 418, scrollTop reaches 102 (full range);
  at the bottom the legend chips (424..480) and the ruins hint (489..533) sit
  inside the panel and are visible.
- regression, 430x932 where content fits: scrollH == clientH == 418, panel
  still hugs its content (height 420), min-height reports 0px, no visual
  change.

New hashes in HANDOFF section 2.

### B11. Handoff updated for publishing + workspace cleanup (owner request)

**What.**

- HANDOFF.md section 2 rewritten: absolute paths for every file, and a history
  note that the older builds were deleted.
- HANDOFF.md gained section 11: the owner's wish to publish the map as a
  GitHub Pages site, with costs, the missing `index.html`, and the licensing
  cautions (Esri imagery, Nominatim).
- AGENTS.md: the "originals must not be overwritten" bullet updated (they are
  gone), plus a do-not-publish-without-asking bullet.
- Deleted the previous build files and the macOS-only helper scripts:
  `Plans\luxembourg-wallonie-lidar-v2.html`,
  `Plans\luxembourg-wallonie-lidar-mobile.html`,
  `Plans\serve.command`, `Plans\stop.command`.

**Why.** Owner asked to record the GitHub Pages plan in the handoff, to add
where all the files live, and to delete the previous files while keeping the
newer ones (the tests builds), because the newer ones are the ones that fit.

**Evidence.** `tests\` now holds the only builds; byte sizes and hashes are
unchanged from B10. The previous agent's docs in `Plans\handoff\` were KEPT on
purpose: they document the architecture and the id contract. Say the word to
remove them too.

---

## Verification summary for change set B

All runs: builds served over http on localhost:8000, driven with this
machine's headless Edge (the sandboxed browser cannot open localhost).

| Check | Result |
| --- | --- |
| mapdata parses in both builds | yes, 8,478 points, 1,026 bunkers |
| bikedata parses in both builds | yes, 23,764 lines |
| no leftover setMaxBounds / viscosity | 0 occurrences |
| new element ids present exactly once | bikes, bikesLbl, bikeHint, bikedata |
| source diff vs previous build | exactly the 15 intended regions per file |
| drag at both-mode, zoom 9, pre-fix | 420 px drag clamped to 86 px (bug reproduced) |
| drag at both-mode, post-fix | full 420 px movement, no clamp |
| Hondsbesch bunker view | green dot + label rendered, 18 dots total |
| bike canvas at that view | 1,391 px drawn, checkbox off/on cycles |
| sources page, deskop + mobile | all new figures render |
| mobile boot | rail 5 buttons, legend 6 chips, bikes switch present, no JS errors |

Known test limitations (do not overclaim later): phone never tested on real
hardware; bike canvas pixel re-checks are flaky under headless virtual time;
the drag reproduction needed real-motion runs and worked reproducibly once
clamped vs always free post-fix.

## Change set C - published as a GitHub Pages site (owner green-light)

### C1. New `site/` folder, the GitHub Pages deliverable

**What.** A new folder `Plans/site/` was created and is now a git repository
pushed to GitHub. Contents:

```
site/
  index.html                              landing page, new file
  luxembourg-wallonie-lidar-v2.html       copy of the tests/ build, byte identical
  luxembourg-wallonie-lidar-mobile.html   copy of the tests/ build, byte identical
  README.md                               project readme, new file
  .nojekyll                               stops Pages processing the data blocks
  .gitattributes                          "* -text", keeps the builds byte exact
  docs/                                   HANDOFF, ARCHITECTURE, CHANGES, AGENTS
```

The builds in `tests/` are untouched and remain the working copies. The site
copies are refreshed by the new `Plans/sync-site.cmd`.

**Where.** Repo `A097MPRUS/luxembourg-wallonie-lidar`, public, branch `main`,
Pages serving from `main` at `/`. Live at
`https://a097mprus.github.io/luxembourg-wallonie-lidar/`.

**Why.** HANDOFF section 11 was the plan and said nothing was to be created
until the owner green-lit it. The owner green-lit it on 2026-09-20 and chose
GitHub Pages, public, under the A097MPRUS account.

**Evidence.** Pages build `built` in 22.2 s, no error. Live site loaded in a
real browser at 1280x900: landing page renders, `Open the map` points at the
v2 build, the map itself loads 48 of 48 tiles over HTTPS with zero console
errors and zero mixed content warnings, `mapdata` parses to 8,478 points.

### C2. `index.html`, the landing page

**What.** A new single page in the project's own visual language: the same
IBM Plex Sans/Mono, the same light and dark tokens read out of the build
(`--ground`, `--panel`, `--ink`, `--line`, `--accent`), the same radii. House
style respected: no em dashes, no pill buttons, no gradients, no purple, no
emoji, no scroll animations.

It carries a primary `Open the map` action, explicit links to both builds, a
`recommended` marker on the one matching the current device, the layer table,
the point counts with colour swatches, the data sources with the Esri
"not open data" note, the no-cookies statement, and the Scan beta caveat.

**Device routing.** `matchMedia("(pointer:coarse)") && innerWidth < 900` sends
the primary action to the mobile build. It is a link change, not a redirect,
so either build is always reachable. Flipping it to an automatic redirect is a
two line change if the owner prefers landing straight in the map.

**Figures.** Every number on the page was read out of the shipped data blocks
with a script, not copied from a doc: ruins 8,478 (kinds 1,348 / 1,026 / 2,341
/ 3,153 / 139 / 471; LU 2,162, WAL 6,316), bike lines 23,764 (LU 7,458,
WAL 16,306), places 4,583, shapes 2,550. Identical in both builds. This also
re-confirmed the recorded hashes: v2 `dd710c1d...` 2,344,433 bytes, mobile
`805131a2...` 2,354,504 bytes.

**Two bugs found and fixed while testing, both in the new page only:**

1. Both `recommended` chips showed at once. `.rec{display:inline-block}` beat
   the user agent's `[hidden]{display:none}`. Fixed with an explicit
   `[hidden]{display:none !important}`, the same guard the builds already use.
   Verified: on desktop `recD.hidden=false`, `recM.hidden=true`; at 375 px the
   pair is reversed and `goHref` becomes the mobile build.
2. At 375 px the chip wrapped onto its own line under the wrong link. The link
   and its chip are now wrapped in a `white-space:nowrap` span.

**Evidence.** Served locally on port 8011 and driven in a real browser. All
eight linked paths return 200. No horizontal overflow at 375 px. Desktop build
from the site folder: 48 tiles, 15 panes, `ui` `swipe` `bikes` present, 8,478
points, no console errors. Mobile build from the site folder: icon rail, bottom
bar, panel, bikes toggle, no overflow, no console errors.

### C3. `README.md` and `sync-site.cmd`

**What.** A README covering the two builds, how to run without a build step,
the feature table, the point counts, the data sources with attribution, the
privacy statement, the caveats (Scan is beta, the four dead ends in automatic
detection, phone untested on hardware, Nominatim is personal scale), and links
to the four docs. `Plans/sync-site.cmd` re-copies the builds and docs from
`tests/` and `claude/` into `site/` and prints the commit and push commands.

**Why.** The site copies would otherwise drift silently from the working
builds, which is exactly the class of problem this change log exists to stop.

### C4. Git identity used for the public repo

**What.** The repo uses a local git identity of
`A097MPRUS <314761645+A097MPRUS@users.noreply.github.com>`, set on the repo
only, not globally.

**Why.** Commits on a public repo are public. The GitHub noreply address keeps
the owner's real email out of the public commit history. No global git config
was touched.

### C5. Things the owner should decide

- **Esri World Imagery is not open data.** The site is now public and serves
  that layer. This was flagged in HANDOFF section 11 before publishing and is
  a licensing question, not a technical one. The open alternative is the
  geoportail.lu ortho layers. Nothing was changed without asking.
- **Nominatim** address search is now reachable by the public. Fine at the
  current scale, needs a plan if the site gets traffic.
- **Custom domain** is supported and free to point at Pages; only the domain
  registration costs anything.

### C6. "Wallonie" renamed to "Wallonia" in English-facing text (2026-09-21)

**What.** In both builds: the page `<title>` (desktop and mobile), the header
comment, the CSS section comment, the region labels `"Wallonia"` and
`"Luxembourg + Wallonia"`, and the region name on search hits. In the site:
the landing page title and heading, the README heading and Regions row.
Exactly one occurrence of each replaced per build.

**Not changed, on purpose.** French strings ("en Wallonie", "de la
Wallonie"), German ("Wallonien"), the official agency name "Service public de
Wallonie", the domain `geoservices.wallonie.be`, and the file names, because
renaming the files would break the published URLs.

**Why.** Owner: it says wallonie instead of wallonia.

**Evidence.** Both builds served locally: titles read "Luxembourg & Wallonia
LiDAR" and "... Mobile", the region menu reads Luxembourg / Wallonia /
Luxembourg + Wallonia, no console errors. Sizes unchanged (same letter
count); new sha256 prefixes v2 `f75b407a`, mobile `4a403b8e`.

### C7. Repo made private, GitHub Pages switched off (2026-09-21)

**What.** `A097MPRUS/luxembourg-wallonie-lidar` visibility public -> private
(`gh repo edit --visibility private`). Making it private also removed the
Pages configuration (`GET /pages` now 404). No files changed.

**Why.** Owner asked immediately, worried that Esri World Imagery not being
open data means legal exposure.

**Evidence.** API reports `private: true`. A fresh uncached request to the
site returns HTTP 404 with `X-Cache: MISS`; the bare root URL still returned
the old page from GitHub's CDN cache for a few minutes afterwards.

**To undo** (only if the owner asks): make the repo public again and
re-enable Pages from `main` at `/`. Resolve the imagery licence first.

### C8. Repo made public again, Pages re-enabled (2026-09-21)

**What.** Reverted C7: visibility private -> public, Pages re-enabled from
`main` at `/`.

**Why.** C7 was a misreading. The owner's instruction was conditional: make it
private only if the Esri imagery made the project illegal. The answer was
that it does not (standard tile use with the on-map attribution), so the
takedown should never have happened. Owner: "put it back up".

**Evidence.** Pages status `built`; `index.html` and the v2 build both return
HTTP 200 on fresh requests.

## Change set D - government aerial photos replace Esri World Imagery (2026-09-21)

Owner asked which imagery would be safer than Esri World Imagery (not open
data; the map loaded its tiles without an ArcGIS account). Recommended the two
governments' own orthophotos. Owner: build it as an alpha in `Plans\alpha\`,
do not touch the handoff until confirmed. After review: "this seems good to
me", replace the builds in `tests\`, delete all Esri World Imagery, update
GitHub, the website and the handoff.

### D1. The imagery layer (both builds)

**What.** Removed the Esri `L.tileLayer` (server.arcgisonline.com
World_Imagery). Added in its place, in pane `sat` (z 350, swipe clip
unchanged):
- `luOrtho(id)`: ACT WMTS, `wmts{1-4}.geoportail.lu/opendata/wmts/<id>/
  GLOBAL_WEBMERCATOR_4_V3/{z}/{x}/{y}.jpeg`, `maxNativeZoom` 20, LU bounds.
- `walOrtho(svc, png)`: SPW WMS 1.3.0, `geoservices.wallonie.be/arcgis/
  services/IMAGERIE/<svc>/MapServer/WMSServer`, layer `0`, EPSG:3857, WAL
  bounds; JPEG normally, transparent PNG when stacked in the combined region.
- `IMAGERY` table of four options, `imgDef`, `imgYear`, `setImagery(id)`.
  `setImagery` runs at boot and from `setRegion`; the chosen option persists
  across region switches.

**Picker.** New `<div class="grp" id="imgGrp">` right after `layerGrp` in both
builds (inside the Layers page on the phone). `buildImagery()` renders radios
`im-summer`, `im-leafoff`, `im-y2001`, `im-y1970` in the same `.opt` style as
the relief list, with the year in the `.yr` column. Called from boot,
`setRegion` and `setLang` (the `buildLayers(); buildSites();` triple became
`buildLayers(); buildImagery(); buildSites();` in all three places).
Translations in all four languages: `aerial`, `imgSummer`, `imgLeafoff`,
`img2001`, `img1970` (EN Aerial photo / Summer / Winter / spring / Around
2001 / Around 1970; FR Photo aérienne / Été / Hiver / printemps / Vers 2001 /
Vers 1970; DE Luftbild / Sommer / Winter / Frühling / Um 2001 / Um 1970;
LB Loftbild / Summer / Wanter / Fréijoer / Ëm 2001 / Ëm 1970).

| Option | Luxembourg | Wallonia |
| --- | --- | --- |
| Summer (default) | `ortho_2025` | `ORTHO_2023_ETE` |
| Winter / spring | `ortho_2025_winter` | `ORTHO_2026_PRINTEMPS` |
| Around 2001 | `ortho_2001` | `ORTHO_2001_2003` |
| Around 1970 | `ortho_1967` | `ORTHO_1971` |

**Attribution.** `ATTR_LU` / `ATTR_BE` now read "Relief, orthophoto : © ACT /
geoportail.lu" and "Relief, orthophoto : © SPW / geoportail.wallonie.be";
`ATTR_IMG` (Esri, Maxar, Earthstar Geographics) removed and `attribLine`
simplified.

**Legal copy.** Third-party list: Esri entry removed; ACT and SPW entries now
say "LiDAR relief ... and aerial photos". Source licences: the Esri paragraph
replaced by one paragraph per region (see D6 for the wording). Cookies page:
"Esri" dropped from the list of services that set no cookies. Data sources
table: the Esri row replaced by "Aerial photos, Luxembourg | ACT,
geoportail.lu | 1967, 2001, 2025 | varies by year" and "Aerial photos,
Wallonia | SPW, geoportail.wallonie.be | 1971, 2001-2003, 2023, 2026 | varies
by year".

**Kept on purpose.** `esriGeometryPoint` in `elevBE`: a parameter name of
SPW's own ArcGIS elevation service, not Esri imagery. Removing it would break
the Wallonia altitude readout.

### D2. Coverage finding: SPW summer 2025 is unusable

The alpha first used `ORTHO_2025_ETE` for Wallonia's summer. Testing tile
content (not just load events) showed white "no photo" tiles. A coverage sweep
at 10 Walloon towns (Tournai, Mons, Namur, Liège, Eupen, Dinant, Bastogne,
Arlon, Chimay, Wavre):

| Service | Photo at |
| --- | --- |
| ORTHO_LAST, ORTHO_2026_PRINTEMPS, ORTHO_2025_PRINTEMPS, ORTHO_2023_ETE, ORTHO_2001_2003, ORTHO_1971 | 10 / 10 |
| ORTHO_2024 | 7 / 10 (no Tournai, Arlon, Chimay) |
| ORTHO_2025_ETE | 0 / 10 |

`ORTHO_LAST` is pixel-identical (mean difference 0.0) to `ORTHO_2023_ETE` at
Namur, Bastogne and Tournai, so Wallonia's summer uses the dated
`ORTHO_2023_ETE` service and is labelled 2023; a dated service keeps the label
true when SPW moves `ORTHO_LAST` on. The four ACT layers were checked at
Clervaux, Wiltz, Echternach, Esch, Remich, Steinfort and Luxembourg City:
photo everywhere. `ortho_latest` is byte-identical to `ortho_2025` today.
Speed: 12 SPW WMS tiles in 200 to 350 ms, dated or `ORTHO_LAST` alike.

### D3. Combined region: stacking at the border

ACT tiles are white outside Luxembourg; SPW tiles as transparent PNG are
transparent where SPW has no photo. The combined region stacks ACT below and
SPW above. Pixel test at four border tiles (Steinfort/Arlon, Martelange,
Troisvierges/Gouvy, Rodange/Athus): ACT alone 38 to 89% white, stacked result
0% white, 0% empty.

**Known quirk.** SPW's mosaic runs past the Walloon border: fully transparent
only from lon 5.94 at lat 49.66 (Steinfort), within about 1 km at Martelange.
In the combined view a thin strip on the Luxembourg side shows the Walloon
photo. Single-region views are unaffected. Not fixed; would need clipping SPW
tiles to the Walloon polygon.

### D4. Licences checked at the source

- ACT on data.public.lu: `Orthophoto officielle ... édition 2001`, `édition
  été 2025`, `édition hiver 2025` all `cc-zero`.
- SPW: "Conditions d'accès et d'utilisation des services web géographiques de
  visualisation du SPW", v1.1 of 3 August 2016 (LicServicesSPW.pdf, linked
  from the ORTHO_LAST catalogue record): free access and use for any user
  (art. 3), no disproportionate load (art. 5 §1), do not hide or remove the
  source mention (art. 5 §3), do not alter the data (art. 5 §4).

### D5. Promotion into tests\ and the site

`alpha\` builds copied into `tests\` with only the two alpha markers removed
(`<title>` "(alpha)" and "ALPHA" in the header comment); `diff` shows exactly
those two lines per file. The old Esri builds remain in the `site\` git
history (last in commit `4ffc6d9`). `site\` builds synced; `site\index.html`
and `site\README.md` rewritten wherever they mentioned Esri (subtitle,
"What it shows" row, data source rows, licence paragraph) and gained the line
"Every map layer comes from a public body or from OpenStreetMap. No commercial
imagery is used." README caveats gained the border strip.

### D6. Correction found while documenting: the 1967 licence

The alpha's legal copy said all Luxembourg orthophotos are CC0. Checking each
dataset: 2001 and both 2025 editions are `cc-zero`, but `Orthophoto 1967`
(ACT, data.public.lu) has licence `notspecified`. Wording corrected in both
builds (and in `alpha\`, the landing page, the README and HANDOFF) to: "The
2001 and 2025 editions are published on data.public.lu under CC0, which
places them in the public domain. The 1967 edition is published there by ACT
without a stated licence. All are credited." The 1967 option is kept (the
owner approved it in the alpha); dropping it for Luxembourg is a one-row
change, listed in HANDOFF open items.

### D7. Tooling

`work\make_alpha.py` is the exact transformation (every replacement asserts
its expected count). Reproducibility check: applied to the pre-D builds, then
with the alpha markers removed, it produces the current `tests\` builds byte
for byte (both files: True).

### D8. Docs

HANDOFF fully revised (new section 8 on aerial photos, updated file map,
hashes, sources, open items, site section). AGENTS gained the no commercial
imagery rule and the new ids. `handoff/ARCHITECTURE.md` gained the new ids,
functions and the `sat` pane contents. `handoff/HANDOFF.md` marks its Esri
source row as removed so nobody re-adds it from the old baseline.

### Evidence (all 2026-09-21)

- Headless Edge, fresh profile, reduced motion, probe on a COPY of each build:
  Luxembourg x 4 options and Wallonia x 4 options, every sampled tile a real
  photo (16/16 per option in LU, 16/16 or 10/10 in WAL), 0 tile errors;
  combined region 32/32 tiles loaded, 0 errors, both services present.
- French picker labels render; legal templates, `#ui` and the attribution
  contain no Esri, Maxar, Earthstar or arcgisonline (both builds).
- Data sources table rows render as intended (both builds).
- Phone build at 375 px: Layers panel 14 to 309 px wide, picker visible, body
  scrolls, no horizontal overflow, no JS errors.
- Final builds: v2 2,348,169 bytes sha256 `b7b61e82...`, mobile 2,358,242
  bytes `975884fa...`.

## Change set E - Luxembourg's 1967 aerial photo removed (2026-09-21)

**What.** Owner: "yeah remove 1967 please", after D6 showed that ACT's
`Orthophoto 1967` has licence `notspecified` on data.public.lu while every
other Luxembourg photo used is CC0.

- `IMAGERY` row `y1970`: `lu: "ortho_1967", luYr: "1967"` became
  `lu: null, luYr: null`. Wallonia keeps `ORTHO_1971`.
- New `imgAvail(def)`: an option with no photo for the current region is not
  offered. Luxembourg region: three options. Wallonia: four. Combined: four,
  with "Around 1970" showing year "WAL 1971" and only SPW tiles.
- `setImagery`: if the chosen option is not available after a region switch,
  it falls back to Summer and ticks that radio (without the tick the picker
  showed nothing selected, because it is rebuilt before `setImagery` runs).
- Legal copy: Luxembourg licence paragraph now "ACT orthophotos, the 2001 and
  2025 editions, published on data.public.lu under CC0, which places them in
  the public domain. Credited anyway."; Data sources row "2001, 2025".
- Site landing page and README: 1967 and its licence caveat removed; the
  picker is described as "around 1970 in Wallonia only".

**Mishap, recorded for honesty.** The first run of the patch script opened
the desktop build for writing before patching, then stopped on an over-broad
check ("1967" also occurs as digits inside the embedded data blocks). That left
`tests/luxembourg-wallonie-lidar-v2.html` empty for a moment. It was restored
at once from a copy taken just before the run and verified byte-identical
(sha256 `b7b61e82...`, identical to the `site/` copy) before anything else
happened. The script now patches fully in memory and writes only on success,
and its check ignores the JSON data blocks. Script: `work/drop_1967.py`.

**Evidence.** Headless Edge probe (fresh profile, reduced motion) on copies
of both builds: Luxembourg shows Summer 2025 / Winter-spring 2025 / Around
2001 and requests only `ortho_2025` (24 tiles, 0 errors); Wallonia shows all
four with 1971, and 1970 loads `ORTHO_1971` (30 tiles, 0 errors); switching
Wallonia-1970 to Luxembourg falls back to Summer with its radio ticked;
combined 1970 loads only `ORTHO_1971` (28 tiles, 0 errors); legal templates
contain the new CC0 line and no "1967". Same results on the phone build.
`1967` no longer appears anywhere outside the data blocks.
Builds: v2 2,348,671 bytes sha256 `92fe2b43...`; mobile 2,358,744 bytes
`16020f91...`. `alpha/` still has 1967 and is marked superseded in HANDOFF.

## Rebuild recipe (if the data ever needs regenerating)

- Raw pulls: `work/*.overpass` files; responses in `work/*.json`.
- Scripts: `work/build3.py` (writes both builds), `work/process_bike.py`,
  `work/merge_bunkers.py`, `work/build2.py` (change set A), `work/verify_*`.
- Overpass is rate limited per IP: space queries out, one query per slot,
  no loops longer than ~2 minutes in a single shell call.
