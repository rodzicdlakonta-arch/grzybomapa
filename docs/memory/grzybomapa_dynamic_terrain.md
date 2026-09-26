---
name: grzybomapa-dynamic-terrain
description: "_dynLanduse array, ovDynWaterLayer, BDL API-primary pattern, loadDetailTerrain structure — load when touching terrain layers or scanner/heatmap data pipeline"
metadata: 
  node_type: memory
  type: project
  originSessionId: 418394b3-28cc-4db6-abd7-9b370871030b
  modified: 2026-09-22T05:39:06.203Z
---

## _dynLanduse[] — runtime terrain array

`const _dynLanduse=[];` declared after `_bdlLive` (near line ~53727).

**Structure:** `{t: 'wetland'|'meadow'|'farmland'|'pasture'|'water'|'river', p: [[lat,lng],...]}`
Same shape as LANDUSE entries. Populated by `loadDetailTerrain()` Overpass handlers.

**Clear-before-repopulate pattern** (per type, splice-from-end):
```js
for(let i=_dynLanduse.length-1;i>=0;i--)if(_dynLanduse[i].t==='meadow')_dynLanduse.splice(i,1);
```

**Consumers (all use `[...LANDUSE,..._dynLanduse]`):**
- `getTerrainFeatures(drawnPoly)` — sample-point PIP for scanner display
- `nearL` in `_doAreaAnalysis` — bbox-filtered for habPts grid
- `nearLP` in `_doPointAnalysis` — bbox-filtered for habPts grid
- `_waterEdgePts(reg,w,n)` — waterEdge heatmap edge points
- `_farmlandEdgePts(reg,w,n)` — farmlandSpec heatmap edge points
- `_drawDebugEdges()` — F2 debug outlines for water+farmland

**Notes:**
- `t:'river'` is filtered OUT by `nearL`/`nearLP` (`.t!=='river'`), but included by `_waterEdgePts`
- `t:'water'` passes into `nearL` but `habPts` has no `wa:` field — no effect on scanner grid scores
- `_dynLanduse` grows as user pans; never cleared globally — only per-type on Overpass refetch

## loadDetailTerrain() structure

Triggered by `moveend zoomend` (600ms debounce) AND by swamp/meadow/crop toggles (`showDynTerrain=true`).

```
loadDetailTerrain()
  if(zoom<8) return              ← early-exit for all
  compute b2=getBounds(), bb=bbox string
  ─── ALWAYS runs (no toggle gate): ───────────────────────────────
  Overpass: natural=water + reservoir + waterway=river|canal
    → clearLayers(ovDynWaterLayer)
    → clear t:'water'|'river' from _dynLanduse
    → push t:'water'|'river' to _dynLanduse
    → render L.polygon/L.polyline to ovDynWaterLayer
  ─── Gated by (showOv||showDynTerrain): ──────────────────────────
  if(!showOv&&!showDynTerrain) return
  if(area too large) return      ← >1.6°×2.4° bbox
  Overpass: landuse=forest|natural=wood → ovLayer + wyrForests[]
  if(zoom>=9):
    Overpass: wetlands → ovWetLayer + _dynLanduse t:'wetland'
    Overpass: meadows/grassland → ovMeadowLayer + _dynLanduse t:'meadow'
    Overpass: farmland → ovCropLayer + _dynLanduse t:'farmland'
    Overpass: pasture → ovPastureLayer + _dynLanduse t:'pasture'
```

**Layer wiring:**
```js
waterBodyLayer.addLayer(ovWetLayer);      // preloadWyrzysk IIFE wetlands
waterBodyLayer.addLayer(ovDynWaterLayer); // all dynamic water
meadowLayer.addLayer(ovMeadowLayer);
cropLayer.addLayer(ovCropLayer);
pastureLayer.addLayer(ovPastureLayer);
```

## BDL data pipeline (v1.30.32+)

**File:** `grzybomapa/BDL_TREE_SPECIES_INFO.js` — 9,799 forest stands, 52.5–53.85°N, 16.5–18.25°E, 0.01° grid.
**Builder:** `grzybomapa/generate_bdl_data.js` — run `node generate_bdl_data.js` from `grzybomapa/` folder (Node 18+, ~3-5 min).
**Format:** `[[lat, lng, 'SO', age_years], ...]` loaded into `bdlStore` at startup.

**API-primary / file-fallback pattern:**
```js
const bdlStore=[];     // pre-populated from BDL_TREE_SPECIES_INFO at startup
const _bdlLive=new Set(); // grid keys already live-fetched this session

async function _autoBDL(samplePts){
  const fresh=samplePts.filter(([la,lo])=>!_bdlLive.has(`${gk(la)},${gk(lo)}`));
  if(!fresh.length)return;
  // fetch API for each fresh key → override bdlStore entry on success
  // file data preserved if API fails
}
```

`_bdlLive` tracks live-fetched keys — prevents file data from blocking API calls. Pre-loaded file data = fallback only when API returns nothing.

**bdlB proportional scoring (both area and point analysis):**
```js
const conCodes=/^(SO|PI|ŚW|SW|JD|MD)/i, blCodes=/^(BRZ|BK|DB(?:B)?|GB|JW|LP|OL(?:S)?|JS|OS|TP|WB|AK|CZ|B(?=[.\s]|$))/i;
// UPDATED v1.30.43: added DBB, OLS, CZ variants to match heatmap
const matched=inAreaBDL.filter(bd=>bd.cd&&(coniferPref?conCodes.test(bd.cd):broadleafPref?blCodes.test(bd.cd):true)).length;
if(matched)bdlB=Math.min(0.35, matched/inAreaBDL.length*0.5);
// Max 0.35 bonus (was binary 0.15); proportional to match fraction
```

## waterEdge buffer (v1.30.42)

`_waterEdgePts` jitter: `±0.005°lat ≈ ±250m, ±0.007°lng ≈ ±234m` at 53°N.
Was `±0.003°/0.004°` (≈±167m) — too narrow for Polish riparian mushroom zones (realistic: 200–350m).
