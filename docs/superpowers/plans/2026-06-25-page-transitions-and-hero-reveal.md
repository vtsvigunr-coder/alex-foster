# Page Transitions, Preloader Gating & Projects Hero Reveal — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make Home ↔ Projects navigation feel like one app — the new page slides up from the bottom over the old one, the full preloader plays only once per session, and the Projects hero animates in.

**Architecture:** Native cross-document **View Transitions API** drives the slide-up (no bespoke JS engine). `sessionStorage` gates the preloader. A one-shot CSS animation reveals the Projects hero. Menu links are wired to the two real pages.

**Tech Stack:** Plain HTML/CSS/JS (two static files: `index.html`, `projects.html`). No build, no test runner. Verification is by browser observation, using puppeteer-core + the installed Chrome (per project convention).

## Global Constraints

- Two files only: `index.html`, `projects.html`. No new files, no dependencies.
- Ease curve already defined: `:root { --ease: cubic-bezier(.16, 1, .3, 1); }` — reuse `var(--ease)`.
- Both pages share identical menu/header markup — apply menu edits to both.
- All transitions must honor `prefers-reduced-motion: reduce` (fall back to instant / no animation).
- Do NOT change footer fill mechanics or the footer JS — it navigates via `window.location.href`, which inherits the view transition automatically.
- Commit after each task.

---

### Task 1: View Transitions slide-up on both pages

Adds the native cross-document transition so every same-origin navigation (menu links, footer pill) animates as the new page sliding up over the old.

**Files:**
- Modify: `index.html` (CSS block, after `body.preloading { overflow: hidden; }` at line 36)
- Modify: `projects.html` (CSS block, after `body.preloading { overflow: hidden; }` at line 36)

**Interfaces:**
- Produces: a `@keyframes vt-slide-up` + `::view-transition-*(root)` rule set present on both documents. No JS surface.

- [ ] **Step 1: Add the CSS to `index.html`**

Find this line (≈line 36):

```css
  body.preloading { overflow: hidden; }
```

Insert immediately after it:

```css

  /* ---------- Cross-page transition: new page slides up over the old ---------- */
  @view-transition { navigation: auto; }
  ::view-transition-old(root) { animation: none; }                 /* old stays put underneath */
  ::view-transition-new(root) { animation: vt-slide-up .5s var(--ease) both; }
  @keyframes vt-slide-up { from { transform: translateY(100%); } to { transform: none; } }
  @media (prefers-reduced-motion: reduce) {
    ::view-transition-new(root) { animation: none; }
  }
```

- [ ] **Step 2: Add the identical CSS to `projects.html`**

Find the same line (≈line 36) in `projects.html`:

```css
  body.preloading { overflow: hidden; }
```

Insert the exact same block from Step 1 immediately after it.

- [ ] **Step 3: Commit**

```bash
git add index.html projects.html
git commit -m "Pages: cross-document view-transition (new page slides up over old)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 2: Wire menu links (Home / Projects) on both pages

Currently every menu link is `href="#"`. Point Home → `index.html`, Projects → `projects.html`. Combined with Task 1, clicking them animates the slide-up.

**Files:**
- Modify: `index.html` (BUILD nav, ≈lines 1583–1584)
- Modify: `projects.html` (BUILD nav, ≈lines 1883–1884)

**Interfaces:**
- Consumes: the view-transition rules from Task 1 (links navigate same-origin → animate).

- [ ] **Step 1: `index.html` — Home link**

Replace:

```html
              <a class="m-link" href="#"><span class="rv-mask"><span class="rv">Home</span></span></a>
```

with:

```html
              <a class="m-link" href="index.html"><span class="rv-mask"><span class="rv">Home</span></span></a>
```

- [ ] **Step 2: `index.html` — Projects link**

Replace:

```html
              <a class="m-link" href="#"><span class="rv-mask"><span class="rv">Projects</span></span></a>
```

with:

```html
              <a class="m-link" href="projects.html"><span class="rv-mask"><span class="rv">Projects</span></span></a>
