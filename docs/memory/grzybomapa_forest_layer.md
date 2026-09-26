---
name: grzybomapa-forest-layer
description: "GrzyboMapa forest layer architecture — canvas trees, spatial index, tooltip pipeline, data sources (load when touching forests/tooltips)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 32d749b0-8d6e-480f-870f-25e4bfc347de
  modified: 2026-09-22T03:20:59.714Z
---

## Data sources (priority order for tooltip)

1. **`wyrForests[]`** (line ~52120) — live OSM polygons from detailed layer (`loadDetailTerrain()`); `{type,name,poly,_bb?}`; type from `getTreeType(tags)` which reads `leaf_type`, `species`, `taxon`, `adr_les`/PGL LP heuristic → 'pine'/'oak'/'birch'/etc. Most accurate. Grows as user pans with detailed layer on. Never cleared.
2. **`bdlStore[]`** — user-clicked BDL compartments `{k,lat,lng,cd,age}`; `cd` can be "SO.DB" (dot-separated multi-species); sparse (only clicked points).
3. **`WYR_TREES[]`** (line ~43620) — static ~8k BDL survey pts for Wyrzysk area; `[lat,lng,type,name]`; `name[3]` is SOMETIMES a forest stand name ("Wilcze Doły"), not species — always filter with genus regex `/Sosna|Brzoz|Modrzew|Świerk|Dąb|Olch|Buk|.../` before using as key.

## Spatial index & lookup functions

- **`_FSI`** — 0.1°×0.15° bucket index over FORESTS array; built at startup
- **`_fBboxes[]`** — parallel bbox array for FORESTS
- **`_pip(lat,lng,pts)`** — ray-cast point-in-polygon
- **`_forestTypeAt(lat,lng)`** — returns '311'|'312'|'313' from static FORESTS via _FSI + _pip; returns `null` outside all forests
- **`_namedF[]` / `_namedFBB[]`** — FORESTS entries with `f.n` (name) but no `f.c`; named natural areas (Wilcze Doły etc.)
- **`_namedForestAt(lat,lng)`** — PIP in _namedF; returns forest name string or null
- **`_wyrForestAt(lat,lng)`** — PIP in wyrForests[] with lazy `f._bb` bbox cache; returns `{type,name,poly}` or null

## Canvas tree grid (`_TreeGL`)

- `L.GridLayer` subclass; 256px tiles; zoom 12+
- Groups polygons by type ('311'/'312'/'313') per tile
- `ctx.clip()` on all polygons of a type → hard boundary
- Emoji: 311=🌳, 312=🌲, 313=alternating 🌲🌳 by grid parity
- Pane: 'treePins' z-index 450 (above forests, below markers)

## Hover tooltip pipeline

```
mousemove → _forestTypeAt() → if null, hide tip
→ _namedForestAt() → fname (or null)
→ wyrForests.length > 0? → _wyrForestAt() → show type label + TREE_NAME_PL[type]
→ bdlStore nearby? → split cd by '.', count _BDL_SP per sub-code
→ WYR_TREES scan (R=0.02°) → count by genus-filtered name
→ fallback: 'Las'
```

## Key constants (near tooltip code)

- `_FOREST_LABEL` — `{pine:'Bór sosnowy', spruce:'Bór świerkowy', larch:'Las modrzewiowy', birch:'Brzezina', oak:'Las dębowy', alder:'Ols', beech:'Buczyna', mixed:'Las mieszany'}`
- `_BDL_SP` — BDL code → Polish name (SO, MD, SW, BRZ, DB, DBB, GB, OL, BK, JS, LP, AK, OS, WB, TP + aliases S, B, OLS)
- `TREE_NAME_PL` (line 44033) — type→name same as `_FTN` but at a different scope
- `_DET_COLORS` — `{311:'#4a8a2a', 312:'#255015', 313:'#2d6a15'}`

## FORESTS array structure

- Line ~638–~11800 in index.html
- 10,510 OSM polygons from original fetch (52.5-53.8°N, 16.5-18.2°E) — but had gaps
- +201 Bakowo/Osiek (53.05-53.17°N, 17.25-17.45°E) added v1.30.29
- +598 Glesno/Kraczki/Ruda (53.08-53.27°N, 17.15-17.65°E) added v1.30.30
- Named forests (no `c:`, not rendered as polygons): Wilcze Doły, Lisi Kąt, Rezerwat Zielona Góra, etc.
- 97.4% are c:'313' (mixed) — Polish OSM rarely has leaf_type tag

## Pending improvements

- Use `_wyrForestAt()` in scanner terrain scoring (not just tooltip) so pine species score correctly in bory sosnowe
- Replace WYR_TREES with live BDL data fetched per viewport
- Fetch more missing forest polygons on demand (still gaps outside fetched sub-areas)
