---
name: feedback-mushroom-checklist
description: "Required checklist when adding new mushroom species to GrzyboMapa — fields, scanner check, habitat check"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 75267ebb-051f-4f3f-b407-6f25d684a1c9
  modified: 2026-09-20T16:32:05.397Z
---

When adding new mushroom species, ALWAYS go through this checklist before committing:

## 1. Verify against OCR guide
Grep `C:\Users\abibl\Desktop\Claude\grzyby_guide_ocr_full.txt` for the Latin name first. Confirm Polish name, habitat, season, edibility, and lookalikes match the guide.

## 2. Required fields — match every existing species exactly

All species must have:
- `name`, `latin`, `cat`, `rare`, `wiki`
- `season` (array of month numbers)
- `terrain` (array of Polish habitat labels)
- `soil`, `ph`, `elev`
- `habitat` (prose paragraph)
- `desc` (visual description — include size in cm, color, key features in CAPS)
- `danger` (even for edible: note any prep warnings)
- `lookalikes` (with toxicity labels)
- `weights` (regional weight object)

**Edible species ALSO need:**
- `q` (1–5 taste rating, placed before `name:`) — used for pip display + default sort
- `edib` (culinary note — how to prepare, what to watch for)

Inedible/poisonous/deadly species: NO `q` or `edib`. Only `danger`.

**Why:** `q` drives the "Smak" pip rating in the info panel and default sort order. Missing it leaves a blank pip row. Found missing on maslak_zw, lakowka, golabek_sin, kozlarz_pom (fixed v1.29.40).

## 3. Check terrain for scanner impact

After writing terrain[], check each label against these patterns:
- `isForest`: `/las|bór|iglaste|liściaste|sosnow|świerkow|bukow|dąbrow|mieszany|brzezin|olszyn/i` — does the species hit this?
- `openHab`: `/łąk|pastw|murawa|polana|nieużytk/i` AND !isForest — open-ground species?
- `urbanSpec`: `/ogród|trawnik|przydroż|park|skwer|zieleń.*miejsk/i` AND !isForest — urban specialist?
- `wetHab`: `/torf|bagien|podmok|bagno/i` — wetland species?
- `waterEdge`: `/olsz|łęg|rzek|potok|strum|brzeg|jezioro/i` — riparian species?
- `preferredCodes`: SO/PI (sosnow), ŚW/SW (świerkow/świerczyn), JD (jodłow), MD (modrzew), BK (bukow/buczyn), DB (dąbrow/dębowy), GB (grabowy/grąd), OL (olsz/olch), BRZ (brzezin/brzoz), JS (jesion), LP (lipow)

**If a new species needs a habitat type not covered by any pattern above** → update the relevant regex in both `_buildBaseHeat` (L~40883) and `_getScanOpts` (L~41202).

## 4. Add to all 5 data structures

Every new species key must appear in:
1. `MUSHROOMS` — main data object
2. `MUSH_ICON_CFG` — shape + colors
3. `MUSHROOM_LOCAL` — regional presence `{s:'p'|'f'|'r'|'n', n:'note'}`
4. `SPECIES_DIFF` — 1–4 ID difficulty
5. `SPECIES_RARITY` — 1–5 rarity in Poland

Missing any one leaves the species partially broken (no icon, no rarity pip, etc.).

## 5. Check for duplicates first

Common "obvious" species that are ALREADY in the DB (Latin → key):
- Xerocomus subtomentosus "Podgrzybek zajączek" → `podgrzybek_zaj`
- Lactarius quietus → `mleczaj_deb`
- Lactarius volemus → `mleczaj_obrz`
- Lactarius piperatus → `mleczaj_piep`
- Suillus granulatus → `maslak_ziarn`
- Suillus variegatus → `maslak_pst`
- Armillaria mellea → `opienka`
- Armillaria ostoyae → `opienka_ciem`
- Amanita muscaria → `muszomor_cz`

**How to apply:** Before every mushroom addition session, grep `latin:'` in index.html to get the current full Latin-name list, then compare against intended additions.
