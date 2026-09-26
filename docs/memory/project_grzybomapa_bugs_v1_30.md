---
name: project-grzybomapa-bugs-v130
description: GrzyboMapa open bugs v1.30.x — real-device issues (Pixel 9 Pro XL); track until all confirmed fixed on device
metadata: 
  node_type: memory
  type: project
  modified: 2026-09-21T22:23:17.478Z
  originSessionId: 996fcca4-52e4-4e3f-ae23-32147383b3ce
---

**Why:** Found during real-device testing on Pixel 9 Pro XL after the v1.30.0 navigation redesign.

## Fixed and device-confirmed ✅

- **Landscape MQ not matching** — `max-height:700px and orientation:landscape` + `isLargeLS()` using `innerHeight<700` (v1.30.4) ✅
- **Chips: only static category filters** — rewritten for all active filters (v1.30.4) ✅
- **Chips: hallucinogenic/ulubione/w sezonie missing** — syncMobChips handles activeCats without static chips (v1.30.5) ✅
- **Chips: Trujące always inactive** — `data-mcat="dangerous"` → `"poisonous"` (v1.30.5) ✅
- **🎛️ Filtry chip scrolling out of view** — pinned to firstChild (v1.30.5) ✅
- **Info panel stuck after LS→portrait rotation** — orientationchange resets scrollTop (v1.30.5) ✅
- **Portrait layers arrow invisible when collapsed** — overflow:visible + min-width fix (v1.30.5) ✅
- **Portrait layers arrow disappears when layers open** — `#mc{overflow:hidden}` → `overflow:visible` (v1.30.6) ✅
- **Portrait layers arrow too small (14×42px)** — bumped to 28×48px (v1.30.6) ✅
- **Zoom number pushed up by old JS** — removed `_zdbg.style.bottom='66px'` (v1.30.3) ✅
- **Sidebar border visible (1px) when closed in landscape** — `translateX(-302px)` instead of -300px (v1.30.11) ✅
- **BDL unreachable in landscape layers panel (MOST PERSISTENT BUG)** — `#mc{position:fixed!important}` in landscape MQ removes `#mc` from `#mw{overflow:hidden}` stacking context; combined with `#mc-body{padding-bottom:80px}` (v1.30.13) ✅ **DEVICE CONFIRMED**
- **Landscape MQ not applying on Pixel 9 Pro XL** — `pointer:coarse` not matching on Pixel → switched to `max-width:1366px` in both CSS MQ and `isLargeLS()` (v1.30.13) ✅ **DEVICE CONFIRMED (BDL fix proves MQ applies)**
- **Info panel minimize broken after LS→portrait rotation** — minimize handlers (`ipMin.click`, `ipHandle.click`, swipe) were inside `if(innerWidth<=767)` IIFE which never ran when page loaded in landscape; moved to unconditional IIFE (v1.30.12) ✅
- **PC desktop regressions from landscape MQ** — filters, right menu padding, sidebar layout broken on small-window desktops; fixed by adding `and (max-width:1366px)` to landscape MQ (v1.30.13) ✅

## Fixed in v1.30.7, Playwright-confirmed ✅

- **Portrait info panel name at very top** — `padding-top:6px` on `#ip` in portrait MQ ✅ (Playwright 390×844)
- **LS info panel best-spot button cut off at bottom** — `#pbody{padding-bottom:calc(56px + env(safe-area-inset-bottom))}` ✅ (Playwright 900×400, button fully visible)
- **LS list last item still cut off** — padding-bottom 48px → 72px on `#ml` ✅ (user confirmed)

## Fixed in v1.30.14–16 (2026-09-22) ✅

- **Sidebar sticks out 2px when closed** — `translateX(-302px)` → `-320px`; min-width:320px was the actual width
- **Old collapsible filter UI shown in landscape** — `#sb-tabs{display:none!important}` removed → Grzyby/Filtry tabs now show; removed landscape overrides that nullified tab switching
- **Mob-handle arrow reversed** — `‹`/`›` swapped; closed=`›`, open=`‹`
- **Mob-handle `left:300px` short when open** — sidebar computed width is 320px (min-width wins); changed to `left:320px`
- **Filters auto-expand** — clicking Filtry tab or opening portrait filter modal now auto-expands the collapsible body
- **Filters panel too small in landscape Filtry tab** — `body.sbt-filtry #sb-top{max-height:100vh!important;flex:1}`

## Root-cause lessons learned

- `overflow:hidden` on a positioned element clips absolutely-positioned children that stick outside (mc-tab at left:-28px)
- Nested scroll conflicts: child `.cp.scroll{flex:1;overflow-y:auto}` inside parent `#mc-body{overflow-y:auto}` means touch scroll grabs the parent first; use single scroll on parent with `overflow:visible` on children
- **`pointer:coarse` unreliable on Pixel 9 Pro XL** — Chrome Android on this device does NOT report `pointer:coarse`, likely due to stylus/AI pen flag or Chrome version. Use `max-width:1366px` instead to discriminate phones (≈1040px CSS width) from laptops (≥1440px). See [[feedback-pixel9-pointer-coarse]]
- `#ip.scrollTop` not reset on reopen — always reset before AND 320ms after adding 'open' class
- `position:fixed!important` on `#mc` in landscape MQ = key to scrollable layers panel; removes element from any ancestor stacking context including `#mw{overflow:hidden}`
- Info panel handlers inside `if(window.innerWidth<=767)` run ONCE at page load — if page loads in landscape (width>767) these handlers never attach; always use unconditional IIFEs with optional chaining for handlers needed in both orientations
