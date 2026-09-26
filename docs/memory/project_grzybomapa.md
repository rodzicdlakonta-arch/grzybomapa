---
name: project-grzybomapa
description: GrzyboMapa — science-backed Polish mushroom map; 110 species; v1.30.0; live at rodzicdlakonta-arch.github.io/grzybomapa
metadata: 
  node_type: memory
  type: project
  originSessionId: 0f66b839-ccd9-465e-87a1-2bddb4202eec
  modified: 2026-09-26T15:09:32.967Z
---

## Core facts

- **File:** `C:\Users\abibl\Desktop\Claude\grzybomapa\index.html` — single file, all CSS+HTML+JS inline, no build step
- **Git remote:** `https://github.com/rodzicdlakonta-arch/grzybomapa.git` (master branch)
- **Live URL:** https://rodzicdlakonta-arch.github.io/grzybomapa/
- **110 species** as of v1.29.39
- **Current version: v1.30.48** (2026-09-22)

## After every change

1. Bump version in `<span style="letter-spacing:.04em">vX.Y.Z</span>` in the logo subtitle
2. `git add index.html && git commit && git push` — GitHub Pages auto-deploys
3. Test in browser preview before committing (see [[feedback-test-ui-before-commit]])

## Vision

Free, science-backed mushroom map for Polish foragers. Two pillars: live map (observations, heatmap, scanner) + knowledge base (101 species, descriptions, safety, photos). Long-term: phone app + domain GrzyboMapa.pl (not yet bought).

## Detailed sub-files (load when relevant)

- [[grzybomapa-tech]] — data objects (MUSHROOMS, FORESTS, LANDUSE, URBAN_DATA, bdlStore), layout, filter/sort, UI components
- [[grzybomapa-heatmap]] — genHeat opts, soft limits, BDL bonus, HAB_PATTERNS, _buildBaseHeat logic
- [[grzybomapa-scanner]] — habPts structure, _getScanOpts, scoring, buildTerrainHTML, best-spot algorithm, URBAN_DATA critical notes
- [[grzybomapa-species]] — species data rules, OCR guide, terrain classification, known fixes, terrain keywords

## Pending / planned

**HIGH priority:**
1. More species (101 → 150+) — expand database, check OCR guide at `Desktop/Claude/grzyby_guide_ocr_full.txt`
2. Entoloma sinuatum (Wieruszka różowawa) — add properly; was removed due to wrong Latin name
3. Mushroom Radar — "what's findable near me now": season + weather + location → readiness score

**LOWER priority:**
- User-submitted sightings
- Multiple photos per species
- Phone app (after web solid)
- Domain GrzyboMapa.pl
- Crowd-sourced "W sezonie" sidebar — intended to show iNaturalist/GBIF observations for current month + visible map area (currently static placeholder)

**Expansion (when adding new regions):**
- Weights keys in MUSHROOMS: only `wy/nak/sep` + nearby Wielkopolska keys are populated now; fill missing region keys (ta/bi/be/sd/ro/etc.) as map coverage expands to those areas

**Known pending bugs:**
- Heatmap jitter during smooth scroll-zoom (complex, deferred)
- Still missing forest polygons in some areas (original Overpass fetch was truncated); fetch more on demand when user reports gaps

**UPCOMING: Data accuracy sweep**
- Scanner/heatmap forest type — currently 97% of FORESTS are c:'313' (mixed); use `_wyrForestAt()` + wyrForests[] in scanner scoring so pine species score correctly in bory sosnowe
- Fresh WYR_TREES from BDL — replace static 8k-point Wyrzysk-only array with live BDL data for any visited area (bigger architecture change)
- Weather predictions scientific accuracy — verify mushroom-growth weather rules in code
- Water/wetland polygons — check LANDUSE completeness

## Version history (milestones)

