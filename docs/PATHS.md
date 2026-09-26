# GrzyboMapa — File Paths Reference

> One file to find everything. Update this when adding files.

---

## Project files (this folder)

| File | Purpose |
|------|---------|
| `index.html` | **The entire app** — all HTML+CSS+JS inline, ~7.4MB, no build step |
| `BDL_TREE_SPECIES_INFO.js` | Static fallback: ~9,800 real Polish forest stands (BDL database) |
| `generate_bdl_data.js` | Script that generates BDL_TREE_SPECIES_INFO.js from raw BDL data |
| `LoadingScreen.png` | Loading screen background image |
| `AGENTS.md` | Rules for OpenCode (local Ollama agent); NOT read by Claude Code |
| `.claude/launch.json` | Dev server config for Claude Code browser preview |
| `.gitignore` | Git ignore rules |

## Docs folder (`docs/`)

| Path | Purpose |
|------|---------|
| `docs/PATHS.md` | **This file** |
| `docs/memory/` | Copies of all GrzyboMapa memory files (read-only reference; originals auto-loaded by Claude from `.claude/projects/memory/`) |
| `docs/superpowers/plans/` | Superpowers agent plans |
| `docs/superpowers/specs/` | Superpowers agent specs |

## Live site & git

- **Live URL:** https://rodzicdlakonta-arch.github.io/grzybomapa/
- **Git remote:** https://github.com/rodzicdlakonta-arch/grzybomapa.git (branch: master)
- GitHub Pages auto-deploys on push to master

## Claude memory files (auto-loaded by Claude Code)

Location: `C:\Users\abibl\.claude\projects\C--Users-abibl-Desktop-Claude\memory\`

> Note: `docs/memory/` has synced copies of the GrzyboMapa-specific ones.
> Claude reads from the path above — the docs/memory copies are for your reference only.

### GrzyboMapa — architecture

| Memory file | What's in it |
|-------------|-------------|
| `project_grzybomapa.md` | Version, file path, git remote, pending work, full version history |
| `grzybomapa_tech.md` | Data objects (MUSHROOMS, FORESTS, LANDUSE), layout, UI component map |
| `grzybomapa_heatmap.md` | genHeat opts, BDL bonus, HAB_PATTERNS, _buildBaseHeat |
| `grzybomapa_scanner.md` | habPts, _getScanOpts, scoring, best-spot, URBAN_DATA, F2 debug |
| `grzybomapa_species.md` | Species data rules, terrain classification, known fixes, OCR guide workflow |
| `grzybomapa_dynamic_terrain.md` | _dynLanduse[], ovDynWaterLayer, BDL API pipeline, loadDetailTerrain |
| `grzybomapa_forest_layer.md` | Canvas trees, _FSI spatial index, wyrForests, WYR_TREES, tooltips |
| `grzybomapa_forests_how_made.md` | How forest polygons were built (Overpass query, dedup process) |

### GrzyboMapa — project & history

| Memory file | What's in it |
|-------------|-------------|
| `project_grzybomapa_redesign.md` | AllTrails nav arch decisions (v1.30.0) |
| `project_grzybomapa_bugs_v1_29.md` | Historical v1.29 bug log (all fixed) |
| `project_grzybomapa_bugs_v1_30.md` | Historical v1.30.0–v1.30.16 bug log (all fixed) |

### GrzyboMapa — workflow & lessons

| Memory file | What's in it |
|-------------|-------------|
| `feedback_mushroom_checklist.md` | Required fields when adding species (q+edib, 5 data structures) |
| `feedback_grzybomapa_ocr_check.md` | Always verify against OCR guide before editing species |
| `feedback_update_version_before_commit.md` | Bump BOTH logo version strings before every commit |
| `feedback_test_ui_before_commit.md` | Open browser preview and verify before committing |
| `feedback_use_playwright.md` | http.server 8765, resize 900×400 landscape, Playwright commands |
| `feedback_backup_frequency.md` | Skip .backup.html for routine patches; only on big structural changes |
| `feedback_grzybomapa_f2_debug.md` | Extend _lastDebugData + F2 panel whenever adding features |
| `feedback_grzybomapa_ip_close_list.md` | #pcls close must check #ap open state first |
| `feedback_grzybomapa_loading_screen.md` | Always center/cover, never fixed %; covers all orientations |
| `feedback_grzybomapa_landscape_mobile.md` | 7 mobile/landscape layout lessons |
| `feedback_grzybomapa_mobile_nav.md` | DOM timing, CSS cascade, teleport pattern for mobile nav |
| `feedback_pixel9_pointer_coarse.md` | Use max-width:1366px not pointer:coarse for phone targeting |
| `feedback_grzybomapa_data_cohesion.md` | 12 data quality issues found and fixed 2026-09-22 |
| `feedback_grzybomapa_w_sezonie.md` | "W sezonie" sidebar = crowd-sourced placeholder, not MUSHROOMS.season[] |

### GrzyboMapa — data sources

| Memory file | What's in it |
|-------------|-------------|
| `reference_grzyby_guide.md` | Ewald Gerhardt guide; OCR at `Desktop/Claude/grzyby_guide_ocr_full.txt` |

### Other projects (not GrzyboMapa)

| Memory file | What's in it |
|-------------|-------------|
| `project_repair_business_site.md` | Dziadek Komodor Astro site — live at dziadekkomodor.pl |
| `project_szkolawchmurze.md` | School subjects, exam types, deadline Mar 2027 |
| `project_prezentacje.md` | Gamma presentations tracker per subject |
| `reference_headroom.md` | Headroom compression tool (installed, MCP registered) |
| `reference_active_skills.md` | Superpowers + frontend-design plugin skills reference |
| `feedback_commit_after_changes.md` | Always commit so user only has to push |
| `feedback_nontechnical_instructions.md` | GUI/plain-language over git CLI/jargon |
| `feedback_context_window.md` | Use /compact early, scope Globs to src/ |
| `feedback_save_mem.md` | User types "save mem" to trigger memory save before clearing |
| `feedback_reread_last_message.md` | After session summary, re-read last user message before acting |

---

## Data files outside the project folder

| Path | What it is |
|------|-----------|
| `C:\Users\abibl\Desktop\Claude\grzyby_guide_ocr_full.txt` | Full OCR of Ewald Gerhardt's mushroom guide (726 pages, 710KB) |
| `C:\Users\abibl\Desktop\Claude\grzybomapa_Backup\` | Backup copy of grzybomapa folder (older snapshots) |
