# Padelmiðstøðin — website prototype

Design prototype for **padelm.fo** (Padelmiðstøðin, Á Hjalla 2, Hoyvík).

Open [`prototype/index.html`](prototype/index.html) directly in a browser — no build step.
Tailwind (Play CDN) and MapLibre GL load from CDNs, so an internet connection is needed
for styling and the map. Browsers without WebGL get a static map image automatically.

- **`prototype/subpage.html`** — generic subpage template (page header with breadcrumb,
  article content, booking band) sharing the front page's nav, footer, and design system.
- **`prototype/DESIGN.md`** — design tokens, typography plan, component rules, asset
  inventory, and the porting checklist.
- **`prototype/assets/`** — all images (WebP) and extracted brand vectors (SVG).

The production site will be built on Laravel + Statamic with a real Tailwind build;
this prototype is the visual reference for that port.
