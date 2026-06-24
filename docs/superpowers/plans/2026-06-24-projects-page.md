# Projects Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `projects.html` — the "next page" reached from the main-page footer pill — reproducing the Figma "Projects page" (desktop `72:478` + mobile `268:1462`) with full scroll animations.

**Architecture:** A single standalone static HTML file at repo root, mirroring `index.html`'s self-contained pattern (inline `<style>` + inline `<script>`, Satoshi `@font-face`, no build step). Header/menu and footer markup + JS are copied from `index.html` so both pages stay in sync. Sections are full-viewport; scroll animations use vanilla JS scroll listeners + `requestAnimationFrame`, matching the existing site's approach.

**Tech Stack:** Plain HTML5, CSS (no framework — convert the Figma Tailwind to hand-written CSS), vanilla JS. Image assets exported from Figma → `images/projects/*.webp` via the project's Pillow PNG→WebP pipeline (quality 82).

## Global Constraints

- Font: **Satoshi** (already in `/fonts`); copy the `@font-face` block from `index.html` verbatim. Weights used: Medium (500), Bold (700).
- Type tokens (copy exactly): H2 = 80px / lh 84 / tracking -5% (≈ -4px); H3 = 64px / lh 68 / tracking -4% (≈ -2.56px); Eyebrow L = 20px Bold / lh 24; Eyebrow S = 14px Medium / lh 18 uppercase; Body/Button = 16px Medium / lh 20.
- Colors: white `#fff`, black `#000`, Neutral400 `#555`, Neutral200 `#B2B2B2`, grey `#808080`.
- Desktop reference width 1440; mobile reference 393. Desktop ≥ 900px, mobile ≤ 768px (match index.html's breakpoint).
- Respect `prefers-reduced-motion: reduce` → disable scroll-scrub animations, show final/static state with simple opacity fades.
- Header color per section via `data-header="light|dark"`, same mechanism as index.html.
- Commit after each task. Single-page static site → no unit tests; **verification = open the file in a browser and visually compare against the Figma reference screenshots** saved in `/tmp` (p_hero, p_ventures, p_shared, p_cta, v_vertus, v_alpha, v_vanquish) and the live Figma.

---

## Asset map (Figma → file)

| File (`images/projects/`) | Figma node | Asset URL const |
|---|---|---|
| `hero.webp` | 264:1830 | imgImage2039883998 `4b76ceac-5d81-47ab-a298-00d08e681d31` |
| `chrome-vertus.webp` | 239:2490 (2039883997) | imgImage2039883997 `6fe4859a-236a-42cf-949b-fe73388af682` |
| `shared-poster.webp` | 84:1370 (Rectangle 11) | imgRectangle11 `4684720a-4b9e-4ae0-827a-9939e1e6c15b` |
| `cta-lightbox.webp` | 78:1222 (Rectangle 9) | imgRectangle9 `a07e4d7f-14be-46bb-a0b2-d5fb46e2dc06` |
| `chrome-alpha.webp` | 2039883996 (in 78:917) | fetch via get_design_context on `78:917` at build time |
| `chrome-vanquish.webp` | 2039883995 (in 78:923) | fetch via get_design_context on `78:923` at build time |

Footer F-mark / social icons / etc. are already reused from `index.html` markup, so no new export needed for the footer.

---

### Task 1: Scaffold `projects.html` + wire the main-page link

**Files:**
- Create: `projects.html`
- Modify: `index.html:2346` (`NEXT_PAGE_URL`)

**Interfaces:**
- Produces: `projects.html` containing, in order, `<head>` (copied `@font-face`, reset, shared header/footer CSS), `<body>` with `.panel` header markup (copied), an empty `<main class="projects">` placeholder, the copied `.footer` markup, and the copied header/footer `<script>` (menu toggle, click-outside, `ftProgress` pill). Footer's `NEXT_PAGE_URL` on this page = `"index.html"`.

- [ ] **Step 1:** Copy from `index.html` into a new `projects.html`: the full `<head>` `@font-face` + base reset + the `.panel`/`.menu-backdrop`/`.footer`/`.ft-*` CSS rules; the `.panel` + `.menu-backdrop` body markup; the `.footer` markup; and the JS for menu toggle + footer progress pill. Set this page's `NEXT_PAGE_URL = "index.html"`.
- [ ] **Step 2:** Insert `<main class="projects"></main>` between header and footer as the section container.
- [ ] **Step 3:** In `index.html` change `const NEXT_PAGE_URL = "index.html";` → `"projects.html";` (remove the TODO comment).
- [ ] **Step 4 (verify):** `open projects.html` — header bar + menu open/close work, footer renders, footer pill fills on scroll/overscroll. From `index.html`, filling the footer pill navigates to `projects.html`.
- [ ] **Step 5:** Commit: `Scaffold projects.html (shared header/footer) + wire main-page next-page link`.

---

### Task 2: Export & convert image assets

**Files:**
- Create: `images/projects/hero.webp`, `chrome-vertus.webp`, `chrome-alpha.webp`, `chrome-vanquish.webp`, `shared-poster.webp`, `cta-lightbox.webp`

- [ ] **Step 1:** `curl` each asset URL from the asset map to a temp PNG (and fetch Alpha/Vanquish chrome URLs via `get_design_context` on `78:917`/`78:923`).
- [ ] **Step 2:** Convert each PNG → WebP with the Pillow pipeline (quality 82) into `images/projects/`.
- [ ] **Step 3 (verify):** `ls -la images/projects/` shows all six WebP files at reasonable sizes; spot-open two to confirm correct images.
- [ ] **Step 4:** Commit: `Add Projects page image assets (webp)`.

---

### Task 3: Hero section

**Files:** Modify `projects.html` (`<main>` + CSS)

**Interfaces:**
- Consumes: `images/projects/hero.webp`.
- Produces: `<section class="pj-hero" data-header="light">` — bg image + veil gradient (`linear-gradient(180deg, rgba(0,0,0,0) 0%, rgba(0,0,0,.25) 100%)` over a flat `rgba(0,0,0,.6)`), centered `<h1>` "Built with intention. Designed to create lasting impact." (H3 token, white, max-width 800), and an "Explore ventures" pill that smooth-scrolls to `#pj-ventures`.

- [ ] **Step 1:** Add markup + CSS for `.pj-hero` (100vh, image cover bottom, veil, centered title stack gap 28px, glass pill button).
- [ ] **Step 2:** Wire "Explore ventures" → `scrollInto({behavior:'smooth'})` on `#pj-ventures`.
- [ ] **Step 3 (verify):** Compare to `/tmp/p_hero.png` — headline wrap, button, veil match.
- [ ] **Step 4:** Commit: `Projects: Hero section`.

---

### Task 4: Ventures Overview intro + pinned venture carousel

**Files:** Modify `projects.html`

**Interfaces:**
- Consumes: `chrome-vertus.webp`, `chrome-alpha.webp`, `chrome-vanquish.webp`.
- Produces: `<section id="pj-ventures" class="pj-ventures">` containing the white intro (eyebrow "▪ VENTURES OVERVIEW", H3 "Beyond individual projects", centered chrome image, "SCROLL TO EXPLORE ⌄" bottom-left) and a pinned horizontal carousel of 3 venture cards with name (H3), description (body), glass "Explore Venture" pill, and chrome object. Data array `VENTURES = [{name, desc, img}]` drives the cards.

- [ ] **Step 1:** Build the white intro panel; eyebrow uses a 6px black square dot + 20px Bold uppercase.
- [ ] **Step 2:** Build the carousel track (3 cards) and the white→black background transition as it enters.
- [ ] **Step 3:** Implement pinned horizontal scroll: pin the section for ~3×viewport of vertical scroll, translate the track horizontally proportional to scroll progress (rAF-smoothed), like the existing pinned sections in index.html. Reduced-motion → stack cards vertically, no pin.
- [ ] **Step 4 (verify):** Compare to `/tmp/p_ventures.png`, `v_vertus.png`, `v_alpha.png`, `v_vanquish.png` — names, descriptions, pill placement, chrome objects, bg transition.
- [ ] **Step 5:** Commit: `Projects: Ventures overview + pinned venture carousel`.

---

### Task 5: Shared Vision section

**Files:** Modify `projects.html`

**Interfaces:**
- Consumes: `shared-poster.webp`.
- Produces: `<section class="pj-shared" data-header="dark">` — black bg, eyebrow "▪ SHARED VISION" (white dot) top-center, centered poster (172×186, rounded 8, dark overlay 40%), and two giant H3 words "CREATE IMPACT" (left, gradient fading to white toward centre) + "SHARED VISION" (right, gradient from white) that translate horizontally on scroll so they sweep across/past the poster.

- [ ] **Step 1:** Markup + static CSS (poster, eyebrow, the two gradient-clipped headline words at their Figma rest positions).
- [ ] **Step 2:** Scroll-scrub: map section progress → horizontal translate of the two words (converge/cross at centre), rAF-smoothed; poster slight parallax scale. Reduced-motion → static rest positions.
- [ ] **Step 3 (verify):** Compare to `/tmp/p_shared.png` — gradient text, poster, eyebrow.
- [ ] **Step 4:** Commit: `Projects: Shared vision section`.

---

### Task 6: CTA / Future Ventures section

**Files:** Modify `projects.html`

**Interfaces:**
- Consumes: `cta-lightbox.webp`.
- Produces: `<section class="pj-cta" data-header="dark">` — black bg, centered lightbox poster (400×530, rounded 10), giant H2 "FUTURE VENTURES" (80px) lower-centre flanked by left/right corner-bracket SVGs, with scale/parallax on entry.

- [ ] **Step 1:** Markup + CSS (lightbox image, H2 wordmark, two bracket SVGs inline — reuse the bracket path style from the footer `.ft-brk`).
- [ ] **Step 2:** Scroll entry animation: lightbox + wordmark scale-up/parallax. Reduced-motion → static.
- [ ] **Step 3 (verify):** Compare to `/tmp/p_cta.png` — lightbox, wordmark, brackets.
- [ ] **Step 4:** Commit: `Projects: CTA / Future Ventures section`.

---

### Task 7: Mobile layout + reduced-motion pass + final polish

**Files:** Modify `projects.html`

**Interfaces:**
- Produces: `@media (max-width:768px)` rules converting every section to the single-column mobile frame (`268:1462`): Hero (393×852 proportions), Ventures stacked (text then 393-wide chrome image; venture cards stacked vertically with "Explore Venture" link), Shared vision (poster + "CREATE IMPACT SHARED VISION" stacked + paragraph), Future Ventures CTA, mobile footer (the `Footer / mobile` variant already used by index.html). Plus the global `prefers-reduced-motion` block.

- [ ] **Step 1:** Add mobile media queries per section (reference: mobile Figma frame; carousel becomes stacked cards — no horizontal pin on mobile).
- [ ] **Step 2:** Add/confirm the `prefers-reduced-motion` fallbacks across all sections.
- [ ] **Step 3 (verify):** Browser at 393px width — each section matches the mobile Figma; at 1440px nothing regressed; reduced-motion shows static layout.
- [ ] **Step 4:** Commit: `Projects: mobile layout + reduced-motion + polish`.

---

## Self-Review

- **Spec coverage:** Hero→T3, Ventures intro+carousel→T4, Shared vision→T5, CTA→T6, Footer+header reuse→T1, mobile→T7, assets→T2, NEXT_PAGE_URL wiring→T1, animations→T4/T5/T6, reduced-motion→T7. All spec sections covered.
- **Open question resolved:** projects.html footer next-page → `index.html` (loop back), per spec default.
- **Placeholder scan:** none — every task has concrete nodes, tokens, and verification.
- **Naming consistency:** section ids/classes `pj-hero`, `#pj-ventures`, `pj-shared`, `pj-cta`; data array `VENTURES`; asset filenames consistent across tasks.
