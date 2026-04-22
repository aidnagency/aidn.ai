# CLAUDE.md

Guidance for AI assistants working in this repo. Read this before making changes.

## What this repo is

Static marketing site for **AiDN Agency** — a French AI-native art direction studio. Deployed on Vercel at <https://www.aidnagency.fr>.

- **Pure static HTML/CSS/JS.** No build step, no bundler, no package manager, no framework.
- **Three pages**, each one a self-contained HTML file with inline `<style>` and `<script>`:
  - `index.html` — the whole marketing homepage (~3000 lines)
  - `mentions-legales.html` — French legal notice (LCEN)
  - `politique-confidentialite.html` — privacy policy (RGPD)
  - `404.html` — custom 404
- **No tests, no CI for tests, no lint config.** If you want to test a change, you open it in a browser. There is no `package.json`.

## File layout

```
.
├── index.html                      Homepage — CSS, JS, data all inline
├── 404.html
├── mentions-legales.html
├── politique-confidentialite.html
├── vercel.json                     Vercel config: headers, cache, cleanUrls
├── robots.txt
├── sitemap.xml                     3 URLs (must be kept in sync with pages)
├── site.webmanifest                PWA manifest
├── favicon.ico
├── assets/
│   ├── icons/                      Favicons + apple-touch + PWA icons
│   └── images/logo.png             Local logo fallback (remote is Cloudinary)
└── README.md
```

There is no `src/`, no `public/`, no `dist/`. Files are served exactly as they are from repo root — `vercel.json` has `"outputDirectory": "."`.

## Running locally

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

That's it. No install step. Any static server works.

## Deployment

- Auto-deploys from Git (Vercel is wired to the connected branch).
- Manual: `vercel --prod` from repo root.
- Headers, caching, `cleanUrls` (`/mentions-legales` resolves to `mentions-legales.html`) all configured in `vercel.json`.

## Architecture of `index.html`

One file, roughly split into:

1. `<head>` (lines ~1-60) — meta, Open Graph, Twitter Card, JSON-LD `Organization`, preconnects to Google Fonts + Cloudinary, preload of the hero image.
2. `<style>` (lines ~61-2264) — all CSS. Big but organized top-to-bottom following the page sections.
3. `<body>` (lines ~2265-2815) — markup in semantic order:
   - `nav.nav#header` + `.mobile-menu#mobileMenu`
   - `<main id="main">` containing sections in this order: `.hero`, `.zoom` (sticky parallax), `.marquee-wrap`, `.manifesto`, `#approche`, `#services`, `#realisations` (feed + project modal trigger), `#offres` (pricing), `.faq`, `.contact`
   - `<footer class="foot">`
   - `.project-modal#projectModal` — single modal reused for all case studies
4. `<script>` (lines ~2817-3118) — all JS.
5. Vercel Analytics + Speed Insights scripts (last).

### CSS design tokens (`:root`)

```
--ink: #0E0E0E      text / dark bg
--paper: #F2EFE8    background / light text
--orange: #FF5B0C   accent (used sparingly — hero «futur», badges, selection)
--muted: #8A8680
--sans: Inter       via Google Fonts
--serif: Instrument Serif    italic display type
--ease: cubic-bezier(0.22, 1, 0.36, 1)
```

Keep the palette minimal. Orange is an editorial accent — don't spread it.

### JS features (all inline, vanilla)

- Custom cursor (`.cursor`, `.cursor-dot`) with magnetic hover via `data-magnetic`, disabled on `pointer: coarse`.
- `.dark-region` body class toggled when dark sections are mid-viewport (inverts cursor/colors).
- `IntersectionObserver` reveals `.reveal`, `.reveal-line`, `.reveal-word` on enter.
- **Zoom section** (`.zoom` / `.zoom-sticky` / `.zoom-frame` / `.zoom-floats`): sticky-scroll parallax. Uses native `animation-timeline: view()` when supported, with a JS fallback that calculates progress from `getBoundingClientRect`. Floating decorative images (`.float-img`) drive in from the center, drift outward with per-image parallax factors (`--p` mid, `--pe` end), and fade out as the central frame zooms in.
- Marquee scroller (`.marquee-track`).
- Accordions: `toggleService(id)`, `toggleFaq(id)` — single-open behavior.
- Mobile menu: `toggleMobileMenu()` toggles `.mobile-menu.open` and locks body scroll.
- **Project modal**: driven by `projectsData` (object at bottom of script, keyed `1-4`) + `openProject(id)`. Each entry carries `title`, `category`, `tags`, `projet`, `univers`, `intention`, `defi`, `da`, `livrables`, `note`, `gallery`. If you add a project, update both `projectsData` and the `.feed-item` trigger.
- Smooth scroll for `a[href^="#"]` (overrides default to keep the header offset from `scroll-margin-top: 80px`).
- `prefers-reduced-motion` disables all animations globally via the CSS rule at the top.

## Conventions — follow these

