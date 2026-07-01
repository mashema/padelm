# Padelmiðstøðin — Design System Notes

Reference for porting the prototype (`prototype/index.html`) to the Laravel + Statamic + Tailwind build.
Source of truth for visuals: the Illustrator PDF in the client's asset package
(`Heimasíðuhugskot_padelmidstodin.fo.pdf`). Where the Claude Design project's `brand.css` and the PDF
disagree, the PDF wins.

---

## 1. Color tokens

Copy-paste ready for `tailwind.config.js`:

```js
colors: {
  // Core brand palette
  'pm-blue':  '#9ec2e8', // primary accent: blue CTAs, halftone dots, floor-plan outlines
  'pm-steel': '#31537c', // highlighted "HER", icons, court badges, section dividers
  'pm-ink':   '#131111', // headings + body on light backgrounds
  'pm-navy':  '#0f1e35', // footer / dark section base
  'pm-panel': '#e9f1fb', // light-blue full-width bands (features section)

  // Supporting text tints
  'pm-slate': '#35435c', // muted body text on light
  'pm-ice':   '#dde6f2', // body text on dark photo sections
  'pm-frost': '#c9d4e4', // body text on navy (footer)
  'pm-fog':   '#8ea3bf', // fine print on navy (copyright)

  // Brand palette, reserved (in the PDF/brand sheet but unused on this page)
  'pm-cyan':  '#14add7',
  'pm-deep':  '#223674',
  'pm-water': '#1d3056',
}
```

