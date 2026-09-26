---
name: grzybomapa-tech
description: "GrzyboMapa tech stack — data objects, layout, filter/sort, UI components. LINE ANCHORS included."
metadata: 
  node_type: memory
  type: project
  originSessionId: 0f86b839-ccd9-465e-87a1-2bddb4202eec
  modified: 2026-09-22T03:20:23.157Z
---

## Tech stack

- **Map:** Leaflet.js 1.9.4 + leaflet.heat@0.2.0 + leaflet.draw (area drawing)
- **All deps via CDN** — no npm, no build step. Single `index.html`, everything inline.
- **Map zoom limits:** minZoom:8, maxZoom:15

## Data arrays — line anchors (index.html)

| Array | Line | Notes |
|-------|------|-------|
| `REGIONS` | **475** | 39 named regions `{c:[lat,lng], r:radius}` |
| `FORESTS` | **517** | CLC polygons `{p:[[lat,lng],...], c:'311'|'312'|'313'}` |
| `LANDUSE` | **905** | `{t:'wetland'|'meadow'|'pasture'|'farmland'|'water'|'river', p:[...]}` |
| `URBAN_DATA` | **19959** | Raw `[[lat,lng],...]` — **no `.p` wrapper!** |
| `MUSHROOMS` | **30942** | Species data objects |
| `MUSH_ICON_CFG` | **31978** | Icon shape + color |
| `getMushSVG(mid)` | **32084** | Returns SVG string |
| `MUSHROOM_LOCAL` | **32153** | `{s:'c'|'f'|'r'|'n', n:'note'}` per mid |
| `bdlStore` | **42258** | `[{lat,lng,cd,age,k}]` — BDL stands from user clicks; `cd` can be "SO.DB" (multi-species dot-separated) |
| `wyrForests` | **52120** | `[{type,name,poly,_bb?}]` — live OSM forests from detailed layer; grows as user pans; `_bb` lazy-cached bbox |
| `WYR_TREES` | **~43620** | `[lat,lng,type,name]` — static BDL survey pts, Wyrzysk area; `type`='pine'|'birch'|'mixed'; `name` may be species OR forest stand name (filter with genus regex) |

## UI functions — line anchors

| Function | Line |
|----------|------|
| `selectMushroom(mid,e)` | **40961** |
| `clearHeat()` | **41013** |
| `_showBestSpot(mid)` | **41061** |
| `showInfoPanel(mid)` | **41115** |
| `_mkMushCard(mid,m)` | **42072** |
| `renderList()` | **42082** |

## Data objects

- `MUSHROOMS` — species data: Polish name, Latin, cat, seasons, terrain, soil, ph, elev, habitat, desc, edib, danger, lookalikes, weights
- `MUSH_ICON_CFG` — icon shape + color. Shapes: `cap`, `flat`, `chan`, `aman`, `para`, `brac`, `puff`, `more`, `clus`, `cora`, `lion`, `cali`, `truf`
- `MUSHROOM_LOCAL` — regional presence: `{s:'c'|'f'|'r'|'n', n:'note'}`. `'n'`=absent (removed from heatmap); `'c'`=+12%, `'f'`=+6%, `'r'`=-20%
- `FORESTS` — CLC forest polygons: `{p:[[lat,lng],...], c:'311'|'312'|'313'}` (311=broadleaf, 312=conifer, 313=mixed)
- `LANDUSE` — water/river/wetland/meadow/farmland polygons: `{t:'wetland'|..., p:[...]}`
- `URBAN_DATA` — urban polygons. **Raw `[[lat,lng],...]` arrays — NOT `{p:[...]}` objects.** See [[grzybomapa-scanner]].
- `REGIONS` — 39 named regions: `{c:[lat,lng], r:radius}` — Krajna/Wyrzysk focused
- `bdlStore` — BDL stands: `{lat, lng, cd, age}` where `cd`=tree species code, `age`=stand age

## Key layout

**Desktop:** `body{display:flex}` — `#sidebar` (320px, resizable) | `#sb-resize` | `#mw` (flex:1, map)
- `#ip` (info panel): `position:fixed; right:-400px` → `right:0` when `.open`
- `#ap` (analysis panel): `position:fixed; bottom:-100%; left:320px` → `bottom:0` when `.open`
- Weather panel `#wx` auto-collapses when species selected

**Mobile (≤767px):**
- `#sidebar` → `position:fixed` bottom sheet, 80vh, `.mob-open` slides it up
- `#mob-nav` → fixed bottom bar (56px): 🗺️ Mapa / 🍄 Grzyby / 🔍 Filtry
- `#ip` → full-width, slides up via `transform:translateY`
- All bottom overlays +56px offset for nav bar

## Filter/sort system (v1.27.0)

- `activeCats` — Set of categories
- `activeSeasons` — Set of month numbers
- `activeHabs` — Set of keywords: forest, meadow, wetland, conifer, broadleaf, mountain, urban, sandy, alpine, park, dung
- `activeSortBy` — 'default'|'name-az'|'name-za'|'rarity-common'|'rarity-rare'|'edibility'|'id-easy'

`renderList()` (L42082) re-renders sidebar on any filter change. Sort by id-easy uses `SPECIES_DIFF[mid]` ascending.

## Multi-select heatmap

Shift+click cards to combine species; long-press on mobile (500ms); × to clear.
`selectedMids` Set tracks all active; `_midHeatCache` stores heat pts per species; `_buildBaseHeat(mid)` extracted helper.

## UI components

**Season bar:** `.sm.cur` class + `::after` triangle marks current month in info panel.
**"W sezonie" badge:** `_CURMONTH` const at top of `_mkMushCard`; green badge on list cards.
**Photo lightbox:** click cycles 1x→2x→4x→1x; scroll wheel 1x–5x; `_openLightbox(src,attr)` global IIFE. Attribution: `#photo-lb-attr`.
**Pip rating system:** fork SVG (smak), magnifying glass (trudność ID), star (rzadkość); `--pip-empty` CSS var.
**GPS button:** `<a>` element; `a.textContent='○'` + spin animation during fetch; `busy` flag prevents double-click.
**Sidebar tab:** `position:fixed; left:{width}px` when collapsed so tab stays visible at x=320.
**Zoom debug pill:** `_zdbg` at `bottom:22px;right:6px`, rgba(0,0,0,.28) bg, very low visibility.
**Nominatim weather:** `zoom=14` + suburb fallback for precise location names.
