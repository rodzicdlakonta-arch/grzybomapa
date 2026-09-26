---
name: feedback-grzybomapa-mobile-nav
description: "Critical architecture lessons for GrzyboMapa v1.30 mobile nav: DOM timing, CSS cascade, filter teleport, globals pattern"
metadata: 
  node_type: memory
  type: feedback
  modified: 2026-09-21T21:57:31.045Z
  originSessionId: 996fcca4-52e4-4e3f-ae23-32147383b3ce
---

## New mobile UI elements added in v1.30.0

| Element | Purpose |
|---------|---------|
| `#mob-top` | Fixed 52px top bar (logo + search icon), portrait only |
| `#mob-chips` | Fixed chip row 44px below top bar, horizontal scroll, portrait only |
| `#mob-pill` | Floating pill (📋 Lista / 🗺️ Mapa), toggles `#sidebar` |
| `#filter-modal` | Slide-up bottom sheet for filters, contains `#filter-modal-body` |
| `#mob-handle` | 14×48px pull-tab on left edge, landscape phones only |
| `#sb-tabs` | Desktop sidebar tabs (Grzyby/Filtry), hidden on mobile |

## DOM Timing Rule — ALL new mobile JS IIFEs must go in the SECOND script block

`index.html` has two `<script>` blocks:
1. **Main script** (starts ~line 42000, before `#mob-top`/`#mob-chips` HTML) — runs BEFORE new elements exist
2. **Mob-nav handler script** (at end of file, after all HTML) — runs AFTER elements exist

**Never** put `#mob-chips`, `#mob-pill`, `#filter-modal`, `#mob-handle` IIFEs in the main script — `querySelectorAll` returns empty NodeList, event handlers never attach.

**Why:** Lesson from this session — chips IIFE was initially in wrong block; `.mchip` onclick handlers silently failed.

## CSS Cascade Rule — `!important` required for media query overrides

`#mob-top{display:none}` and `#mob-chips{display:none}` are appended near the END of `<style>`, which comes AFTER `@media(max-width:767px)`. So the global rule wins unless media query uses `!important`:

```css
@media(max-width:767px){
  #mob-top{display:flex!important}   /* needs !important */
  #mob-chips{display:flex!important} /* needs !important */
}
```

## Filter Teleport Pattern

`#filters-panel` + `#season-bar` live in `#sb-top` (desktop sidebar). On mobile they're teleported into `#filter-modal-body` by the filter-modal IIFE at page load:

```js
if(window.innerWidth<=767){
  const fp=document.getElementById('filters-panel');
  const sb=document.getElementById('season-bar');
  if(fp)body.appendChild(fp);
  if(sb)body.appendChild(sb);
}
```

On desktop they stay in `#sb-top` — `#filter-modal` never opens on desktop. This means the same filter DOM elements work in both contexts.

## Global Interfaces

- `window._closeSidebar` — portrait IIFE sets it to `_closeMobList`; landscape IIFE overrides to its own (with `_update()` call). Scanner's `_openApPanel()` calls it at line ~41646.
- `window._openMobList` / `window._closeMobList` — set by pill IIFE for cross-IIFE access
- `window.syncMobChips` — set by chips IIFE; called by category filter button onclick to keep chips in sync with `activeCats`

## Desktop tabs via body classes

`sbt-grzyby` / `sbt-filtry` on `<body>` control which sidebar section is visible. CSS rule example:

```css
@media(min-width:768px){
  body.sbt-filtry #sw,body.sbt-filtry #ml{display:none!important}
  body.sbt-grzyby #filters-panel,body.sbt-grzyby #season-bar{display:none!important}
}
```

## Media query breakpoints

- `@media(max-width:767px)` — portrait phones ONLY (landscape phones are ~812px wide, above threshold)
- `@media(max-height:700px) and (orientation:landscape) and (max-width:1366px)` — landscape phones specifically (1366px excludes most laptops; DO NOT use `pointer:coarse` — unreliable on Pixel 9 Pro XL, see [[feedback-pixel9-pointer-coarse]])
- `@media(min-width:768px)` — desktop + tablet + landscape phones

**Why:** `#mob-handle` is visible at 812×375 (landscape) because 812px > 767px. At that width, desktop CSS applies too, so `#sb-tabs` shows inside the sidebar overlay — this is intentional and functional.
