# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Static one-page marketing site for **AiDN Agency** (aidnagency.fr). Zero dependencies, zero build step. HTML + CSS + vanilla JS only, deployed on Vercel.

Language of the site content and commit messages: **French**. Keep new copy and commit messages in French unless told otherwise.

## Commands

This repo has no package manager, no build, no test suite.

```bash
# Local preview (http://localhost:8000) — required to test anything UI
python3 -m http.server 8000

# Deploy to production (requires Vercel CLI + auth)
vercel --prod
```

Git push on the connected branch auto-deploys via Vercel.

## Architecture

### Single-file design

Almost everything lives in `index.html` (~3k lines): HTML, inline `<style>`, and inline `<script>`. There is no CSS or JS split into separate files — **do not introduce bundlers, preprocessors, or external stylesheets**. When adding a feature, extend the existing `<style>` and `<script>` blocks in place. The three other HTML pages (`404.html`, `mentions-legales.html`, `politique-confidentialite.html`) follow the same self-contained pattern.

### Page section order (inside `<main id="main">`)

`hero` → `zoom` (sticky scroll-zoom) → `manifesto` → `approche` (#approche) → `services` (#services, accordion) → `feed-section` (#realisations, 12-tile grid) → `pricing` (#offres) → `faq` → `contact` → `foot`. Anchor nav links in the top `<nav class="nav" id="header">` and `#mobileMenu` match these IDs.

### Design tokens

All theming is driven by CSS custom properties on `:root` around index.html:62:

- `--ink: #0E0E0E` / `--paper: #F2EFE8` / `--orange: #FF5B0C` (brand accent) / `--muted`, `--muted-dark`, `--line`, `--line-dark`
- Fonts: `--sans` (Inter), `--serif` (Instrument Serif — used via `.serif` class for italic/display words), `--mono` (also Inter)
- `--ease: cubic-bezier(0.22, 1, 0.36, 1)`

Reuse these tokens instead of hardcoding colors. The `.hl` class highlights a word in orange; the `.serif` class swaps to Instrument Serif.

### Scroll-driven animation pattern

The `.zoom` section uses modern **`animation-timeline: view()`** with a `@keyframes` block and a JS fallback guarded by `if (!CSS.supports('animation-timeline: view()'))` (around index.html:2868). When adding new scroll-linked effects:

1. Author the CSS `@keyframes` + `animation-timeline: view()` + `animation-range: entry X% cover Y%`.
2. Mirror the effect in the fallback `onScroll` handler so Safari/older Chromium still works. The fallback computes `progress` from `section.getBoundingClientRect()` and applies `transform`/`opacity` inline.
3. Read per-element tuning values from CSS custom properties (e.g. `--tx`, `--ty`) via `getComputedStyle(el).getPropertyValue(...)` so CSS stays the single source of truth.

Other animation primitives already wired up:

- `.reveal` / `.reveal-line` / `.reveal-word` — IntersectionObserver adds `.revealed` once (threshold 0.15, `rootMargin: '0px 0px -10% 0px'`). Per-element stagger via inline `style="--d:0.15s"`.
- `data-hover` — toggles `body.cursor-hover` for the custom cursor.
- `data-magnetic` — magnetic pull on mousemove (15% X, 20% Y); only applied when `(pointer: coarse)` is false.
- Custom cursor (`.cursor` + `.cursor-dot`) runs an rAF loop, disabled on touch and on `prefers-reduced-motion`.
- Dark-region detection: when a section in `.services, .cases, .testi, .pricing, .foot` is under the viewport midline, `body` gets a class toggled so the cursor inverts.

Every animated element should have a `prefers-reduced-motion: reduce` fallback (see how `.zoom-frame img` resets under that media query).

### Interactive components

Accordions (`.service-row`, `.faq-item`) toggle via `toggleService(id)` / `toggleFaq(id)` — clicking one collapses siblings. Mobile menu via `toggleMobileMenu()` locks `body.style.overflow` while open.

Project modal: `openProject(id)` pulls data from the `projectsData` object (index.html:2972 onward — 4 projects keyed by numeric id) and injects into `#projectModal`. `closeProject()` clears it. Escape key closes. Clicking outside content closes. **Several gallery images are `placehold.co` placeholders** (noted in README.md as TODO before public launch).

### Assets & CDN

- Product/editorial imagery: Cloudinary under `res.cloudinary.com/dke6iy9pe/image/upload/...`. Always include `f_auto,q_auto` for automatic WebP/AVIF + quality. Use `c_fill,w_<n>,h_<n>,g_auto` for responsive crops. Preconnect + preload the LCP hero image is already set up in `<head>`.
- Local assets (`assets/icons/`, `assets/images/logo.png`, `favicon.ico`) are the only things served from the origin. Fonts load from Google Fonts (preconnected).
- Analytics: Vercel Web Analytics + Speed Insights scripts at the bottom of `index.html`. Both load from `/_vercel/...` and only work on the deployed Vercel URL.

### Vercel config (`vercel.json`)

- `cleanUrls: true` — links to legal pages use `/mentions-legales` (no `.html`).
- Cache policy: images/fonts 1 year immutable; css/js 30 days; HTML must-revalidate. Already handled via `Cache-Control` headers — do not add `<meta http-equiv="cache-control">`.
- Security headers (HSTS, X-Frame-Options SAMEORIGIN, Referrer-Policy, Permissions-Policy) are set globally.
- No build, no install: `framework: null`, `buildCommand: null`, `installCommand: null`, `outputDirectory: "."`. If you're tempted to add a toolchain, don't — the site ships the repo root as-is.

### SEO / legal surface

`sitemap.xml` lists the 3 canonical URLs (home + 2 legal). `robots.txt` points to it. JSON-LD Organization block is at the top of `index.html`. The two legal pages have `[À COMPLÉTER]` placeholders (company registration details, DPO contact) that must be filled before public launch — see the checklist in README.md.

## Conventions

- **Test changes in the browser.** There's no unit/E2E harness. Before reporting UI work done, run `python3 -m http.server 8000` and load the page; check the golden path on desktop and mobile viewport (DevTools responsive ≤ 768px is the main breakpoint).
- **Never add dependencies or a build step** (no npm install, no bundler, no framework). The repo's value proposition is that it ships as plain files.
- **Vanilla JS only**, no frameworks, no TypeScript.
- **French content.** UI text, `alt` attributes, ARIA labels, and commit messages are in French. Commit prefix style from `git log`: `Feat:`, `Fix:`, `Tune:` followed by a French summary.
- **Accessibility is part of every change.** Preserve skip link, landmarks (`<main>`, `<nav>`, `<footer>`), `aria-label` / `aria-expanded` / `aria-controls`, and visible `:focus` outlines. Hide decorative-only parallax/animation layers behind `prefers-reduced-motion: reduce`.
- **Respect stacking order in `.zoom-sticky`.** `.zoom-label` and `.zoom-caption` are `z-index: 2`, `.zoom-frame` sits at `z-index: 1`, background parallax layers go at `z-index: 0`. Anything absolutely positioned inside the sticky must opt in to this scheme.
- **Placeholders to watch for before public launch** (tracked in README.md §⚠️): WhatsApp number `+33 6 12 34 56 78` (8 occurrences), `[À COMPLÉTER]` fields in the two legal pages, `placehold.co` images in `projectsData`.
