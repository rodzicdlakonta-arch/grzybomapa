---
name: project-grzybomapa-bugs-v129
description: GrzyboMapa bug history v1.29.71–77 — original 9 bugs all fixed; 3 new real-device bugs fixed in v1.29.77
metadata: 
  node_type: memory
  type: project
  modified: 2026-09-21T15:43:09.323Z
  originSessionId: 929571ba-2a04-4f56-bd84-de41f103cfe8
---

All 9 originally-reported mobile bugs were fixed in v1.29.71–76. Three additional bugs found during real-device testing were fixed in v1.29.77.

**How to apply:** After the next push, ask user which remaining issues (if any) they see on real device.

## Fixed in v1.29.71–76 (original 9 bugs)

1. Portrait — right layers panel (mc) slightly too big / bottom cut off ✅
2. Landscape — Grzyby (🍄) button sometimes needs two presses ✅
3. Landscape — left sidebar arrow shows "open" with no visible panel ✅
4. Landscape — right layers panel arrow (◄/mc-tab) not visible (initial fix) ✅
5. Landscape — layers panel can't scroll far enough to reach BDL ✅
6. Landscape — opening scan panel should close sidebar and layers panel ✅
7. Portrait AND landscape — species list (#ml) cut off by bottom nav bar ✅
8. Leaflet attribution link moved from corner ✅
9. Heatmap opacity too high ✅

## Fixed in v1.29.77 (new real-device bugs)

**A — Double-tap to close list after info page close:**
`pcls` handler removed ALL `.mob-tab` active states, so `#mob-list` was not active → first Grzyby tap re-activated it (no visual change), second tap closed. Fix: after removing all active, explicitly re-add `active` to `#mob-list`.

**B — mc-tab arrow disappears when right panel collapsed (landscape):**
`#mc.mc-col{transform:translateX(100%)}` slid entire mc off screen. In landscape, mc-tab is at `left:0` (inside mc) so it went off-screen too. Fix: landscape CSS overrides `#mc.mc-col{transform:none!important}` and `#mc.mc-col #mc-body{display:none!important}` — mc stays at right:0 as a 14px-wide strip, only mc-tab visible.

**C — Info panel stays open when scan panel opens:**
`_openApPanel()` closed sidebar and mc but never closed `#ip`. Fix: added `#ip.classList.remove('open','ip-mini')` inside `_openApPanel()` before opening `#ap`.

## Pending

- Desktop fix: `@media(max-height:700px)and(orientation:landscape)and(max-width:1024px)` — added `max-width:1024px` in v1.29.76 to prevent landscape CSS hitting laptops (1366×768 viewport ~648px was matching the old query).
- User wants to install superpowers + frontend-design plugins for UI design work — needs interactive terminal:
  - `claude /plugin marketplace add obra/superpowers-marketplace`
  - `claude /plugin install superpowers@superpowers-marketplace`
  - `claude /plugin marketplace add anthropics/claude-code`
  - `claude /plugin install frontend-design@claude-code-plugins`
