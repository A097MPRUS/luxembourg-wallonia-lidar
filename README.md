# Luxembourg & Wallonia LiDAR

A LiDAR prospecting map for **Luxembourg** and **Wallonia (Belgium)**. It puts high resolution
shaded relief derived from the two national LiDAR surveys next to the governments' own aerial
photos, split by a draggable swipe divider, for finding ruined and abandoned structures, especially under forest
canopy where aerial imagery shows nothing but treetops.

Live site: **https://a097mprus.github.io/luxembourg-wallonie-lidar/**

## Builds

| File | For |
| --- | --- |
| `luxembourg-wallonie-lidar-v2.html` | Desktop |
| `luxembourg-wallonie-lidar-mobile.html` | Phone, icon rail chrome |

Each build is a **single self contained HTML file**. No server, no build step, no database, no
package manager, no accounts, no API keys. Open the file directly, or serve the folder:

```bash
python -m http.server 8000
```

Then `http://localhost:8000/`. Serving over HTTP is required to test the Scan tool, because it
reads pixels from cross origin tiles through a canvas.

The only library is Leaflet 1.9.4 from cdnjs, and its CSS is inlined in each file on purpose.

## Features

| Feature | Notes |
| --- | --- |
| Swipe compare | Relief left, aerial photo right. Drag the divider, or use arrow keys and Home/End. |
| Aerial photos | Government orthophotos: ACT in Luxembourg, SPW in Wallonia. Summer, winter or spring, around 2001; around 1970 in Wallonia only. |
| Regions | Luxembourg, Wallonia, or both. Everything outside is masked. |
| Relief layers | Terrain (DTM), Surface (DSM), Above ground (DSM minus DTM, computed in browser). |
| Ruins and abandoned | 8,478 OpenStreetMap points across six kinds. |
| Bike trails | 23,764 lines of cycleways and signed routes. |
| Place names | 4,583 settlements. Cities from zoom 8, hamlets from zoom 13. No roads at any zoom. |
| Building footprints | Official cadastre. ACT for Luxembourg, SPW PICC for Wallonia, from zoom 15. |
| Condition outlines | 2,550 OSM building outlines tagged ruined, abandoned, disused or bunker, from zoom 14. |
| Scan (beta) | Finds structures with no cadastral footprint. See the caveats below. |
| Search | Coordinates (decimal, DMS, comma decimals), embedded town names, Nominatim addresses. |
| Elevation readout | Real 50 cm LiDAR in Wallonia, Copernicus GLO-90 at 90 m in Luxembourg. |
| Languages | EN, FR, DE, LB. |
| Legal pages | Privacy, Terms, Cookies, Data sources. Hash routed: `#privacy`, `#terms`, `#cookies`, `#sources`. |

### Point counts

| Colour | Hex | Kind | Count |
| --- | --- | --- | --- |
| Red | `#d1495b` | Ruins | 1,348 |
| Green | `#2f6f4e` | Bunkers | 1,026 |
| Orange | `#e07a1f` | Abandoned | 2,341 |
| Yellow | `#e8c33a` | Disused, off by default | 3,153 |
| Near black | `#33383d` | Mine shafts | 139 |
| Olive | `#8a7a1f` | Brownfield, off by default | 471 |
| Dark brown | `#5b3a1e` | Bike trails, lines not dots | 23,764 |

Luxembourg 2,162 points, Wallonia 6,316. Raw Overpass pull was 10,046, collapsed to 8,478 by
merging same kind points within 35 m. All figures verified against the shipped data blocks.

## Data sources

| Layer | Provider | Facts |
| --- | --- | --- |
| LU relief | ACT, geoportail.lu | Flown February 2019, 15 pts/m², 50 cm raster, ±3 cm h / ±6 cm v. |
| LU buildings | ACT, geoportail.lu | Cadastral footprints. |
| WAL relief | SPW, geoservices.wallonie.be | LiDAR 2021 to 2022, 50 cm. |
| WAL buildings | SPW PICC | Building footprints and other structures. |
| LU aerial photos | ACT, geoportail.lu | 2001, 2025 summer, 2025 winter. Public domain (CC0 on data.public.lu). |
| WAL aerial photos | SPW, geoservices.wallonie.be | 1971, 2001-2003, 2023 summer, 2026 spring. Free use under SPW's web service conditions. |
| Elevation (LU) | Open-Meteo, Copernicus DEM GLO-90 | 90 m. Not the LiDAR. |
| Elevation (WAL) | SPW MNT 2021-2022 | Real 50 cm terrain model value. |
| Geocoding | Nominatim, OpenStreetMap | Only when a query is not a coordinate and not an embedded town. |
| Points and lines | OpenStreetMap contributors via Overpass | Pulled 2026-09-20. |

OpenStreetMap data is © OpenStreetMap contributors, available under the Open Database License.
The LiDAR relief, cadastre and aerial photo layers all come from ACT and SPW. Luxembourg's photos are
public domain (CC0). SPW lets anyone use its web map services free of charge, provided the source
stays credited, the images are not altered, and the services are not overloaded. **No commercial
imagery is used.**

## Privacy

The page sets no cookies and uses no `localStorage`, `sessionStorage` or IndexedDB. Tiles,
elevation values and address lookups are requested directly from the providers above, so those
providers see the requests. One third party cookie appears, `BIGipServer~PRODUCTION~PO_GEOSERVICES_SSL`
from `geoservices.wallonie.be`, an F5 load balancer session cookie that is HttpOnly, Secure and
session only.

## Caveats

- **Scan is beta.** It differences the surface and terrain hillshades, thresholds, labels connected
  components, then filters on size, aspect, rectangularity and interior smoothness. It saturates
  under closed canopy, because the canopy itself is above ground. It is not a survey instrument.
- **Automatic building detection has already hit four dead ends**, all measured. Read the project's
  handoff notes before trying again. The short version:
  neither provider publishes raw elevation over the web, hillshade edge detection fails because a
  single sun azimuth hides walls parallel to the light, and Luxembourg's forest floor is full of
  rectilinear earthworks that swamp any heuristic.
- **The phone build has not been tested on real hardware.** It boots clean in headless testing.
- Address search uses Nominatim's public API, which is sized for personal use.
- In the combined Luxembourg + Wallonia view, SPW's photos run about 1 to 2 km past the Walloon
  border, so a thin strip on the Luxembourg side shows the Walloon photo. Single region views are
  unaffected.

## Documentation

| File | Contents |
| --- | --- |
| [docs/AGENTS.md](docs/AGENTS.md) | Working rules for anyone, human or agent, editing the builds. |

## Working on it

The two builds share one JavaScript body and resolve every control through `getElementById`, so
**element ids must stay identical across both files**, and changes apply to both unless they are
genuinely layout only. See `docs/AGENTS.md` before editing.

The builds in this repository are copies. The working builds live in `Plans\tests\` on the owner's
machine. After changing one, run `sync-site.cmd` in `Plans\` to refresh the copies here, then
commit and push.
