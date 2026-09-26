---
name: grzybomapa-species
description: "GrzyboMapa species data rules — OCR guide, terrain classification, known fixes, terrain keywords"
metadata: 
  node_type: memory
  type: project
  originSessionId: 0f66b839-ccd9-465e-87a1-2bddb4202eec
  modified: 2026-09-22T06:11:00.999Z
---

## Species data rules

**Always verify against OCR guide before adding/changing species:**
- `C:\Users\abibl\Desktop\Claude\grzyby_guide_ocr_full.txt` (710KB, 726 pages)
- Polish name conventions: "Kolczakówka"=Hydnellum (inedible); "Kolczak"=Hydnum (edible); "Sarniak"=Sarcodon; "Wieruszka"=Entoloma
- `cat:'rare'` is NOT valid — use `rare:true` for rarity flag; edibility = `edible`/`poisonous`/`deadly`/`inedible`/`hallucinogenic`
- Protected species: `protected:true` + `cat:'inedible'`

**Known species fixes (v1.21.1–v1.21.2):**
- gaska_ziel: cat edible→poisonous (rabdomioliza)
- lejkowka: rare:true, habitat fixed (no olive trees in Poland; 1 known site)
- maczuznik + zasloniak_fio: cat rare→inedible
- kolpaczek: renamed to Łysak wspaniały; hallucinogenic removed; danger→gymnopilin
- smardz_pol + smardz_st: kwas helvellowy→gyromitrin
- sarniak: fixed to Sarcodon imbricatus (was mislabeled as Hydnum repandum)
- kolczakowka: restored as Hydnum repandum "Kolczak obłączasty"

## Terrain classification rules (v1.29.18)

**"Skraje lasów" problem:** "las" in "lasów" falsely triggers isF. Fix: strip `skraje?\s+las\w*` before forest test. Applied in both `_getScanOpts` and `_buildBaseHeat`.

**isF regex (after strip):** `/las|bór|iglaste|liściaste|sosnow|świerkow|bukow|dąbrow|mieszany/i`
**openHab:** `/łąk|pastw|murawa|polana|nieużytk/i` AND !isF
**farmlandSpec:** `/ogród|kompost|trawnik|pobocz|przydroż|obornik|nawóz/i` AND !isF
**wetHab:** `/torf|bagien|podmok|bagno/i` (wilgotn removed v1.29.29)

## v1.29.34 terrain audit — 11 fixes (broadleafPref/openHab mismatches)

**Root cause A — birch terrain labels don't trigger broadleafPref:**
`'Brzeziny'`, `'Lasy z brzozą'`, `'Lasy z osiką'`, `'Topólki'` match NONE of `/liściaste|dąbrow|bukow|grabowe|grąd/`. Every birch/aspen/poplar-only species needs `'Lasy liściaste'` added.

**Root cause B — `'Buczyny'` doesn't match `/bukow/`:**
Beech forests use noun-suffix derivation (buk→buczyna) not adjectival (buk→bukowy). Any species with only `'Buczyny'` terrain needs `'Lasy liściaste'` added.

**`'Skraje lasów'` is safe — do NOT remove it:**
"las" in "lasów" would normally trigger isForest, but the `terrainForF` strip regex `/skraje?\s+las\w*/gi` removes it before the forest check (added v1.29.18). `openHab` uses `terrainForF`, so 'Skraje lasów' never blocks openHab. Keep it — it's an accurate habitat label.

**Fixes applied v1.29.34–35:**
- kozlarz_cz: `+Lasy liściaste` (obligate aspen/poplar, broadleaf)
- kozlarz_b: `+Lasy liściaste` (obligate birch)
- muszomor_cz: `+Lasy liściaste` (co-equal birch host)
- krowiak: `+Lasy liściaste` (co-equal birch host)
- zasloniak_krus: `+Lasy liściaste` (obligate birch)
- golabek_zolt: `Olszyny→removed` (false waterEdge; birch-only), `+Lasy liściaste`
- zasloniak_fio: `+Lasy liściaste` (grows under beech; Buczyny≠/bukow/)
- maslak_mod: `+Lasy iglaste` (larch = conifer; none of the trigger words was present)

**Species corrected in v1.29.18:**
- czubajka kania: removed 'Dąbrowy' (grows at oak edge, not inside)
- smardz jadalny: removed 'Lasy liściaste' (orchard/garden primary) → terrain: `['Sady','Ogródki']`
- strzępiak niebieskozielony: removed 'Lasy mieszane' (park/urban specialist) → terrain: `['Parki','Trawniki','Zieleń miejska','Ogrody']`
- czubajka czerwieniejąca: removed 'Lasy iglaste' (garden/compost primary) → terrain: `['Ogrody','Kompostowniki','Parki']`

