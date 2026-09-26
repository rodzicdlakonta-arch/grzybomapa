---
name: grzybomapa-scanner
description: "GrzyboMapa scanner — habPts structure, _getScanOpts, scoring, best-spot, F2 debug, URBAN_DATA critical notes. LINE ANCHORS included."
metadata: 
  node_type: memory
  type: project
  originSessionId: 0f86b839-ccd9-465e-87a1-2bddb4202eec
  modified: 2026-09-26T15:09:40.094Z
---

## Line anchors (index.html)

| Symbol | Line |
|--------|------|
| `pipTest(point, polygon)` | **52551** |
| `polyCentroid(pts)` | **52562** |
| `_getScanOpts(m)` | **52566** |
| `buildTerrainHTML(terrain)` | **52611** |
| `_doAreaAnalysis(layer)` | **52717** |
| `analyzeArea(layer)` | **52789** |
| `showAreaPanel(results,terrain)` | **52811** |
| `_doPointAnalysis(lat,lng,radiusM)` | **53084** |
| `analyzePoint(lat,lng,radiusM)` | **53180** |
| `_drawDebugEdges()` | **53425** |
| `_showDebugOverlay()` | **53459** |
| `_refreshDebugOverlay()` | **53524** |
| `_showBestSpot(mid)` | **52420** |
| `showInfoPanel(mid)` | **52474** |

## Forest tooltip (v1.30.48)

`_showForestTip(ll)` — extracted from `map.on('mousemove')`, takes a `LatLng` object. Called from both `mousemove` (desktop hover) and `click` (mobile tap, when `!bdlActive`). Auto-hides `_fHoverTip` after 2500ms on mobile tap.

`L.Layer.prototype.bindTooltip` is patched on touch devices (before L43870): adds `click` handler to every polygon/polyline that opens the tooltip at tap latlng and closes after 2500ms. Skips layers that also have `_popup` bound (to avoid conflict with iNat markers).

## Scanner architecture (v1.29.8+)

- 🔍 toggle button → expands to ▭ (rectangle) and ✏ (polygon) draw tools
- Green dashed ring + green dot on dblclick; single-click reopens panel, dblclick destroys
- Radius slider (200–3000m) for pin scan, hidden for polygon
- Shows results immediately from cached data, fetches Overpass in background, auto-updates if panel still open
- F2 debug overlay: snapshot after any scan (CLC/wyrStands/BDL/landuse counts) + heatmap opts + best-spot table

## URBAN_DATA — CRITICAL

Entries are raw polygon arrays `[[lat,lng],...]`, NOT `{p:[...]}` objects.

```js
// CORRECT filter:
const nearU=URBAN_DATA.filter(u=>u.some(p=>p[0]>s0-.01&&...));
// CORRECT pip test:
ur:nearU.some(u=>pipTest([la,ln],u))   // u, not u.p
// WRONG (crashes):
u.p.some(...)   // u.p is undefined!
```

Helper `_inUrban(la,ln)` at L43546 uses `URBAN_DATA.some(up=>pipTest([la,ln],up))`.

## habPts entry structure

```js
// nearL = [...LANDUSE,..._dynLanduse] filtered by bbox (since v1.30.39)
// _dynLanduse has live Overpass entries: t:'wetland'|'meadow'|'farmland'|'pasture'|'water'|'river'
habPts.push({la, ln,
  f: nearF.filter(f=>pipTest([la,ln],f.p)),  // CLC forest polygons
  we: nearL.some(l=>l.t==='wetland'&&pipTest([la,ln],l.p)),
  me: nearL.some(l=>l.t==='meadow'&&pipTest([la,ln],l.p)),
  pa: nearL.some(l=>l.t==='pasture'&&pipTest([la,ln],l.p)),
  fa: nearL.some(l=>l.t==='farmland'&&pipTest([la,ln],l.p)),
  ur: nearU.some(u=>pipTest([la,ln],u))   // NOTE: u not u.p
});
```

`getTerrainFeatures(drawnPoly)` also uses `[...LANDUSE,..._dynLanduse]` for its sample-point PIP pass.

## _getScanOpts(m) — L52566

