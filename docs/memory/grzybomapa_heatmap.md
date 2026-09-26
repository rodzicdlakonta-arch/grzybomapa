---
name: grzybomapa-heatmap
description: "GrzyboMapa heatmap system — genHeat opts, soft limits, BDL bonus, derived opts per species. LINE ANCHORS included."
metadata: 
  node_type: memory
  type: project
  originSessionId: 0f86b839-ccd9-465e-87a1-2bddb4202eec
  modified: 2026-09-22T06:10:52.947Z
---

## Line anchors (index.html)

| Symbol | Line |
|--------|------|
| `genHeat(weights,seed,total,opts)` | **43552** |
| `_inUrban(la,ln)` | **43546** |
| `_inFarmland(la,ln)` | **43550** |
| `_buildBaseHeat(mid)` | **52228** |
| `selectMushroom(mid,e)` | **52312** — destructures `_buildBaseHeat` result |
| Forest filter in `genHeat` | ~**43650** (search for `coniferPref&&!broadleafPref`) |

## genHeat system

`genHeat(weights, seed, total, opts)` **L32349** — generates heat points for selected species.

**opts fields:**
- `openHab` — meadow/open-ground species (uses LANDUSE meadow polygons)
- `wetHab` — wetland/bog species (uses LANDUSE wetland polygons)
- `farmlandSpec` — farmland/garden species (uses **farmland polygon EDGES** via `_farmlandEdgePts` — boundary vertices ±0.003°/0.004°, same as waterEdge; 60% edge pts + 40% open scatter; NOT interior `_sp` since crop interiors = tractor-damaged)
- `waterEdge` — bonus points near water bodies (points placed at polygon edge ±~250m, NOT inside water — `_waterEdgePts` picks a boundary vertex then offsets ±0.005°/0.007°; was ±0.003°/0.004°=~150m before v1.30.42)
- `coniferPref` / `broadleafPref` — filter CLC forests + skip mismatched BDL trees
- `mountainSpec` — downweight flat regions (lat>51.5), upweight mountain regions (lat<50.5)
- `avoidUrban` — soft urban avoidance (always true)
- `urbanTolerant` — species grows in parks/gardens: 30% urban pass-through (default 5%)
- `preferredCodes` — specific BDL tree codes e.g. `['SO','PI']`; matching stands get +1 bonus

**Forest filter (L32455) — CRITICAL BUG FIXED v1.29.25:**
```js
if(coniferPref&&!broadleafPref&&f.c&&f.c!=='312')return false;
if(broadleafPref&&!coniferPref&&f.c&&f.c!=='311')return false;
// if BOTH set: accept any forest type (mixed habitat)
```
Old code used `&&` instead of exclusive checks — rejected ALL forests when both prefs set.

**Soft limits (no hard blocks as of v1.21.0):**
- Urban: 5% pass-through for forest/wetland species; 30% if `urbanTolerant`
- Farmland: wetland 2%, forest fallback 3%, openHab 10%, farmlandSpec exempt
- `urbanTolerant` true when terrain text contains: park, skwer, trawnik, ogród miejski, zieleń miejska, skraj miasta

**BDL bonus (v1.30.45):**
- Age threshold: 15 years minimum
- Intensity = `min(1, age/80) × 0.55`
- `typeBonus +1` when coniferPref+conifer tree or broadleafPref+broadleaf tree
- `codeBonus +1` when `b.cd` starts with one of `preferredCodes`
- `isSaprofit` (regex `/saprofit|pasożyt/i` on `m.substrate||m.ph`) → up to 3 points for old stands (FIXED v1.30.43)
- **BDL step sampling (v1.30.45):** `bdlStep=Math.max(1,Math.floor(bdlStore.length/pts.length))` — only processes every Nth entry so BDL adds ≈ same number of pts as genHeat (prevents 25× domination)
- **coniferPref+broadleafPref both-true fix (v1.30.45):** `if(coniferPref&&!broadleafPref&&bIsBroadleaf)return` — same fix as genHeat/scanner; mixed-forest species (both prefs=true) now accept all tree types for BDL bonus
- **rare filter moved post-BDL (v1.30.45):** `if(m.rare)pts=pts.filter((_,i)=>i%3!==0)` now runs AFTER BDL loop, so rare species get 33% fewer BDL pts too (not just 33% fewer genHeat pts)
- **BDL data count:** 9,799 raw entries → 3,832 unique after 0.02° grid dedup (`_gk(v)=Math.round(v*50)/50`). Not a bug.

**Geo-consistent radius:** `Math.max(12, Math.round(250 * Math.pow(2,z) / 94234))` — ~250m real-world blob at any zoom

## _buildBaseHeat (L52228) — derived opts

```js
const terrainStr=(m.terrain||[]).join(' ')+' '+(m.habitat||'');
const terrainForF=terrainStr.replace(/skraje?\s+las\w*/gi,'');  // strip forest-edge phrases before isF check
```

- `openHab`: HAB_PATTERNS.meadow OR `/nieużytk/i` AND !forest (after stripping "Skraje las*")
- `urbanSpec`: `/ogród|trawnik|przydroż|park|skwer|zieleń.*miejsk/i` AND !forest — uses meadow polygons + urban scatter
- `farmlandSpec`: `/kompost|pobocz|obornik|nawóz|pastw.*pole|pole.*pastw/i` AND !forest — uses farmland polygon EDGES only (`_farmlandEdgePts` uses `[...LANDUSE,..._dynLanduse]` since v1.30.41)
- `wetHab`: HAB_PATTERNS.wetland
- `waterEdge`: olsz/łęg/rzek/potok/strum/brzeg/jezioro  ← olch REMOVED v1.29.30 (matched "olcha/olchy" as host tree in wood-rotters)
- `_waterEdgePts` uses `[...LANDUSE,..._dynLanduse]` since v1.30.41 — dynamic Overpass water bodies (t:'water'|'river') feed riparian edge scoring
- CRITICAL: strip `terrainStr.replace(/skraje?\s+las\w*/gi,'')` before forest check (else "Skraje lasów" falsely triggers isF)
- `rare:true` species: `if(m.rare)pts=pts.filter((_,i)=>i%3!==0)` AFTER BDL loop (moved v1.30.45) — ~33% point reduction applies to both genHeat AND BDL pts
- Returns: `{pts, openHab, wetHab, waterEdge, farmlandSpec, urbanSpec, coniferPref, broadleafPref}`
  - **ALL six opts are returned** — destructure all of them in caller (missing any → ReferenceError)

## HAB_PATTERNS (defined near L41955 area in code, referenced everywhere)

```js
forest:/las|bór|iglaste|liściaste|śwkrk|sosnow|świerkow|bukow|dąbrow|mieszany|brzezin|olszyn/i  ← added brzezin|olszyn v1.29.30
meadow:/łąk|pastw|murawa|polana|ogród|skraj/i
wetland:/torf|bagien|podmok|bagno/i  ← wilgotn REMOVED v1.29.29 (matched "wilgotny mech" in forest habitats)
sandy:/piaszczyst|bór sosnow|sucha gleba/i
alpine:/górsk|górskie|piętro regla|subalpejsk|tatrzańsk|Tatry|Karpat|Beszczad|Sudety/i
park:/park|trawnik|ogród|skwer|zieleń miejsk/i
dung:/obornik|nawóz|kał|koprof|łąkach z wypasem|pastwisk/i
wood:/pniu|saprofit|drewno|kłod|pasożyt/i
```

**Data-area restriction:** Regions with no CLC polygon AND no LANDUSE data nearby are skipped.
