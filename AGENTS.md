# Dumb Offline Agents — Project Rules for the Local Coding Agent

(File is still named AGENTS.md on disk — that exact filename is required
for OpenCode to auto-load it every session. Don't rename the actual file,
just think of it by this title.)

**Scope note:** This file is read only by OpenCode (local models via Ollama).
Claude Code reads a separate file, `CLAUDE.md`, and does not read this one.
Keep the two separate on purpose — this file's rules are written assuming a
weaker, offline, more error-prone model and are intentionally more
restrictive than what you'd want for Claude Code.

This file is read automatically by OpenCode at the start of every session.
It exists to keep the agent safe and predictable on a complex, multi-system
project. Edit the placeholders below to match your actual project.

## Project overview

**GrzyboMapa** — a free, science-backed Polish mushroom-foraging map. Live at
https://rodzicdlakonta-arch.github.io/grzybomapa/ (GitHub Pages, auto-deploys
from `master` on push). Two pillars: a live interactive map (observations,
heatmap, terrain scanner) and a knowledge base of 110 mushroom species.

**Structure — READ THIS FIRST:**
- `index.html` — the entire app. All HTML+CSS+JS inline. No build step.
  **This file is ~7.3MB.** It does not fit in this agent's context window in
  one read. Never attempt to read or rewrite the whole file in one go — see
  rule 16 below.