**Species that CORRECTLY stay as forest despite having Parki/Ogrody:**
- sromotnik bezwstydny: `['Lasy liściaste','Lasy mieszane','Parki','Ogrody']` — ectomycorrhizal, forest primary
- koprówka atramantowa: `['Lasy liściaste','Lasy iglaste','Parki','Ogrody']` — tree-base, forest primary
- krowiak podwinięty: `['Lasy liściaste','Parki','Ogrody','Trawniki']` — ectomycorrhizal, needs trees

## Species added v1.29.38–39 (to avoid re-adding)

**v1.29.38 (keys: gaska_sos, lysiczka_fal):**
- gaska_sos = Tricholoma vaccinum (Gąska sosnowa) — inedible, pine forest, reddish fibrous scales
- lysiczka_fal = Psilocybe cyanescens (Łysiczka falista) — hallucinogenic/poisonous, wood-chip mulch, urban parks

**v1.29.39 (keys: maslak_zw, mleczaj_wel, borowik_sz, lakowka, lejkowka_mgla, golabek_sin, kozlarz_pom):**
- maslak_zw = Suillus luteus (Maślak zwyczajny) — edible, pine, slimy cap+ring
- mleczaj_wel = Lactarius torminosus (Mleczaj wełnianka) — inedible, birch, woolly cap edge, white burning latex
- borowik_sz = Rubroboletus satanas (Borowik szatański) — poisonous, rare, calcareous oak/beech
- lakowka = Laccaria laccata (Łakówka jesienna) — edible, all forests, pink fading cap Jun–Nov
- lejkowka_mgla = Lepista nebularis (Lejkówka mglistozielonkawa) — inedible, grey cap with bump, sweet marzipan smell, fairy rings
- golabek_sin = Russula cyanoxantha (Gołąbek siniejący) — edible, elastic gills (unique among Russula), variable purple/green/grey cap
- kozlarz_pom = Leccinum versipelle (Koźlarz pomarańczowożółty) — edible, orange cap+black scabers, flesh blackens, birch/heathland

**Before adding new species — check these already-present keys:**
podgrzybek_zaj (= Xerocomus subtomentosus "Podgrzybek zajączek"), mleczaj_deb (= Lactarius quietus), mleczaj_obrz (= Lactarius volemus?), mleczaj_piep (= Lactarius piperatus)

**Terrain keywords NOT covered by any regex (edge cases to watch):**
- "Sady" (orchards) — only matters if species has ONLY Sady without other garden keywords. Covered by 'ogród' if also has Ogródki. Smardz stożkowaty has `['Lasy iglaste','Lasy mieszane','Sady']` → correctly forest (genuine conifer forest species).
- "Wąwozy" (ravines), "Aleje" (avenues), "Łęgi" (riparian) — no special flag; absorbed by forest or open as appropriate
- "Szuwary" (reedbeds) — should be wetHab but doesn't appear in current species data

**Pasture/open ground rule (user confirmed):** We have no data on whether open ground has animals. Treat all open/fallow ground as possibly pasture — the `pa` flag exists but LANDUSE pasture data is sparse. For farmlandSpec species with dung keywords (obornik), open ground `pa` also counts.

## Known terrain/data fixes v1.30.43–45

- **maslak_ziarn** (Suillus granulatus): terrain `'Lasy górskie z sosną'`→`'Lasy iglaste z sosną'` — removed 'górskie' which falsely triggered mountainSpec=true (v1.30.45). Suillus granulatus is NOT a mountain specialist; grows in lowland pine forests 0–1400m.
- **kurka_zim** (Craterellus tubaeformis): terrain includes `'Bory bagienne'` + habitat `'podmokłych'` → both trigger wetHab=true. Scanner scores wetland cells (h.we) only; if no wetland polygons in scan area, score=0 even though species grows in boggy CONIFER forest. Pending fix: wetHab+forest species should fall back to forest scoring when no wetland cells found.

## BDL tree codes

SO/PI=sosna/pine, ŚW/SW=świerk/spruce, JD=jodła/fir, MD=modrzew/larch (conifers)
BK=buk/beech, DB=dąb/oak, GB=grab/hornbeam, JW=jawor/maple, LP=lipa/linden, OL=olcha/alder, JS=jesion/ash, CZ=czereśnia/cherry (broadleaf)
