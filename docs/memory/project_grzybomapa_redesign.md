---
name: project-grzybomapa-redesign
description: GrzyboMapa nav redesign — shipped v1.30.0; AllTrails pattern; pivot options documented
metadata: 
  node_type: memory
  type: project
  modified: 2026-09-21T22:28:19.994Z
  originSessionId: 4327bfd7-0ccf-48f3-96e2-329601ace473
---

## Status: COMPLETE — shipped v1.30.0 (2026-09-21)

AllTrails pattern implemented: slim top bar (`#mob-top`) + horizontal chip row (`#mob-chips`) + full-screen map + floating pill toggle (`#mob-pill`) for species list. Desktop: Grzyby/Filtry sidebar tabs (`#sb-tabs`). Landscape: pull-tab handle (`#mob-handle`).

Backup: `index.backup-v1.29.77.html` in grzybomapa folder.

## Pivot options (if users complain list is hard to find)

- **Option B — Google Maps tabs** (medium cost): re-enable `#mob-nav` (`display:none` → flex), rewrite tab handlers to show/hide sheet sections exclusively. ~50 lines.
- **Option C — Swipe drawer** (high cost): no DOM scaffolding, needs touch gesture handler + new CSS from scratch.
- Option A (current AllTrails) pivot back: `#mob-nav` still in DOM.

## Key architecture decisions

- `#filters-panel` + `#season-bar` teleported into `#filter-modal-body` on mobile via JS at page load
- `window._closeSidebar` / `window._openMobList` / `window.syncMobChips` — cross-IIFE globals
- All new mobile JS IIFEs must go in the SECOND script block (after HTML elements exist)