Component-internal colors (not tokens, live where they're used):

| Value | Where | Why |
| --- | --- | --- |
| `#1970a7` | `assets/state-lounge-script-blue.svg` | sampled from the blue STATE ENERGY logo so the script matches it |
| `rgba(6,12,26,…)` gradients | hero overlays | legibility scrim over the hero photo |
| `rgba(10,18,32,…)` gradient | State section overlay | ditto |
| `rgba(13,24,46,.2)` | CTA hover shadow | navy-tinted, never gray |
| map palette (`land #f2f7fc`, `water #c9def4`, `roadMajor #31537c`, `roadMinor #9db6d4`, `label #1d3056`) | map init JS | recolors OpenFreeMap layers to brand |
| `#5b6b83` | static-map attribution line | map furniture |

Spot colors from the print world (package report): PANTONE 1797 C (Atlantic red), PANTONE 287 C (blue).

## 2. Typography

**Real brand fonts (Adobe Fonts — embed pending):**

- **Transducer** — everything: headings, body, nav, CTAs. The design uses Regular/Medium/Bold/Black,
  plus **Extended** for the hero headline and **Condensed** variants. Get the embed at
  fonts.adobe.com → Web Project → `<link rel="stylesheet" href="https://use.typekit.net/XXXX.css">`.
- **Shelby Bold** — the script "Café"/"Lounge". NOT needed as a webfont: the words are extracted
  as vector SVGs (`assets/state-*-script*.svg`).

**Current stand-in** (swap when the embed arrives):
`"Lucida Grande", "Lucida Sans Unicode", "Lucida Sans", "Segoe UI", Tahoma, sans-serif`
Also update the `font-family` attrs on the floor-plan SVG `<text>` badges.

**Type scale in the prototype** (mobile → desktop):

| Role | Size | Weight/style |
| --- | --- | --- |
| Hero line 1 | 44 → 84px | bold italic |
| Hero line 2 | 44 → 84px | light, upright |
| Intro statement | 20 → 26px | medium, uppercase, `pm-ink` |
| Section heading (h2) | 28 → 34px | semibold |
| Feature headings (h3) | 26 → 30px | bold |
| State headings (h3) | 24 → 28px | semibold, white |
| Body | 16 → 17px | regular, `pm-slate` / `pm-ice` |
| CTA label | 16 → 19px | semibold, tracking .07em; "HER" bold |
| Footer headings | 19px | bold |
| Footer body | 15px | `pm-frost`, leading 1.75 |

## 3. Signature elements

**CTA button** — the brand shape: only the top-right corner rounded (`rounded-tr-[26px]`, nav 18px).
Label `BÓKA VØLLIN` + accent word `HER` one weight step bolder and in the contrast color:

- On light: `bg-pm-blue text-pm-ink`, HER = white
- On dark photo: `bg-white text-pm-ink`, HER = `pm-steel`
- Hover: 2px lift, 300ms `cubic-bezier(.2,.75,.2,1)`; navy shadow **only on light backgrounds**
  (`pm-cta-on-dark` kills the shadow — dark blur on dark photos reads as a smudge)
- Never animate the corner radius. `prefers-reduced-motion` disables all of it.

**Halftone dots** — decorative fields, always masked so they fade:
`radial-gradient(circle, rgba(158,194,232,.7) 2px, transparent 2.5px)` on a 22px grid
(26px grid + bigger dot for the hero). See `.dots-blue`, `.fade-*` utilities.

**Photo treatment** — all photography is duotone navy. The State image ships hotter than the hero,
so it gets `filter: saturate(0.55) brightness(0.78)` + a light navy gradient. Any new photography
should be graded to match the hero (`Mynd State Tennis.jpg`) or given the same filter treatment.

**Page rhythm** — full-width bands alternating: dark hero → white intro → `pm-panel` band → white →
dark photo → light map → navy footer. Content container: `max-w-[1280px]`, `px-5 md:px-10`.
Non-interactive content gets no hover effects.

## 4. Assets (`prototype/assets/`)

| File | Source | Note |
| --- | --- | --- |
| `hero-state-tennis.webp` 2880w | `Links/Mynd State Tennis.jpg` (4000px) | hero bg |
| `state-bg.webp` 2560w | `Links/Mynd.jpg` (5000×7000, top crop) | State bg, gets CSS filter |
| `state-energy-white.webp` / `-blue.webp` | `Links/STATE_LOGO_*` (13–16k px) | lockups |
| `state-cafe-script.svg` / `state-lounge-script.svg` (white), `state-lounge-script-blue.svg` | extracted from the PDF (Shelby Bold outlines) | never rasterize |
| `sponsor-atlantic.webp`, `sponsor-foroya-banki.webp` | rendered from the PDF @4× | designer's exact crops |
| `sponsor-smyril.webp`, `sponsor-busetur.webp`, `sponsor-xl-borg.webp` | `Links/` PNGs/JPG | **XL Borg needs a bigger source** (225px only) |
| `map-static.webp` | screenshot of the styled MapLibre view | no-WebGL fallback — re-capture after any map change |

Main logo: inline SVG `<symbol id="pm-logo">` in the page, reused via `<use>`.
Pipeline for new images: `cwebp -q 75-82`, kebab-case names, size ≈ 2× the largest CSS display width.

## 5. Map

MapLibre GL + OpenFreeMap `positron` style (no API key), recolored to brand in `style.load`.
Location: `62.0467495, -6.7858677` (Á Hjalla 2). Cooperative gestures with a 500ms intent delay
before the hint shows; hint text localized to Faroese. Address popup anchored to the pin, links to
`https://maps.app.goo.gl/hRV5iu2jbPa3KAJx6`. Keep OSM attribution (collapsed ⓘ is fine).

## 6. Porting checklist

- [ ] Replace Play CDN with real Tailwind build; move `tailwind.config` block into `tailwind.config.js`
- [ ] Add Adobe Fonts embed; swap `fontFamily.sans` to Transducer; update floor-plan SVG text fonts
- [ ] Replace invented Vøllir copy with verified court details (marked with an HTML comment)
- [ ] Real Facebook/Instagram URLs in the footer (currently `#`)
- [ ] Booking URL: all CTAs point at the `#temp-booking` placeholder — swap in the final Matchi deep link
- [ ] XL Borg logo: swap in higher-res/vector source when available