- **Never split inline CSS/JS into separate files** unless the user explicitly asks. The "one HTML file per page, everything inline" pattern is intentional — zero extra requests, zero build, no cache invalidation dance.
- **No dependencies.** Don't add npm, don't add a build step, don't bring in a framework. If you're tempted to reach for a library, you're probably solving the wrong problem.
- **Keep it French.** All user-facing copy is French. Don't translate to English. Match the existing typographic style: curly apostrophes (`l'`), em dashes (`—`), French accentuation.
- **CSS custom properties for positioning / animation knobs.** The `.float-img` pattern (`--x`, `--y`, `--r`, `--p`, `--pe`) is the house style: keyframes reference variables so each instance tweaks itself via `style="--x: …"`. Reuse this pattern rather than writing new classes per instance.
- **Respect `prefers-reduced-motion`.** Any new animation should degrade gracefully — the global rule at the top handles the common case, but check that scroll-linked JS doesn't force motion.
- **Respect `pointer: coarse`.** The custom cursor and magnetic hover are desktop-only. Don't add anything hover-dependent without a touch fallback.
- **Keep all three pages in visual sync.** `mentions-legales.html` and `politique-confidentialite.html` use a trimmed version of the same tokens and type system. If you change the palette or type scale in `index.html`, update those pages too.
- **Update `sitemap.xml` `<lastmod>`** if you materially change a page's content.
- **No cookies, no trackers beyond Vercel Analytics / Speed Insights.** The privacy policy claims "no cookies" — honor it.
- **`cleanUrls: true`.** Internal links to legal pages use `/mentions-legales` (no `.html`), not `/mentions-legales.html`.

## Scroll-linked animation gotchas

- `animation-timeline: view()` is Chromium-only as of writing. The JS fallback path (`if (!CSS.supports('animation-timeline: view()'))`) must be kept in parity with any keyframe change. When you edit `@keyframes zoomIn` / `zoomImgPar` / `floatPar`, update the fallback `onScroll` in the same commit.
- `.zoom` section height (`360vh` desktop, `320vh` mobile) controls the sticky scroll runway. Shortening it compresses the zoom; lengthening it gives more parallax room. Don't change it without checking both the CSS ranges (`animation-range: entry X% cover Y%`) and the JS progress bounds (`(progress - 0.2) / 0.4`, etc.) — they must move together.
- `.float-img` transforms stack a `translate(-50%, -50%)` (centering) with `translate(var(--x), var(--y))` (offset) with `rotate()` and `scale()`. Keep that order.

## Known placeholders (real TODOs)

The homepage and legal pages ship with placeholder business data. Do **not** invent values for these — they require real information from the business owner:

- `wa.me/33612345678` — placeholder WhatsApp number, 9 occurrences in `index.html`. Real number needed before public launch.
- `placehold.co/...` — 12 `.feed-item` images + 12 gallery images across 4 entries in `projectsData`. Replace with real Cloudinary-hosted visuals.
- `[À COMPLÉTER]` fields in `mentions-legales.html` — raison sociale, SIRET, RCS, capital, adresse, téléphone, responsable légal.
- `[À COMPLÉTER]` fields in `politique-confidentialite.html` — responsable du traitement.

If asked to "finish the TODOs" without data being provided, stop and ask. Don't fabricate legal or contact information.

## Common tasks

### Edit a section's copy
Find the section by its `id` (`#services`, `#offres`, etc.) or by its `.sec-label` ("01 — Manifeste", "02 — …"). Edit the text in place.

### Add a project to the feed
1. Append an entry to `projectsData` with a new numeric key.
2. Add a `.feed-item` in `#realisations` with matching `onclick="openProject(N)"` and a thumbnail.
3. Make sure image URLs are real (Cloudinary with `f_auto,q_auto` for WebP/AVIF).

### Add / change an animation
1. Write the CSS `@keyframes` + `animation-timeline: view()` + `animation-range`.
2. Mirror it in the `!CSS.supports('animation-timeline: view()')` fallback block.
3. Verify with both a modern Chromium (native path) and either Firefox or Safari (fallback path).

### Change colors or fonts
Edit `:root` tokens in `index.html`, then replicate the same changes in `mentions-legales.html` and `politique-confidentialite.html`.

## What NOT to do

- Don't add `package.json`, `tsconfig`, `vite.config`, or any build/lint/format tooling without being asked.
- Don't extract CSS or JS into separate files.
- Don't swap the static-site deploy for SSR or a framework.
- Don't add unit tests — there's no logic to unit test here. If testing is requested, the right layer is Playwright smoke + axe + lychee (link check) in a GitHub Action.
- Don't touch placeholders (WhatsApp number, SIRET, etc.) with invented values.
- Don't commit screenshots, local backups (`*.backup.html`), or `.vercel/` — all covered by `.gitignore`.

## Git workflow

- Commit messages in this repo follow a loose `Type: short description` pattern in French (`Feat:`, `Fix:`, `Tune:`, `Refonte …`). Match the existing style.
- One focused change per commit.
- No PR template; keep PR descriptions short.