- v1.21.0: Soft limits, BDL preferredCodes
- v1.22.0: Filter/sort, GPS button
- v1.26.0: Pip ratings, hallucinogenic species, lightbox, mobile nav fix
- v1.27.0: Multi-select heatmap, season arrow, "W sezonie" badge, "Łatwe ID" sort
- v1.28.0: Loading screen, satellite overlay, lightbox drag-to-pan, copy icons
- v1.29.8: Scanner redesign fully working (draw tools, terrain display, F2 debug, best-spot)
- v1.29.17: Urban detection in scanner, soil/pH/elevation modifiers, farmlandSpec fixes
- v1.29.18: Species terrain audit (4 fixes), Skraje-lasów regex fix, best-spot bucket algorithm
- v1.29.19: farmlandSpec scores ur+pa only (crop field interiors not suitable — tractor damage)
- v1.29.20: F2 overlay shows heatmap opts (scoring type, all flags ✓/✗, pH/elev) + best-spot process (top buckets table with ★ winner, km, score)
- v1.29.21: farmlandSpec heatmap uses _farmlandEdgePts (boundary ±150m, like waterEdge) not interior _sp; scanner re-adds h.fa to farmlandSpec scoring
- v1.29.22: F2 live-updates on heatmap load (_buildBaseHeat calls _refreshDebugOverlay); labels "skraj pól" / "skraj wód"
- v1.29.23: F2 draws edge polygon outlines on map — blue for waterEdge, orange for farmlandSpec; clears on F2 close; _dbgEdgeLayer
- v1.29.24: split farmlandSpec→urbanSpec (ogród/trawnik/park/przydroż) vs farmlandSpec (kompost/pobocz/obornik/nawóz); F2 draws pink URBAN_DATA outlines for urbanSpec
- v1.29.25: fix urbanSpec missing from destructuring (ReferenceError → can't open mushrooms); fix coniferPref&&broadleafPref both true filtered all forests (fix: only filter when exactly one set); same fix in _showBestSpot; F2 now draws green CLC forest outlines for forest species
- v1.29.26: same coniferPref+broadleafPref AND-impossible bug fixed in _doAreaAnalysis (L41369) and _doPointAnalysis (L41679); memory sub-files updated with line number anchors
- v1.29.27: F2 edges wiped by scan (area/point scan replaced _lastDebugData, losing heatOpts) — fixed with spread-merge; _getScanOpts pH parsed wrong (string char → regex like _buildBaseHeat); urbanSpec/farmlandSpec/openHab pattern gaps synced
- v1.29.28: _getScanOpts mountainSpec m.elev[0] always-false → regex parse; unified mountain regex across both functions; habLabel shows "Lasy iglaste i liściaste" when both prefs set
- v1.29.29: remove `wilgotn` from HAB_PATTERNS.wetland + _getScanOpts wetHab regex (matched "wilgotny mech" → Borowik and other forest species wrongly showed "Mokradła i torfowiska"); fix _drawDebugEdges inRange to use polyCentroid (was any-vertex → huge river corridor polygons spanned whole map as dashed outlines)
- v1.29.30: species data audit fixes — remove `olch` from waterEdge regex (false waterEdge for wood-rotters listing alder as host); fix zasloniak_krus/golabek_zolt terrain (Lasy liściaste→Brzeziny); smardz terrain (Sady/Ogródki→Łęgi/Lasy liściaste/Sady); huba terrain (Olszyny→Buczyny); fix smardz_pol/smardz_sto edib (gyromitrin→hemolizyny — morchella doesn't have gyromitrin); HAB_PATTERNS.forest += brzezin|olszyn; scanner Halucynogenne filter added; river corridor F2 debug draws one bank as polyline
- v1.29.31: category filters AND logic (every() not some()) — Jadalne+W sezonie now shows only BOTH; fix hallucinogenic filter to check m.hallucinogenic flag (not m.cat) since those species have cat:'poisonous'
- v1.29.32–33: (applied during session — waterEdge in _buildBaseHeat/_getScanOpts for wetland species; species count display in sidebar; activeHabs filter no longer checks m.soil)
- v1.29.34: full 101-species terrain audit — 11 broadleafPref/coniferPref/openHab fixes (birch labels, Buczyny, Skraje lasów — see [[grzybomapa-species]] for root causes)
- v1.29.35: revert 3 bad Skraje-lasów removals (czubajka, purchawka_olb, twardzioszek); Skraje lasów is valid — strip regex handles it
- v1.29.36: preferredCodes block rewritten — independent if blocks + Set dedup; added MD/BRZ/JS/LP codes; fixed świerczyn/buczyn/brzezin/olch patterns
- v1.29.37: coniferTerrain/broadleafTerrain (_buildBaseHeat) + coniferPref/broadleafPref (_getScanOpts) expanded to cover specific tree-name terrain labels (Buczyny, Brzeziny, Świerczyny, etc.)
- v1.29.38: add Gąska sosnowa (Tricholoma vaccinum, gaska_sos) + Łysiczka falista (Psilocybe cyanescens, lysiczka_fal); total 103 species
- v1.29.39: add 7 species (103→110): maslak_zw (Suillus luteus), mleczaj_wel (Lactarius torminosus), borowik_sz (Rubroboletus satanas), lakowka (Laccaria laccata), lejkowka_mgla (Lepista nebularis), golabek_sin (Russula cyanoxantha), kozlarz_pom (Leccinum versipelle)
- v1.29.40: fix missing q+edib fields on 4 new edible species (maslak_zw, lakowka, golabek_sin, kozlarz_pom)
- v1.29.41: mobile CSS — layers+weather start collapsed, info panel minimize button + drag handle, compass higher, mob-ico smaller, landscape media query, 🎛️ Filtry icon
- v1.29.42: layers panel toggle via mob-filters; toggle-to-close on mob-list/mob-filters; leaflet-rotate touchRotate; compass click→reset bearing; compass SVG rotates with map; Q/E desktop rotation; toast tap-to-dismiss; removed duplicate mob-nav script
- v1.29.51–v1.29.55: mobile UX pass — rarity sort uses combined SPECIES_RARITY+MUSHROOM_LOCAL score (same as pips); info panel X calls clearHeat() then reopens list on mobile; layers scroll fixed; map rotation removed (didn't work); lightbox pinch-zoom + swipe-to-minimize info panel; 🍄 tab handles minimized panel; filters start collapsed on mobile by default (user preference remembered in localStorage fc-filters)
- v1.29.56–v1.29.60: mobile UX pass 2 — list scroll reset on panel open/close; live search filters list in real-time; auto GPS locate with blue dot on load; BDL named forests fixed (layers 0-5 in GetFeatureInfo); landscape layout for large phones (max-height:500px); landscape 🗺️/📋 toggle button (shows map-only or info mode); pcls (X) resets sidebar split height
- v1.29.61: fix landscape scanner draw, #ap position, ls-toggle visibility — see [[grzybomapa-scanner]] for touch draw lesson
- v1.29.62–v1.29.77: (various fixes — see [[project-grzybomapa-bugs-v1-29]])
- v1.30.0 (2026-09-21): Full navigation redesign — desktop sidebar tabs, mobile portrait top bar+chips+pill+filter modal, landscape pull-tab handle; pushed
- v1.30.3–v1.30.10: Mobile bug fixes (chips, arrows, scroll, portrait layout)
- v1.30.11: Sidebar border fix (translateX -302px), global `#mob-handle{display:none}`, `#mc{position:fixed}` for BDL (partial), sidebar handle 28×48px
- v1.30.12: Info panel minimize handlers moved outside `if(innerWidth<=767)` — portrait minimize now works after LS→portrait rotation
- v1.30.13: `pointer:coarse` → `max-width:1366px` in landscape MQ + `isLargeLS()`; `#mc-body{padding-bottom:80px}`; BDL CONFIRMED FIXED on Pixel 9 Pro XL
- v1.30.14: LS panel fixes — `translateX(-320px)` hides sidebar fully; `#sb-tabs{display:flex!important}` shows Grzyby/Filtry tabs in landscape; removed override rules that nullified tab switching; arrow fixed (`›` closed, `‹` open); handle `left:320px` when open (matches actual min-width:320px sidebar)
- v1.30.15: Auto-expand "Filtry i sortowanie" collapsible when switching to Filtry tab (landscape) or opening filter modal (portrait)
- v1.30.16: `body.sbt-filtry #sb-top{max-height:100vh!important;flex:1}` — filters panel fills full sidebar height in landscape Filtry tab
- v1.30.20: LS scroll preserved on #pcls close (check landscape condition before scrollTop=0); multi-select badge `#ip-sel-badge` shows "2×" etc. when selectedMids.size≥2
- v1.30.21: FORESTS replaced — 344 CLC polygons → 10,510 OSM polygons for Wyrzysk/Wielkopolska north (52.5-53.8°N, 16.5-18.2°E); viewport lazy rendering via `_fBboxes`+`_fObjs`+`_updateForestLayer()` on `moveend zoomend`
- v1.30.21–v1.30.27: Canvas tree emoji grid (`_TreeGL` L.GridLayer), spatial index `_FSI`, ray-cast PIP `_pip()`, canvas clip path per polygon type, type-specific emoji (311=🌳 312=🌲 313=checkerboard), hover tooltip with WYR_TREES species lookup
- v1.30.28: Tooltip fix — WYR_TREES name[3] (sometimes forest stand name) filtered via genus regex; named forest lookup `_namedForestAt()`; bdlStore integration; `_BDL_SP` species code map
- v1.30.29: +201 OSM forest polygons for Bakowo/Osiek nad Notecią (53.05-53.17°N, 17.25-17.45°E)
- v1.30.30: +598 OSM forest polygons for Glesno/Kraczki/Ruda area (53.08-53.27°N, 17.15-17.65°E); tooltip species diversity fix (WYR_TREES name[] with genus filter, not _FTN[type] which collapsed all to 100% pine)
- v1.30.31: `_wyrForestAt()` — PIP in live `wyrForests[]` (from detailed OSM layer) with lazy bbox cache; tooltip priority: wyrForests > bdlStore > WYR_TREES; `_FOREST_LABEL` map; BDL cd splitting for "SO.DB" multi-species codes
- v1.30.32–v1.30.37: BDL data pipeline — `BDL_TREE_SPECIES_INFO.js` (9,799 real stands, 52.5-53.85°N, 16.5-18.25°E) generated by `generate_bdl_data.js`; `_bdlLive` Set tracks live-fetched keys so API always runs (file = fallback only); heatmap viewport BDL fetch wired into selectMushroom Promise.all; bdlB scoring changed to proportional Math.min(0.35, matched/total*0.5) instead of binary
- v1.30.38: terrain layer improvements — wetland/meadow/farmland/pasture tooltips in Polish; meadow build chunked (300/frame) to avoid freeze; `showDynTerrain` flag lets Overpass load when terrain toggles active (not only BDL); `ovWetLayer` kept in waterBodyLayer (not swampLayer) so Wyrzysk startup wetlands stay visible
- v1.30.39: `_dynLanduse[]` array — Overpass-fetched terrain (wetland/meadow/farmland/pasture) now stored in parallel to LANDUSE; scanner `nearL`/`nearLP`/`getTerrainFeatures` spread `_dynLanduse` so dynamic terrain feeds habPts scoring
- v1.30.40: `ovDynWaterLayer` — fetches natural=water lakes, reservoirs, waterway=river|canal from Overpass on every moveend (zoom≥8); fills gaps in static LANDUSE water data; always visible regardless of terrain toggle state
- v1.30.41: dynamic water/farmland feeds heatmap scoring — `_dynLanduse` now gets t:'water'|'river' entries from water fetch; `_waterEdgePts`/`_farmlandEdgePts` use `[...LANDUSE,..._dynLanduse]`; `_drawDebugEdges` updated to show dynamic water+farmland outlines; river type check changed from `lu.desc?.includes('river')` to `lu.t==='river'`
- v1.30.42: waterEdge heatmap buffer widened from ±167m to ±250m (0.003→0.005 lat, 0.004→0.007 lng) — better matches Polish lowland riparian mushroom zones
- v1.30.43: data cohesion fixes — duplicate maslak removed (kept maslak_zw), mountainSpec geographic latFactor in both scanners, BDL blCodes updated (DBB/OLS/CZ), rare:true density reduction in heatmap, isSaprofit reads m.substrate||m.ph, rain window extended to 7 days, season warning in weather widget, MUSHROOM_LOCAL key maslak→maslak_zw
- v1.30.44: season calendar reverted to static curated list ("W sezonie" sidebar = crowd-sourced placeholder, NOT MUSHROOMS.season[]); "Maślak żółty"→"Maślak zwyczajny" in static S{} map
- v1.30.45: heatmap aggression fix — BDL loop now uses bdlStep=floor(bdlStore/genHeatPts) to cap BDL contribution; fixed coniferPref&&broadleafPref both-true bug in BDL loop (same fix as genHeat/scanner: both prefs = accept all tree types); moved rare filter to after BDL loop so rare species have lower BDL density too; maslak_ziarn terrain 'Lasy górskie z sosną'→'Lasy iglaste z sosną' (removed false mountainSpec trigger)
- v1.30.46: wetHab scanner fallback — _getScanOpts now returns isF; _doAreaAnalysis and _doPointAnalysis fall back to forest scoring at 60% when wetHab species finds no wetland cells AND area has water bodies (rivers/lakes); fixes kurka_zim, kozlarz_b, golabek_wym, zasloniak_rub, golabek_zolt, zasloniak_krus scoring 0 in forested scan areas
- v1.30.47: habitat audit — maczuznik terrain +Lasy iglaste (Cordyceps militaris parasitizes insects in any forest type, was broadleafPref-only)
- v1.30.48: right-click Google Maps nav + mobile tap tooltips + long-press menu — right-click popup adds "Nawiguj w Google Maps" link (google.com/maps?q=lat,lng); mobile long-press on map (600ms) fires same contextmenu popup; forest tooltip extracted to _showForestTip(ll), reused on mobile tap (auto-hides 2.5s); L.Layer.prototype.bindTooltip patched on touch devices so all polygon tooltips (water, wetland, meadow, farmland, protected) respond to tap