```

- [ ] **Step 3: `projects.html` — Home link**

Apply the exact same replacement as Step 1 (`href="#"` → `href="index.html"` on the `Home` `m-link`).

- [ ] **Step 4: `projects.html` — Projects link**

Apply the exact same replacement as Step 2 (`href="#"` → `href="projects.html"` on the `Projects` `m-link`).

- [ ] **Step 5: Commit**

```bash
git add index.html projects.html
git commit -m "Header menu: wire Home -> index.html, Projects -> projects.html

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 3: Gate the preloader to once per session

The `index.html` preloader must play only on the first open of the site in a session. Other arrivals (including coming back from Projects) skip it and reveal the site synchronously, so the incoming view-transition snapshot shows the hero — never a flash of preloader.

**Files:**
- Modify: `index.html` — add inline `<head>` script (after `<title>` at line 6), add CSS skip rules (after the Task 1 block), guard `run()` and set the flag (≈lines 2231 and 2264–2266), set flag in `projects.html`.
- Modify: `projects.html` — add inline `<head>` script (after `<title>` at line 6).

**Interfaces:**
- Consumes: nothing from prior tasks (CSS placement is after Task 1's block but independent).
- Produces: `sessionStorage` key `introPlayed` ('1' once the site has been opened); `<html class="intro-done">` marker when set.

- [ ] **Step 1: `index.html` — inline head guard (before first paint)**

Find (≈line 6):

```html
<title>Alex Foster</title>
```

Insert immediately after it:

```html
<script>
  /* If the intro already played this session, mark the document so CSS hides the
     preloader and reveals the site synchronously (keeps the incoming page-transition
     snapshot showing the hero, not a flash of preloader). */
  try { if (sessionStorage.getItem('introPlayed')) document.documentElement.classList.add('intro-done'); } catch (e) {}
</script>
```

- [ ] **Step 2: `index.html` — CSS to reveal instantly when skipping**

Find the closing of the Task 1 block (the `@media (prefers-reduced-motion: reduce)` that wraps `::view-transition-new(root) { animation: none; }`). Immediately after that block, insert:

```css

  /* Intro already seen this session → don't render the preloader at all */
  html.intro-done .preloader { opacity: 0; visibility: hidden; pointer-events: none; }
  html.intro-done body.preloading { overflow: visible; }
```

- [ ] **Step 3: `index.html` — early-return in `run()` and set the flag**

Find the start of `run()` (≈line 2231):

```js
    async function run() {
      await preload();
```

Replace with:

```js
    async function run() {
      if (document.documentElement.classList.contains('intro-done')) {
        // already revealed by CSS this session — just settle state and stop
        preloader.classList.add("done");
        document.body.classList.remove("preloading");
        return;
      }
      await preload();
```

- [ ] **Step 4: `index.html` — record the flag after the intro actually plays**

Find the tail of `run()` (≈lines 2263–2266):

```js
      // 5 — reveal the site
      preloader.classList.add("done");
      document.body.classList.remove("preloading");
    }
```

Replace with:

```js
      // 5 — reveal the site
      preloader.classList.add("done");
      document.body.classList.remove("preloading");
      try { sessionStorage.setItem('introPlayed', '1'); } catch (e) {}
    }
```

- [ ] **Step 5: `projects.html` — mark the site as opened**

Find (≈line 6):

```html
<title>Projects — Alex Foster</title>
```

Insert immediately after it:

```html
<script>
  /* Landing here counts as "the site is open" — so a later jump to Home won't replay the intro. */
  try { sessionStorage.setItem('introPlayed', '1'); } catch (e) {}
</script>
```

- [ ] **Step 6: Commit**

```bash
git add index.html projects.html
git commit -m "Preloader: play once per session; skip + reveal instantly on later arrivals

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 4: Projects hero on-load reveal (all at once)

On a direct load of `projects.html` (no transition), the hero background + content fade and rise in together. On an internal navigation it composes with the page slide-up and still reads as "all at once."

**Files:**
- Modify: `projects.html` — hero CSS (after `.pj-hero-title` rule ≈line 1579) and hero markup classes (≈lines 1946 and 1948).

**Interfaces:**
- Consumes: `var(--ease)`.
- Produces: `.pj-hero-reveal` class behavior (one-shot entrance animation).

- [ ] **Step 1: `projects.html` — add the reveal CSS**

Find the `.pj-hero-title` rule (≈line 1578):

```css
  .pj-hero-title { max-width: 800px; color: #fff; font-weight: 500;
    font-size: clamp(40px, 4.6vw, 64px); line-height: 1.0625; letter-spacing: -0.04em; }
```

Insert immediately after it:

```css
  /* Hero entrance — background + content arrive together (all at once) */
  .pj-hero-reveal { animation: pj-hero-in .7s var(--ease) both; }
  @keyframes pj-hero-in { from { opacity: 0; transform: translateY(24px); } to { opacity: 1; transform: none; } }
  @media (prefers-reduced-motion: reduce) { .pj-hero-reveal { animation: none; } }
```

- [ ] **Step 2: `projects.html` — add the class to the hero background**

Replace (≈line 1946):

```html
      <div class="pj-hero-bg"><img src="images/projects/hero.webp" alt="" decoding="async" /></div>
```

with:

```html
      <div class="pj-hero-bg pj-hero-reveal"><img src="images/projects/hero.webp" alt="" decoding="async" /></div>
```

- [ ] **Step 3: `projects.html` — add the class to the hero content**

Replace (≈line 1948):

```html
      <div class="pj-hero-inner">
```

with:

```html
      <div class="pj-hero-inner pj-hero-reveal">
```

- [ ] **Step 4: Commit**

```bash
git add projects.html
git commit -m "Projects: hero fades + rises in on load (background and content together)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 5: Verify in a real browser

Confirm the four behaviors hold together. Use puppeteer-core + the installed Chrome (project convention), or drive manually if preferred.

**Files:**
- None (observation only).

- [ ] **Step 1: Serve the site locally**

```bash
cd "/Users/valeriytsvigun/Alex Foster website" && python3 -m http.server 8000 >/tmp/afsrv.log 2>&1 &
```

- [ ] **Step 2: First-open preloader**

Open `http://localhost:8000/index.html` in a fresh session (clear sessionStorage / new context). Expected: full preloader plays ("Driven by ambition…" → images → whiteout → site).

- [ ] **Step 3: Menu navigation slides up, no preloader**

Open the menu, click **Projects**. Expected: Projects page slides up from the bottom over Home; hero is visible at the top of the slide; NO preloader. Open menu, click **Home**. Expected: Home slides up; NO preloader replay; hero (not preloader) is shown.

- [ ] **Step 4: Projects hero reveal on direct load**

In a fresh session, open `http://localhost:8000/projects.html` directly. Expected: hero background + title + pill fade and rise in together once.

- [ ] **Step 5: Footer pill → smooth slide**

On Home, scroll to the footer and keep scrolling so the pill fills. Expected: the moment it fills, the next page slides up smoothly (no abrupt hard cut). Same from Projects' footer back to Home.

- [ ] **Step 6: Reduced motion**

Re-run Step 3 with `prefers-reduced-motion: reduce` emulated. Expected: navigation is instant (no slide), pages still work, hero shows without the entrance animation.

- [ ] **Step 7: Stop the server**

```bash
kill %1 2>/dev/null; echo done
```

---

## Self-Review

- **Spec coverage:** §1 view-transition → Task 1; §4 menu links → Task 2; §2 preloader gating → Task 3; §3 hero reveal → Task 4; §5 footer (no change, inherits transition) → verified in Task 5 Step 5. All covered.
- **Placeholders:** none — every code step shows exact code.
- **Type/name consistency:** `introPlayed` and `intro-done` used consistently across Task 3 steps; `pj-hero-reveal` / `pj-hero-in` consistent in Task 4.
- **Note:** No automated tests exist in this repo; Task 5 is browser verification, matching project convention.
