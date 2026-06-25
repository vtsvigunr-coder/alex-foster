# Page transitions, preloader gating, and Projects hero reveal

Date: 2026-06-25
Scope: `index.html`, `projects.html`

## Goal

Make navigation between the two pages feel like a single app: a new page slides
up from the bottom over the old one, the full preloader plays only on the first
open of the site, and the Projects hero animates in. No scroll is involved in any
page-to-page transition (the footer "fill" is the one intentional exception).

## Decisions (confirmed with user)

1. The transition is the **new page itself** (hero included) sliding up from the
   bottom over the old one — no separate colored curtain panel.
2. The Projects hero content appears **all at once** (one soft fade + rise), not
   staggered.
3. The full preloader plays **once per browser session** (`sessionStorage`).

## Design

### 1. Page transition — native cross-document View Transitions API

Both pages opt in. No bespoke JS transition engine; the browser animates the
document swap.

```css
@view-transition { navigation: auto; }
::view-transition-old(root) { animation: none; }                 /* old stays in place */
::view-transition-new(root) { animation: vt-slide-up .5s var(--ease) both; }
@keyframes vt-slide-up { from { transform: translateY(100%); } to { transform: none; } }
```

Because the new view paints above the old and is opaque, it reads as the new
page sweeping up over the still old page.

- Triggers automatically for every same-origin document navigation: menu links,
  `window.location.href` from the footer pill, etc. No scroll.
- In-page hash links (`#pj-ventures`) are same-document and are NOT affected.
- Fallback: browsers without support, and `prefers-reduced-motion: reduce`, get
  an instant navigation (and the custom keyframe is neutralized under reduced
  motion). Everything still works.

### 2. Preloader gating — once per session

- Flag: `sessionStorage.getItem('introPlayed')`.
- `index.html`:
  - An inline script in `<head>` runs before first paint. If the flag is set, it
    marks the document so CSS hides `.preloader` and reveals the site
    synchronously — this keeps the incoming view-transition snapshot showing the
    hero, never a flash of preloader.
  - The preloader timeline (`run()`) early-returns when the flag is set.
  - When the preloader finishes playing for real, it sets the flag.
- The flag is also set on load of **any** page (including `projects.html`), so
  "first open of the site" is honored even if the user lands on Projects first
  and then goes Home — Home will not replay the preloader.
- `projects.html` has no preloader markup (unchanged) — entrance is the slide +
  hero reveal only.

### 3. Projects hero reveal — all at once

- During an internal navigation the hero rides up with the page (primary
  reveal).
- Additionally, a one-shot CSS load animation on `.pj-hero-bg` and
  `.pj-hero-inner` (`opacity 0→1`, `translateY ~24px→0`, single group, ~0.7s)
  so a **direct** load of `projects.html` (no transition) also reveals the hero.
  On an internal navigation both compose and still read as "all at once".
- Honors `prefers-reduced-motion`.

### 4. Menu links

On both pages (markup is identical):
- `Home` → `index.html`
- `Projects` → `projects.html`
- Other items (Blog, AWF chapters, Timeline, Media, Contact) stay `href="#"` —
  out of scope for this change.

### 5. Footer "fill → next page"

- The existing overscroll-fills-the-pill logic is unchanged: when the pill
  reaches 100%, `go()` immediately sets `window.location.href`.
- That navigation now animates automatically via the View Transitions slide-up,
  so the switch is smooth instead of a hard, scroll-coupled cut. This satisfies
  both "as soon as it fills, transition" and "smooth, not an abrupt scroll
  drag." No footer JS change required.

## Files changed

- `index.html` — view-transition CSS; preloader session gating (head script +
  CSS + `run()` guard + set flag); menu Home/Projects links.
- `projects.html` — view-transition CSS; hero on-load reveal (CSS + classes);
  set session flag on load; menu Home/Projects links.

## Out of scope

- Wiring the remaining menu items (Blog, Timeline, Media, Contact, AWF chapters).
- Any change to the index hero entrance.
- Changing the footer fill threshold / mechanics.
