---
name: grzybomapa-forests-how-made
description: "How GrzyboMapa FORESTS array was built and how to add more polygons — Overpass query, dedup, insertion process"
metadata: 
  node_type: memory
  type: project
  originSessionId: 32d749b0-8d6e-480f-870f-25e4bfc347de
  modified: 2026-09-22T03:22:34.260Z
---

## Origin of FORESTS data

The FORESTS array in `index.html` (line ~638) is built from **OpenStreetMap** forest polygons fetched via Overpass API. It replaced the original 344 CLC polygons in v1.30.21.

- **Original fetch:** 52.5–53.8°N, 16.5–18.2°E → 10,510 polygons (from `forests_osm.txt`, 1.89MB)
- **Gap fill v1.30.29:** 53.05–53.17°N, 17.25–17.45°E → +201 polygons (Bakowo/Osiek)
- **Gap fill v1.30.30:** 53.08–53.27°N, 17.15–17.65°E → +598 polygons (Glesno/Kraczki/Ruda)
- Total as of v1.30.31: ~11,300+ polygons

The original large fetch had systematic gaps (server truncation likely). Sub-area re-fetches consistently find missing polygons even inside the original bbox. When user reports visible gaps, re-fetch that specific sub-area.

## Overpass query

```
[out:json][timeout:60];
(
  way["landuse"="forest"](SOUTH,WEST,NORTH,EAST);
  way["natural"="wood"](SOUTH,WEST,NORTH,EAST);
);
out geom;
```

**Important:** Use `overpass.openstreetmap.fr` — `overpass-api.de` returns 406 Not Acceptable in PowerShell.

```powershell
$query = '[out:json][timeout:60];(way["landuse"="forest"](S,W,N,E);way["natural"="wood"](S,W,N,E););out geom;'
$encoded = [System.Uri]::EscapeDataString($query)
$url = "https://overpass.openstreetmap.fr/api/interpreter?data=$encoded"
$wc = New-Object System.Net.WebClient
$wc.Headers.Add("User-Agent","GrzyboMapa/1.0")
$wc.DownloadFile($url, "C:\...\osm_gap.json")
```

## Processing: JSON → FORESTS entries

```powershell
$json = Get-Content "osm_gap.json" -Raw | ConvertFrom-Json

# Build fingerprint set from existing index.html FORESTS first-points
$html = Get-Content "index.html" -Raw
$existingPts = [System.Collections.Generic.HashSet[string]]::new()
$matches = [regex]::Matches($html, '\[\[(\d+\.\d+),(\d+\.\d+)\]')
foreach ($m in $matches) { $existingPts.Add("$($m.Groups[1].Value),$($m.Groups[2].Value)") | Out-Null }

# Generate new entries (skip duplicates by first-point fingerprint)
$newEntries = @()
foreach ($el in $json.elements) {
    if ($el.geometry.Count -lt 3) { continue }
    $p0 = $el.geometry[0]
    $fp = "$([math]::Round($p0.lat,5)),$([math]::Round($p0.lon,5))"
    if ($existingPts.Contains($fp)) { continue }
    $lt = $el.tags.leaf_type
    $c = if ($lt -eq 'broadleaved') { '311' } elseif ($lt -eq 'needleleaved') { '312' } else { '313' }
    $pStr = ($el.geometry | ForEach-Object { "[$([math]::Round($_.lat,5)),$([math]::Round($_.lon,5))]" }) -join ','
    $newEntries += "  {p:[$pStr],c:'$c'},"
}
```

## CORINE type from OSM `leaf_type` tag

| `leaf_type` | CORINE | Polish name |
|-------------|--------|-------------|
| `broadleaved` | `'311'` | Las liściasty |
| `needleleaved` | `'312'` | Bór iglasty |
| *(missing)* | `'313'` | Las mieszany |

**97.4% of Polish OSM forest polygons have no `leaf_type` tag → all end up as '313'.** This is a known OSM data gap for Poland. Real species classification comes from `getTreeType()` using other tags (species, adr_les, PGL LP operator) when the detailed layer is loaded.

## Insertion into index.html

Insert new entries just before the named-forest comment marker:

```powershell
$marker = "  // ── Named forest compartments — PGL Lasy Państwowe / OSM geometry ──"
$html = $html.Replace($marker, $insertBlock.TrimEnd() + "`n" + $marker)
$html | Out-File "index.html" -Encoding utf8 -NoNewline
```

The comment `// ── Named forest compartments …` is the reliable insertion anchor. Named forests (entries with `n:` but no `c:`) follow after it and must NOT get `c:` values (they're for named-area lookup only, not polygon rendering).

## Named forests (no `c:`)

These are entries like `{n:"Wilcze Doły",p:[[...],...]}` — polygon outlines for named areas. They're picked up by `_namedF` / `_namedForestAt()` and used to show the forest name in the hover tooltip. They are NOT rendered as colored polygons. Do NOT add a `c:` field to them.

## When user reports gaps

1. Ask for approximate location or read from screenshot coordinates
2. Fetch a tight bbox around the gap (0.1–0.2° margin) with the PowerShell process above
3. Run dedup against existing first-points
4. Insert before the named-forest marker
5. Bump version, commit, push — no backup needed for data additions
