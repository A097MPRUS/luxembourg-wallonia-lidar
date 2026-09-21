# Architecture

> Updated 2026-09-21: the aerial imagery is now the ACT and SPW orthophotos (Esri World
> Imagery removed), with a picker in `#imgGrp`. The current state of the project is in
> `../claude/HANDOFF.md`; section 8 there covers the imagery in full.

## File shape

Each build is one HTML document in this order:

1. `<head>` — title, Google Fonts link, Leaflet 1.9.4 `<script>` from cdnjs, then one `<style>`
   containing **Leaflet's CSS inlined verbatim** followed by the app's own CSS. Leaflet's CSS is
   inlined because the original target blocked cross-origin stylesheets; it was kept so the file
   works offline apart from the tiles.
2. Body markup — `#app` > `#map`, the swipe divider, `#ui` chrome, then the legal overlay and four
   `<template id="doc-*">` blocks holding the legal documents.
3. `<script type="application/json" id="mapdata">` — the entire dataset, one line of JSON.
4. `<script>` — one IIFE containing all logic.

The mobile build is the same document with different chrome markup, an extra stylesheet appended,
and an icon-rail controller. **The JavaScript is otherwise identical**, which is only possible
because both files expose the same element ids.

Mobile chrome (replaced the original bottom sheet on 2026-09-20 at the owner's request):

- A vertical **icon rail** on the right edge: Layers, Overlay, Find, Scan, Info.
- Tapping a rail button opens `#panel` as a compact card anchored bottom-right, showing exactly one
  `<section class="pg" data-pg="…">` at a time. Tapping the same button again, or tapping the map,
  closes it. This keeps each screen short and the map visible.
- `#botbar` holds the coordinate readout **and** Leaflet's attribution element, which is moved into
  it at boot with `appendChild(attrib._container)` so the two stack as one block rather than being
  positioned independently and overlapping.

## The id contract

The IIFE resolves these by `getElementById`. Both builds must provide all of them:

```
app  map  swipe  knob  tagL  tagR
pill  pillFlag  pillName  langBtn  langCode  regionMenu  langMenu
panel  panelHead  panelTitle  panelBody  layerGrp  imgGrp
opacity  opVal  opLbl  mono  monoLbl  places  placesLbl
ovLbl  buildings  buildingsLbl  ovOpacity  ovOpVal  ovOpLbl
ruins  ruinsLbl  ruinsHint  legend
toolsLbl  scanBtn  scanBtnLbl  scanClear  scanOut  scanHint
sites  sitesLbl
finder  coordInput  goBtn  finderMsg  finderHits
readout  rLat  rLon  rAlt  rSrc
legal  legalTabs  legalBody  legalClose
mapdata
```

Mobile adds `rail`, `railLegal`, `botbar` and the rail labels `rbLayers`, `rbOver`, `rbFind`,
`rbScan`, `rbInfo`. Mobile hides `panelHead`, forces `panelBody` visible with CSS, and sets
`panelTouched = true` so the desktop `fitPanel()` logic never fights the rail.

## Leaflet panes and z-order

```
tilePane   200   relief (the monochrome CSS filter is applied to this pane)
sat        350   government aerial photos (ACT WMTS / SPW WMS), clipped to the right of the divider
build      420   cadastral footprints
mask       480   the outside-the-territory mask plus border outlines
places     520   settlement labels
marks      540   ruin markers
scan       560   scan result rectangles
markerPane 600   the search target reticle
--- outside the map ---
.swipe     620
#ui       1200   chrome. Must exceed Leaflet's control container at 1000.
.legal    1400
```

The swipe works by setting CSS `clip` on the `sat` pane in layer-point space, recomputed on
`move`, `zoom`, `zoomanim`, `viewreset` and `resize`. This is the leaflet-side-by-side technique.

## Data block schema

`JSON.parse(document.getElementById("mapdata").textContent)` yields:

```jsonc
{
  "borders": {
    "lu":   [ [ ring, ring… ] … ],   // GeoJSON-order [lng, lat], polygon then holes
    "be":   [ … ],                    // includes the Comines-Warneton exclave and Voeren hole
    "both": [ … ]                     // a true shapely union, NOT two overlaid polygons
  },
  "places": [ [lat, lon, name, rank, region] … ],  // rank 0 city … 4 hamlet; region 0=LU 1=WAL
  "ruins":  [ [lat, lon, kind, label, region] … ], // kind 0 ruin,1 bunker,2 abandoned,
                                                   // 3 disused,4 mineshaft,5 brownfield
  "shapes": [ [kind, region, minLat, minLon, maxLat, maxLon, flatOffsets] … ]
  // flatOffsets = [dLat0,dLon0,dLat1,dLon1,…] as integers in units of 1e-5 degrees,
  // measured from (minLat, minLon). Decode with ringOf() in the source.
}
```

`places` is sorted by rank so the label renderer can stop early. `ruins` is sorted by kind for the
same reason. Coordinates are rounded to 5 decimals (borders) or 4 (points).

The mask is drawn as **one polygon**: a world rectangle as the outer ring, with every territory
ring pushed in as a hole, filled `evenodd`. That makes enclaves and exclaves fall out for free.
For the combined region the union is used so no sliver appears along the shared border, and the
Luxembourg outline is then drawn on top so the internal border is still visible.

## Regenerating the data

The data was built with Python (shapely, numpy) from public sources. Rough recipe:

1. Borders: Nominatim `lookup?osm_ids=R2171347` (Luxembourg) and `R90348` (Wallonia) with
   `polygon_geojson=1`. Buffer(0), union for `both`, then `simplify(0.0002, preserve_topology=True)`
   which is about 22 m.
2. Places: Overpass, `node["place"~"^(city|town|village|hamlet|suburb|borough|quarter)$"]["name"]`
   over bbox `49.40,2.75,50.92,6.60`, then point-in-polygon filter to the two territories.
3. Ruins: Overpass, the tags listed in HANDOFF section 4, `out center tags`, same filter, then
   collapse same-kind points within 35 m keeping the longest label.
4. Overpass and Nominatim both **require a real User-Agent** or they return 406.

Use a venv; shapely and numpy are not in the system Python on this machine.

## Key functions in the IIFE

| Function | Does |
| --- | --- |
| `Wmts` | `L.TileLayer` subclass that zero-pads `{z}` to two digits for geoportail.lu. |
| `ArcGisDyn` | `L.TileLayer` subclass building ArcGIS `export` URLs per tile bbox. |
| `Ndsm` | `L.GridLayer` that loads the DTM and DSM tiles and paints their difference to a canvas. |
| `luOrtho(id)` / `walOrtho(svc, png)` | ACT orthophoto WMTS layer / SPW orthophoto WMS layer, both in pane `sat`. |
| `setImagery(id)` / `buildImagery()` | Swap the aerial photo per the `IMAGERY` table and region; render the picker into `#imgGrp`. |
| `inTerritory(lng, lat, polys)` | Even-odd ray cast across all rings. Used for masking, region routing and the elevation source choice. |
| `drawMask(key)` | Builds the world-with-holes mask and the border outlines. |
| `drawShapes()` | Draws the condition-coloured OSM building outlines from `DATA.shapes`. |
| `drawPlaces()` / `drawRuins()` | Viewport-filtered DOM label/marker renderers with greedy collision skipping. |
| `updateClip()` | The swipe. |
| `parseCoords(raw)` | Decimal, D/M, D/M/S, hemisphere letters, comma decimals. |
| `searchPlaces(q)` / `geocode(q)` | Embedded town search, then Nominatim fallback. |
| `runScan()` / `analyse()` / `paintScan()` | The Scan tool. |
| `setRegion(id, view)` | Swaps region, layers, mask, sites, bounds, and optionally flies to a view. |
| `setLang(code)` | Relabels every string from the `T` table. Add new UI strings there in all four languages. |

## Gotchas

- `map.getZoom()` can be fractional (`zoomSnap: 0.25`). Round before using it as a tile zoom.
- Clicking a Leaflet control with `element.click()` from a script does not always trigger its
  handler. Dispatch a real pointer event, or call the map API.
- `L.TileLayer.setOpacity` sets opacity on the layer container (`.leaflet-layer`), not on the
  individual tile images. Do not test it by reading a tile's opacity.
- `L.LayerGroup` has no `setOpacity`; iterate children.
- Canvas reads need `crossOrigin = "anonymous"` set **before** `src`, and a tainted canvas throws
  on `getImageData`, so wrap it.
- When overriding the desktop `.panel` rule for mobile, set `top:auto`. The desktop rule sets
  `top:12px`; leaving it in place alongside a `bottom` value stretches the panel between the two
  and it fills its `max-height` instead of hugging its content.
- Leaflet's `.leaflet-control-attribution` is content-box. `width:100%` plus its padding makes it
  wider than its parent and it overflows to the left. Set `box-sizing:border-box`.
