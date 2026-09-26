---
name: feedback-grzybomapa-landscape-mobile
description: "GrzyboMapa mobile landscape CSS/JS lessons — media query pitfalls, touch draw, #ap panel, ls-toggle button"
metadata: 
  node_type: memory
  type: feedback
  modified: 2026-09-21T15:43:19.058Z
  originSessionId: 75267ebb-051f-4f3f-b407-6f25d684a1c9
---

Critical lessons from v1.29.59–v1.29.61 mobile landscape work.

## 1. Never use `display:flex!important` to fight JS inline display control

Rule: If JS controls `element.style.display`, don't also set `display:flex!important` in CSS.

**Why:** CSS `!important` beats inline styles per cascade spec. The button was *always* shown by CSS in landscape, but then JS tried to hide it with `element.style.display='none'` (inline), which lost to `!important`. Result: button always visible (wrong) OR button never toggleable. Removed `display:flex!important`; let JS be sole display gatekeeper.

**How to apply:** Keep CSS for positioning/sizing only. JS sets `display: 'flex'` or `'none'` for conditional visibility.

## 2. `@media(max-height:500px)` too narrow — use `max-height:700px` for landscape phones

Rule: Use `@media(max-height:700px)and(orientation:landscape)` to target all landscape phones.

**Why:** Some phones (e.g., newer large-screen Android with 2x DPR, or phones with browser chrome reducing viewport) report CSS viewport height of 550-650px in landscape. `max-height:500px` missed them. Also, `isMobLandscape()` JS check used `innerHeight<=500` — same issue.

**How to apply:** CSS threshold `700px`, JS check `window.innerWidth > window.innerHeight && window.innerWidth <= 1024` (any landscape on phone/small tablet, no arbitrary height threshold).

## 3. Large phones (width > 767px) in landscape skip the main mobile media query

Rule: Always add an `orientation:landscape and max-height:700px` block for rules that must apply to all landscape phones.

**Why:** `@media(max-width:767px)and(orientation:landscape)` catches small phones but NOT large phones (Galaxy S24, iPhone Pro Max, etc.) whose CSS width in landscape is 800–950px. They fall through to the desktop defaults. This caused `#ap{left:320px}` (desktop sidebar clearance) to apply on large landscape phones, pushing the scanner panel 320px off-screen.

**How to apply:** Put universal landscape mobile rules (like `#ap{left:0!important}`) in the `max-height:700px` block, not just the `max-width:767px` block.

## 4. Leaflet's `map.on('touchstart', handler)` does NOT provide `e.latlng`

Rule: For touch-based drag drawing, always use DOM `addEventListener` on `map.getContainer()`, not Leaflet's `map.on('touchstart')`.

**Why:** Leaflet synthesizes `mousedown`/`click`/`dblclick` from touch events, but does NOT forward raw touch events with `latlng`. `map.on('touchstart', handler)` fires but `e.latlng` is undefined → `e.latlng.lat` throws silently → rectangle draw never starts.

**How to apply:** 
```javascript
map.getContainer().addEventListener('touchstart', e => {
  if (!drawing) return;
  e.preventDefault(); // prevent map interference
  const t = e.touches[0];
  const r = map.getContainer().getBoundingClientRect();
  const ll = map.containerPointToLatLng([t.clientX-r.left, t.clientY-r.top]);
  // use ll as latlng
}, {passive: false}); // passive:false required for preventDefault
```
Add touchmove/touchend the same way. `{passive:false}` is required to call `preventDefault()`.

## 5. `map.on('click')` and `map.on('dblclick')` DO work on touch (reliable)

**Why:** Leaflet synthesizes these from taps. The polygon tool (`map.on('click', _onPolyClick)`) works fine on mobile. Only drag-based gestures (mousedown → mousemove → mouseup) fail with Leaflet's synthetic events on touch.

## 6. In landscape with vertical mob-nav, `#ap.open{bottom:56px}` is wrong

Rule: Override `bottom:56px` to `bottom:0` in landscape media queries.

**Why:** Portrait mobile CSS sets `#ap.open{bottom:56px}` to clear the horizontal bottom nav bar. In landscape, mob-nav becomes a vertical right-side bar — nothing at the bottom — so 56px gap is wasted. Add `#ap.open{bottom:0}` in the landscape blocks.

## 8. In landscape, `#mc.mc-col` must NOT use `transform:translateX(100%)` — override to `none`

Rule: In landscape media query, add `#mc.mc-col{transform:none!important}` and `#mc.mc-col #mc-body{display:none!important}`.

**Why:** In landscape, `#mc-tab` is moved inside mc (`left:0`) so `overflow:hidden` on mc doesn't clip it. But the base CSS `#mc.mc-col{transform:translateX(100%)}` slides the whole mc (including mc-tab) off-screen right. With `transform:none`, mc stays at `right:0` and only mc-body is hidden — mc-tab remains visible as a 14px strip at the viewport's right edge.

**How to apply:** Any time mc-col collapse is used in landscape, the landscape media query MUST override the transform. Otherwise mc-tab disappears when user collapses the layers panel.

## 7. `_update()` call needed on IIFE init for landscape-aware UI

Rule: Call `_update()` at the end of any IIFE that manages conditional visibility.

**Why:** MutationObserver and resize/orientationchange events fire for STATE CHANGES, not initial state. If the page loads in landscape with ip already open (e.g., from a link), no mutation fires and the button stays hidden. One `_update()` call at the end of the IIFE catches this.