```js
function _getScanOpts(m){
  const t=(m.terrain||[]).join(' ')+' '+(m.habitat||'');
  const tForF=t.replace(/skraje?\s+las\w*/gi,'');  // strip forest-edge phrases
  const isF=/las|bór|iglaste|liściaste|sosnow|świerkow|bukow|dąbrow|mieszany/i.test(tForF);
  const wetHab=/torf|bagien|podmok|wilgotn|bagno/i.test(t);
  const openHab=/łąk|pastw|murawa|polana|nieużytk/i.test(t)&&!isF;
  const urbanSpec=/ogród|trawnik|przydroż|park|skwer/i.test(t)&&!isF;
  const farmlandSpec=/kompost|pobocz|obornik|nawóz/i.test(t)&&!isF;
  const coniferPref=/iglaste|bór|sosnow|świerkow/i.test(t);
  const broadleafPref=/liściaste|dąbrow|bukow|grabowe|grąd/i.test(t);
  ...
  return{openHab,wetHab,farmlandSpec,urbanSpec,coniferPref,broadleafPref,...,isF};  // isF added v1.30.46
}
```

## Scoring — CRITICAL BUG FIXED v1.29.26

Used in `_doAreaAnalysis` (L52717) and `_doPointAnalysis` (L53084):

Priority order (first match wins):
1. `wetHab` → count `h.we` cells
2. `urbanSpec` → count `h.ur||h.me||h.pa||(h.f.length===0&&!h.we&&!h.fa&&!h.ur)` cells
3. `farmlandSpec` → count `h.fa||h.pa` cells
4. `openHab` → count `h.me||h.pa||(h.f.length===0&&!h.we&&!h.fa&&!h.ur)` cells
5. else (forest) → **FIXED** formula:

```js
h.f.some(f=>!f.c||(coniferPref&&!broadleafPref?f.c==='312':broadleafPref&&!coniferPref?f.c==='311':true))
// untyped polygon always passes; both prefs = accept any typed forest; only one pref = filter by type
```

Old code used `&&` — rejected ALL typed forests when both coniferPref+broadleafPref were set.

Score modifier: `mountainSpec→latFactor (lat<50.5?1.6:lat>51.5?0.12:1.0), acidicSoil×1.1, alkalineSoil×0.85` (FIXED v1.30.43: was flat 0.2 regardless of scan location; area scanner uses clat=(b.getNorth()+b.getSouth())/2, point scanner uses lat parameter)

## Best-spot algorithm (_showBestSpot L41061) — FIXED v1.29.25

Same coniferPref+broadleafPref fix applied (search `_showBestSpot` at L52420):
```js
else ok=h.f.some(f=>{if(opts.coniferPref&&!opts.broadleafPref)return f.c==='312';if(opts.broadleafPref&&!opts.coniferPref)return f.c==='311';return true;});
```

Bucket algorithm: heatmap points → ~500m cells, sum weights, score by `weight × exp(-km/20)`.

## F2 debug overlay

- Open/close: keydown F2
- `_drawDebugEdges()` — draws polygon outlines on map (uses `[...LANDUSE,..._dynLanduse]` since v1.30.41):
  - Forest species (not wetHab/openHab/farmland/urban): **green CLC outlines** (dark=conifer, medium=broadleaf, teal=mixed)
  - waterEdge: blue LANDUSE+dynamic water outlines; rivers use `lu.t==='river'` check (NOT `lu.desc?.includes('river')` — absent on dynamic entries)
  - farmlandSpec: orange LANDUSE+dynamic farmland outlines
  - urbanSpec: pink URBAN_DATA outlines
- `_refreshDebugOverlay()` L42030 — called after every scan or `_buildBaseHeat`; redraws if overlay is open

## Terrain display (buildTerrainHTML L41244)

Priority per cell (mutually exclusive): `ur > we > conifer(312) > broadleaf(311) > mixed > me > pa > fa > open`
Tags shown when ≥8% of cells (open: ≥15%). Shows `Ugory / polany` if nothing else qualifies.

## Grid density formulas

- Area scan: `GN=Math.max(10,Math.min(55,Math.round(Math.max(n0-s0,e0-w0)*1300)))`
- Point scan: `GP=Math.max(10,Math.min(50,Math.round(r2*2/150)))`
