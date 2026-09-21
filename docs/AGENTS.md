# Operating rules for agents working on this project

Short rules for anyone, human or agent, editing the map in this repository.

## The builds

- Two deliverables, both single self-contained HTML files, no build step, no
  backend: `luxembourg-wallonia-lidar-v2.html` (desktop) and
  `luxembourg-wallonia-lidar-mobile.html` (phone). Do not overwrite or delete
  them without the owner's approval.
- Both builds share one JavaScript body and find every control by
  `getElementById`. Keep element ids identical across both files. Apply every
  change to both unless it is genuinely layout-only.
- Ids that must exist in both builds include `bikes`, `bikesLbl`, `bikeHint`,
  `bikedata`, and `imgGrp` (the aerial photo picker) with its generated radios
  `im-summer`, `im-leafoff`, `im-y2001`, `im-y1970`. Keep them in both builds
  or remove them from both.
- Only dependency is Leaflet 1.9.4 from cdnjs. Keep it that way. Its CSS is
  inlined on purpose.
- No API keys, no env vars, no backend, no cookies set by the page. Do not add
  a secrets manager, a `.env`, or a cookie banner.

## Data and imagery

- **No commercial imagery.** Every layer must come from a public body (ACT for
  Luxembourg, SPW for Wallonia) or OpenStreetMap. Do not add Esri, Google,
  Bing, Mapbox or similar.
- Aerial photos in use: Luxembourg `ortho_2001`, `ortho_2025`,
  `ortho_2025_winter` (ACT, CC0); Wallonia `ORTHO_2001_2003`,
  `ORTHO_2023_ETE`, `ORTHO_2026_PRINTEMPS`, `ORTHO_1971` (SPW web services,
  free use with the source credited). Before adding any year or source, check
  its licence at the source and its coverage across the whole territory; some
  SPW campaigns are partial (summer 2025 covers none of ten test towns).
- Keep the source credits in the map attribution. SPW's terms forbid removing
  them.
- Never display a figure you cannot trace to the data or a provider's
  metadata. When data changes, update every displayed figure in the same pass
  (the Data sources table, the ruins paragraph, the legend).

## Colours

Fixed; do not reuse any of them for something else, and never introduce a
colour close to the neutral footprint blue.

| Colour | Hex | Meaning |
| --- | --- | --- |
| Red | #d1495b | Ruins |
| Green | #2f6f4e | Bunkers |
| Orange | #e07a1f | Abandoned |
| Yellow | #e8c33a | Disused (off by default) |
| Near-black grey | #33383d | Mine shafts |
| Olive-brown | #8a7a1f | Brownfield (off by default) |
| Dark brown | #5b3a1e | Bike trails overlay (lines, not dots) |

## House style

No em dashes in UI copy, no pill buttons, no gradients, no purple, no emoji
icons, no scroll animations, no invented metrics.

## Testing

- Test before claiming. Serve the folder with `python -m http.server` and
  drive the page in a browser; check the console.
- Headless Edge or Chrome works for scripted checks
  (`--headless=new --dump-dom --virtual-time-budget=60000 <url>`). Append any
  probe script to a COPY of a build, never to the build itself. Use a
  throwaway `--user-data-dir`.
- `--force-prefers-reduced-motion` makes region switches instant. Fly
  animations can stall under virtual time or in a hidden window. Canvas
  repaints wait on animation frames, so pixel re-checks after toggling are
  flaky; prefer two complementary checks.
- Check that tiles contain an actual photo, not just that they loaded: a
  service can return white "no data" tiles with HTTP 200.
- Overpass API is rate limited per IP (roughly one query slot per minute).
  Space queries and retry across mirrors.

## Publishing

- This repository is public and serves the site through GitHub Pages at
  https://a097mprus.github.io/luxembourg-wallonia-lidar/. Anything pushed here
  is public at once.
- Do not add anything else to the repository without asking the owner, and do
  not point a custom domain at it or change its visibility on your own.
- When in doubt, ask the owner rather than inventing.
