---
name: feedback-update-version-before-commit
description: "GrzyboMapa: always bump and sync version text in all copies of index.html before committing"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4e82ccda-048f-4b67-b928-3077b0387fba
  modified: 2026-09-21T22:55:57.607Z
---

Before committing any GrzyboMapa change, update the version string in **both places inside index.html** AND across all copies:

There are TWO version strings in `grzybomapa/index.html`:
1. **~L432** — desktop sidebar `<p>`: `v1.30.x`
2. **~L43282** — mobile top bar `#mob-logo span`: `v1.30.x` (drifted to v1.30.13 in the past — use replace_all)

Use `replace_all: true` in Edit to hit both at once, then confirm with Grep that none are left behind.

**Why:** The two strings are far apart and easy to miss. The mobile one drifted unnoticed for several versions.

**How to apply:** Before committing, grep for the OLD version string across all `.html` files under `grzybomapa/`. Replace all hits with the new version. The backup file (`index.backup-v1.29.77.html`) is intentionally frozen — skip it.
