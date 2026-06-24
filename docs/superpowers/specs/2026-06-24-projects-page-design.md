# Projects Page — Design Spec

**Date:** 2026-06-24
**Figma:** `rCR64bnT6VcLAXSYZML7ud`, section "Projects page" (`84:1426`), desktop frame `72:478`, mobile frame `268:1462`
**Source node from request:** `264:1830` (turned out to be only the Hero background image; the full page is `72:478`)

## Goal

Build the **Projects page** — the "next page" reached by filling the footer's
"Keep scrolling → Projects" pill on the main page. It presents Alex Foster's
ventures: an intro, a horizontally-scrolled venture carousel, a "Shared vision"
statement, a "Future Ventures" CTA, and the shared footer.

## Architecture

- **New standalone file `projects.html`** at repo root, alongside `index.html`.
- Update `index.html` line ~2346 `NEXT_PAGE_URL = "index.html"` → `"projects.html"`
  so the footer pill navigates here once filled.
- **Reuse, copied from `index.html`:** the Satoshi `@font-face` block, the
  morphing header/menu (`.panel` + `.menu-backdrop`), and the full footer
  (`.footer` incl. `.ft-cta` and the `.ft-progress` next-page pill). These are
  copied (this is a static multi-page site with no shared partials/build step);
  keep markup identical so the two pages stay visually in sync.
- On `projects.html` the footer's own next-page pill loops back to `index.html`
  (its `NEXT_PAGE_URL`), so the two pages form a loop. (Confirm target with user
  if a third page is planned; default = back to home.)
- Desktop (≥ ~900px) and mobile (≤ ~768px) share one document, switched by media
  queries — same pattern the existing site uses.

## Desktop layout (1440 reference) — stacked full-viewport sections

1. **Hero** (`72:479`) — dark mountains/silhouette photo background, dark veil.
   Shared header bar floating on top. Centered headline
   *"Built with intention. Designed to create lasting impact."* and a pill
   button **"Explore ventures"** (scrolls to the Ventures section). `data-header="light"`.

2. **Ventures Overview intro** (`78:886`) — white background. Eyebrow
   "▪ VENTURES OVERVIEW", headline *"Beyond individual projects"*, a 3D chrome
   object (`image 2039883997`) centered, and **"SCROLL TO EXPLORE ⌄"** bottom-left.

3. **Venture carousel** (`78:906`, `78:917`, `78:923`) — background transitions
   white → black; ventures move through horizontally, pinned while the user
   scrolls vertically. Three ventures, each with name, description, a glass/chrome
   3D object, and an **"Explore Venture"** pill:
   - **Vertus** — chrome U-shape (`2039883997`). "Designed to create meaningful
     solutions through thoughtful execution and distinctive design. Every detail
     reflects a commitment to quality."
   - **Alpha Ledger** — chrome "AW" (`2039883996`). "A venture shaped by
     innovation and precision, bringing together technology and strategy to build
     for the future."
   - **Vanquish** — chrome shape (`2039883995`). "A modern platform focused on
     delivering seamless digital experiences. Built with clarity, performance and
     a long-term vision."

4. **Shared vision** (`72:705`) — black background. Eyebrow "▪ SHARED VISION".
   Giant outline/gradient text **"CREATE IMPACT"** … **"SHARED VISION"** that
   tracks horizontally across the viewport on scroll, with a framed poster/plaque
   image (`Rectangle 11`) centered. (Mobile copy adds the line: "Whether you're
   launching a company, exploring a new direction, or seeking strategic
   collaboration, let's start the conversation.")

5. **CTA / Future Ventures** (`78:1221`) — black background. A lightbox poster
   ("Core Signal" glass-V) centered, giant **"FUTURE VENTURES"** wordmark with
   corner-bracket framing, scaling/parallax on scroll.

6. **Footer** (`72:610`) — the shared footer component (CTA lines + next-page pill
   + © + big F mark).

## Mobile layout (393 reference, `268:1462`)

Same sections stacked, single column: Hero → Ventures Overview (image below text)
→ per-venture cards (Vertus / Alpha Ledger / Vanquish stacked vertically, image
under text, "Explore Venture" link) → Shared vision (poster + "CREATE IMPACT
SHARED VISION" + paragraph) → Projects/Future Ventures CTA → mobile footer
(`Footer / mobile`). No horizontal pin on mobile; the carousel becomes stacked
cards.

## Scroll animations (full fidelity, matching existing site)

- **Hero → Ventures:** standard scroll; "Explore ventures" smooth-scrolls down.
- **Ventures intro → carousel:** background cross-fades white → black; the chrome
  object hands off into the first venture. Carousel is a **pinned horizontal
  scroll** (vertical wheel/scroll advances ventures left), like the existing
  pinned sections in `index.html`.
- **Shared vision:** the oversized "CREATE IMPACT / SHARED VISION" text translates
  horizontally as the section scrolls through; poster scales slightly (parallax).
- **CTA:** "FUTURE VENTURES" and the lightbox scale/parallax on entry.
- Respect `prefers-reduced-motion`: fall back to static positions / simple fades.
- Reuse the site's existing scroll/scrub helpers and easing where possible for a
  consistent feel.

## Assets to export from Figma (PNG → WebP via the project's Pillow pipeline)

Saved under `images/projects/`:
- `hero.webp` — Hero background `264:1830`
- `chrome-vertus.webp` — `2039883997`
- `chrome-alpha.webp` — `2039883996`
- `chrome-vanquish.webp` — `2039883995`
- `shared-poster.webp` — Shared-vision plaque (`Rectangle 11` content)
- `cta-lightbox.webp` — CTA "Core Signal" lightbox
- Mobile crops if they differ materially from desktop (else reuse desktop assets).

Export at 2× where the asset is a photo/3D render; convert with quality 82.

## Reuse / consistency notes

- Fonts: Satoshi family already in `/fonts`; copy the `@font-face` + variable
  setup from `index.html`.
- Header/menu and Footer markup + their JS (menu toggle, click-outside, footer
  progress pill) copied verbatim and wired the same way.
- `data-header="light|dark"` per section drives header color, same mechanism as
  the main page.

## Out of scope (this pass)

- Real destinations for "Explore Venture" / "Explore ventures" buttons (link to
  `#` or smooth-scroll anchors for now).
- Any backend, CMS, or new venture detail pages.
- Refactoring `index.html` beyond the one `NEXT_PAGE_URL` change.

## Open questions

- Footer next-page target **on** projects.html: default = back to `index.html`.
  Change if a third page is intended.
