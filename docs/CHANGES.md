# Change log

Every change made to this project in this working session (2026-09-20, by Kun).
Newest last. One entry per change: what, where, why, and the evidence it works.
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

## Rebuild recipe (if the data ever needs regenerating)

- Raw pulls: `work/*.overpass` files; responses in `work/*.json`.
- Scripts: `work/build3.py` (writes both builds), `work/process_bike.py`,
  `work/merge_bunkers.py`, `work/build2.py` (change set A), `work/verify_*`.
- Overpass is rate limited per IP: space queries out, one query per slot,
  no loops longer than ~2 minutes in a single shell call.