- `BDL_TREE_SPECIES_INFO.js` — static fallback dataset of ~9,800 real Polish
  forest stands (from Poland's BDL forestry database). Live BDL data is
  fetched at runtime and takes priority; this file is only the offline
  fallback.
- `generate_bdl_data.js` — the script that generated the file above. Only
  re-run this if explicitly asked to refresh the forest dataset.
- `docs/memory/` — detailed system documentation, written by a previous
  Claude Code session. Read the relevant file BEFORE touching that system.
  See the routing table below.

## Before touching a specific system, read the matching file first

Do not guess how a system works — these files already explain it, including
past bugs and why certain code looks the way it does. Read the relevant one
before editing, every time:

| If the task involves...                        | Read this first                                    |
|--------------------------------------------------|----------------------------------------------------|
| General app structure, data objects, UI, layout   | `docs/memory/grzybomapa_tech.md`                    |
| The heatmap (density/scoring)                     | `docs/memory/grzybomapa_heatmap.md`                 |
| The terrain scanner / best-spot algorithm         | `docs/memory/grzybomapa_scanner.md`                 |
| Species data, terrain classification, OCR data    | `docs/memory/grzybomapa_species.md`                 |
| Forest polygons / how forest data was built       | `docs/memory/grzybomapa_forests_how_made.md`        |
| The forest rendering layer (canvas/tiles)         | `docs/memory/grzybomapa_forest_layer.md`            |
| Dynamic terrain (Overpass-fetched water/farmland)  | `docs/memory/grzybomapa_dynamic_terrain.md`         |
| Mobile / landscape layout                         | `docs/memory/feedback_grzybomapa_landscape_mobile.md`, `docs/memory/feedback_grzybomapa_mobile_nav.md` |
| Loading screen                                    | `docs/memory/feedback_grzybomapa_loading_screen.md` |
| The "W sezonie" (in season) sidebar                | `docs/memory/feedback_grzybomapa_w_sezonie.md`      |
| F2 debug overlay                                  | `docs/memory/feedback_grzybomapa_f2_debug.md`       |
| Info panel / list closing behavior                | `docs/memory/feedback_grzybomapa_ip_close_list.md`  |
| Data cohesion / cross-system consistency issues    | `docs/memory/feedback_grzybomapa_data_cohesion.md`  |
| Species OCR/data-entry checks                     | `docs/memory/feedback_grzybomapa_ocr_check.md`, `docs/memory/feedback_mushroom_checklist.md` |
| Pixel 9 / pointer-coarse mobile bug history        | `docs/memory/feedback_pixel9_pointer_coarse.md`     |
| Anything about v1.29.x bugs specifically           | `docs/memory/project_grzybomapa_bugs_v1_29.md`      |
| Anything about v1.30.x bugs specifically           | `docs/memory/project_grzybomapa_bugs_v1_30.md`      |
| The navigation redesign (v1.30.0)                  | `docs/memory/project_grzybomapa_redesign.md`, `docs/memory/project_grzybomapa_redesign_options.md` |
| Full project history / version log / roadmap       | `docs/memory/project_grzybomapa.md`                 |

If a task touches a system not listed here, say so and ask rather than
guessing — the file might exist under a name that isn't obvious from the
task description.

## Recurring known bug pattern (read before touching scoring/matching logic)

Multiple past versions (v1.29.25, v1.29.28, v1.30.45) had the same root bug:
code treated `coniferPref && broadleafPref` both being true as "must match
BOTH" when it should mean "accept all forest types." If you're touching any
scoring, filtering, or habitat-matching logic, check for this exact pattern
before assuming new code is correct.

## Hard safety rules (never break these)

1. **Never delete files.** If a file genuinely needs to go, ask first and
   explain why, then wait for explicit confirmation.
2. **Never modify or overwrite `.env`, `.env.*`, or any file containing API
   keys, secrets, or credentials.** Treat these as read-only unless I
   explicitly ask you to change one, and never print their contents back to me
   in full.
3. **Never touch `.git` internals directly** (no manual edits inside `.git/`).
   Use normal git commands only, and only when asked.
4. **Never run destructive terminal commands** (`rm -rf`, `git reset --hard`,
   `git push --force`, database drops/migrations that delete data, etc.)
   without stating exactly what the command does and getting a yes first.
5. **Never install, remove, or upgrade dependencies** (npm/pip packages,
   major library versions) without asking first and explaining why it's
   needed.
6. **Don't refactor or "clean up" code I didn't ask you to touch.** Stay
   scoped to the specific task. If you notice something else that looks
   wrong, mention it, don't fix it unprompted.

## Working style for multi-file / multi-system changes

7. **Before editing multiple files, list every file you intend to touch and
   what you'll change in each one.** Wait for a go-ahead on anything beyond a
   single small file.
8. **Keep systems isolated.** Don't let changes to the maps system leak into
   unrelated systems (auth, payments, etc.) unless the task explicitly
   requires it. If a change needs to cross system boundaries, flag that
   clearly before doing it.
9. **Prefer small, incremental changes over big rewrites.** If a task looks
   like it needs a large rewrite, break it into steps and confirm each step.
10. **Always show a diff-style summary of what you changed and why**, even
    for small edits.
11. **If you're not sure a change is safe, stop and ask** rather than
    guessing. Guessing wrong on a multi-system project is expensive.

## Testing / verification

12. After any code change, if there's an existing test suite or build step,
    run it and report the result honestly — including if it fails.
13. If no tests exist for the area you changed, say so, and briefly describe
    how I could manually verify the change (e.g. "reload the maps page and
    check the marker renders at the right coordinates").

## Style

14. Match the existing code style, naming conventions, and formatting already
    in the project. Don't introduce a different style "because it's better."
15. Add comments only where the logic isn't obvious — don't over-comment
    simple code.

## Rule 16: Editing index.html safely (critical for this project)

`index.html` is too large for this agent to hold in context at once. To
avoid corrupting or silently losing sections of it:

- Never ask for or attempt a full-file rewrite. Only ever target a specific
  function, section, or line range.
- Before editing, search for the specific function/section name first
  (grep-style search) rather than opening the whole file.
- After every real change: bump the version number in the logo subtitle
  (format `vX.Y.Z`), matching the pattern in `docs/memory/project_grzybomapa.md`'s
  version history.
- Only make a backup (`index.backup-vX.Y.Z.html`) before large structural
  changes: full UI redesigns, replacing entire data arrays (FORESTS, MUSHROOMS,
  WYR_TREES), or any change that would be hard to undo via git. Skip backups for
  routine patches, version bumps, and small additions. Keep at most 2 backup
  files — delete the oldest if a third would be created.
- Test the change by actually opening `index.html` in a browser preview
  before committing — do not assume a change works just because it "looks
  right" in the diff.
- After testing successfully: `git add index.html && git commit && git push`
  — GitHub Pages auto-deploys from `master`, so a bad push goes live
  immediately. Do not push without the version bump and a working preview.

## When in doubt
Ask a short, specific question rather than assuming. A wrong assumption on a
system with many moving parts (maps, auth, data, etc.) is worse than a
30-second clarifying question.
