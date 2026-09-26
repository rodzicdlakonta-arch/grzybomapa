---
name: feedback-grzybomapa-f2-debug
description: "GrzyboMapa: use F2 debug overlay proactively when adding or testing features"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 0f66b839-ccd9-465e-87a1-2bddb4202eec
  modified: 2026-09-20T13:59:49.276Z
---

Use the F2 debug overlay as a first-class development tool on GrzyboMapa — not just for the grid scanner.

**Why:** User explicitly asked for this. The overlay is already wired up, auto-updates after background fetches, and shows map-layer visualizations. It's the fastest way to verify what the code is actually computing without console-diving.

**How to apply:** Whenever adding a new feature that computes per-point or per-area data (terrain checks, scoring, heatmap density, BDL matches, etc.), extend `_lastDebugData` to include the relevant values and add them to the F2 panel. When testing existing features, check F2 first before reading raw JS — it gives an instant visual sanity check on map layer + numeric summary.

Examples of what to surface via F2: grid cell habitats, score distributions, BDL stand locations, Overpass fetch results, any intermediate per-point computation.

**Current F2 capabilities (v1.29.24):**
- Scan grid: habPts coverage by type (forest/meadow/wetland/urban/farmland), CLC/wyrStands/BDL counts
- Heatmap section: scoring type, all opt flags ✓/✗ (skraj pól / skraj wód / mokradło / łąki / iglaste / liściaste / góry / urban-tolerant), pH, elev, BDL codes, heat point count
- Best-spot section: ref point, total pts→bucket count, top 6 buckets table with ★ winner
- **Map edge overlays**: blue dashed outlines for waterEdge polygons, orange for farmlandSpec/farmland polygons, pink for urbanSpec/URBAN_DATA polygons — drawn on F2 open, cleared on F2 close
- Live update: `_buildBaseHeat` calls `_refreshDebugOverlay()` so F2 updates in real-time as species switch

**Key debug variables:** `_lastDebugData`, `_dbgMapLayer` (habPts dots), `_dbgEdgeLayer` (edge outlines)
