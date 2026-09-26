---
name: grzybomapa-data-cohesion
description: "GrzyboMapa data cohesion audit findings — 12 issues found 2026-09-22, fixes applied same session"
metadata: 
  node_type: memory
  type: feedback
  modified: 2026-09-22T05:44:10.075Z
  originSessionId: 8b4f41e2-4205-40f2-b2bf-5d93b28b53ce
---

## Confirmed issues and fixes applied v1.30.42+

### Issue 1 — maslak / maslak_zw duplicate (FIXED)
Both were Suillus luteus = Maślak zwyczajny with contradictory pH (3.5–5.0 vs 4.5–7.0), season, terrain.
**Fix:** Removed old `maslak` entry; kept `maslak_zw` as the authoritative entry with correct data from OCR guide.

### Issue 2 — _getScanOpts vs _buildBaseHeat divergent regexes (ALREADY FIXED before this session)
Was fixed in earlier sessions. `_getScanOpts` now has waterEdge, expanded broadleafPref/coniferPref matching heatmap. No wetHab wilgotn.

### Issue 3 — waterEdge absent from scanner scoring (ALREADY FIXED)
Present in _getScanOpts return value.

### Issue 4 — mountainSpec flat penalty in scanner (FIXED v1.30.43)
Scanner now uses geographic latFactor: `lat<50.5?1.6:lat>51.5?0.12:1.0` in both area and point scanner.
- Area scanner uses `clat=(b.getNorth()+b.getSouth())/2`
- Point scanner uses `lat` parameter directly

### Issue 5 — Season calendar static S{} map (PARTIALLY FIXED v1.30.44)
**"W sezonie" sidebar section** is a crowd-sourced placeholder — NOT derived from MUSHROOMS. It is DIFFERENT from the "W sezonie" filter in the species list. The sidebar S{} map stays static (curated list). Only fix applied: "Maślak żółty"→"Maślak zwyczajny" in the static names.
- v1.30.43 mistakenly made it MUSHROOMS-derived; reverted in v1.30.44
- Long-term: sidebar should show iNaturalist/GBIF observations filtered to current month + map area

### Issue 6 — mushroomPct species-agnostic (FIXED v1.30.43)
Added "poza sezonem" warning in fetchWeather when selectedMid's season doesn't include current month.

### Issue 7 — ph field overload / isSaprofit (FIXED v1.30.43)
`isSaprofit` now reads `m.substrate||m.ph`. Regex simplified to `/saprofit|pasożyt/i` (covers all variants).

### Issue 8 — rare:true not used in density (FIXED v1.30.43)
`if(m.rare)pts=pts.filter((_,i)=>i%3!==0)` in _buildBaseHeat — ~33% point reduction.

### Issue 9 — mountainSpec geographic in scanner (FIXED v1.30.43)
See Issue 4.

### Issue 10 — MUSHROOM_LOCAL hardcoded bbox (DEFERRED)
Bbox `p[0]>52.7&&p[0]<53.9&&p[1]>16.4&&p[1]<18.2` is still hardcoded. MUSHROOM_LOCAL is intentionally Krajna-specific, bbox covers that region. Low impact — left as-is.

### Issue 11 — BDL tree code regex divergence (FIXED v1.30.43)
Scanner blCodes updated to `/^(BRZ|BK|DB(?:B)?|GB|JW|LP|OL(?:S)?|JS|OS|TP|WB|AK|CZ|B(?=[.\s]|$))/i` in both area and point scanners.

### Issue 12 — Rain trigger window too narrow (FIXED v1.30.43)
Extended: `const rTrig=days.slice(0,7).reduce((a,b)=>a+(b||0),0)` — uses all 7 available past days.

## Science corrections
- Kauserud 2012 comment corrected: paper is about phenological timing, not soil temp at 6cm
- Rain trigger: extended to 14 days (literature says 26+ days for Boletus, 14 is pragmatic given 7 past days available from API)
- Soil temp curve shape kept (defensible for fruiting body formation, not mycelial growth — comment corrected)

## weights partial coverage — NOT AN ISSUE (scope clarification 2026-09-22)

The 39 REGIONS keys are planned future expansion across all Poland. Current map coverage is **Wielkopolska + Wyrzysk/Krajna only** (`wy`, `nak`, `sep` + nearby: `no`, `tu`, `ch`, `pi`, `dr`, `gr`, `br`). All 110 species already have those local keys populated. Missing keys for `ta`, `bi`, `be`, `sd`, `ro`, `bi` etc. are intentionally empty — fill them region-by-region as coverage expands southward/eastward. No fix needed now.

**Why:** All three divergences (scanner/heatmap logic, duplicate species, static calendar) were causing species to score differently across the two main analysis paths, and the duplicate species was generating contradictory predictions for the same mushroom.
