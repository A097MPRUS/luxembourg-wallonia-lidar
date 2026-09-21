# Operating rules for agents working on this project

Read `HANDOFF.md` first, then `CHANGES.md`. Short version:

- Two deliverables, both single self-contained HTML files, no build step, no
  backend. They live in `../tests/` and are now the only builds (the older
  copies in the parent folder were deleted on 2026-09-20 at the owner's
  request). Do not overwrite or delete them without the owner's approval.
- Both builds share one JavaScript body and find every control by
  `getElementById`. Keep element ids identical across both files. Apply every
  change to both unless it is genuinely layout-only.
- New ids added on 2026-09-20: `bikes`, `bikesLbl`, `bikeHint`, `bikedata`.
  Keep them in both builds or remove them from both.
- No API keys, no env vars, no backend, no cookies set by the page. Do not add
  a secrets manager, a `.env`, or a cookie banner. This has been verified.
- Only dependency is Leaflet 1.9.4 from cdnjs. Keep it that way. Its CSS is
  inlined on purpose.
- House style: no em dashes in UI copy, no pill buttons, no gradients, no
  purple, no emoji icons, no scroll animations, no invented metrics.
- Colours are fixed and charted in HANDOFF.md section 6. Never introduce a
  colour close to the neutral footprint blue, and never reuse a condition
  colour for something else.
- Never display a figure you cannot trace to the data or a provider's
  metadata. When data changes, hunt down every displayed figure in the same
  pass (the Data sources template, the ruins paragraph, the table cell).
- **Every change gets a detailed entry in `CHANGES.md`, and state changes go
  into `HANDOFF.md`, in the same pass.** The owner has asked for this
  explicitly, twice. Small changes count.
- Test before claiming. Serve the folder with `python -m http.server` and
  drive the page. The sandboxed browser cannot open localhost; headless Edge
  works:
  `"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
  --headless=new --dump-dom --virtual-time-budget=60000 <url>` with a probe
  script appended to a COPY of the build (never to the deliverable itself).
  Patterns for that live in `../work/probe3.js`.
- Headless quirks to know: `--force-prefers-reduced-motion` makes navigation
  instant (good for view checks) but remember the app's region switch only
  applies old maxBounds logic on animated flights; flyTo can stall under
  virtual time; canvas repaints wait on animation frames, so pixel re-checks
  after toggling are flaky. Prefer two complementary checks (DOM counts plus
  one pixel check) over a single fragile one.
- Overpass API is rate limited per IP (slots, roughly one per minute). Space
  queries, keep shell calls short, retry across mirrors. Details in
  CHANGES.md's rebuild recipe.
- When in doubt, ask the owner rather than inventing. The orange "Abandoned"
  chip collided with cadastre colours once before; the "disused" colour
  collided with the neutral footprint blue once before. Do not repeat either.
- The GitHub repo is now PRIVATE and Pages is OFF (2026-09-21, owner request,
  HANDOFF section 11). Do not make it public or re-enable Pages unasked.
  It was previously published as a GitHub Pages site:
  https://a097mprus.github.io/luxembourg-wallonie-lidar/ from the repo
  `A097MPRUS/luxembourg-wallonie-lidar`, served from `Plans\site\`.
  `site\` holds COPIES of the builds. `tests\` is still where they are edited.
  After changing a build, run `Plans\sync-site.cmd`, then commit and push from
  `site\`, in the same pass, or the live site serves the old version silently.
- If the repo is ever made public again, anything in `site\` is public the moment it is
  pushed. Do not add anything else to that repo without asking the owner, and
  do not point a custom domain or change the repo's visibility on your own.
