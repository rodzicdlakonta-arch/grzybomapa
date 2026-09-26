---
name: feedback-backup-frequency
description: "Only create .html backups for big/drastic GrzyboMapa changes, not every commit"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 32d749b0-8d6e-480f-870f-25e4bfc347de
  modified: 2026-09-26T15:38:17.316Z
---

Skip making an `index.backup-vX.Y.Z.html` file for routine patches and data additions. Only back up before:
- Full UI redesigns
- Replacing large data arrays (FORESTS, MUSHROOMS, WYR_TREES)
- Any change that could be hard to roll back via git

Keep at most **2 backup files** in the repo at any time. If creating a new backup would make a third, delete the oldest one first.

**Why:** User said "git doesn't need so many backups only the last 2 or new important ones." Normal git history is sufficient for small changes.

**How to apply:** No backup file on tooltip fixes, version bumps, adding ≤1000 polygons. Do back up before replacing thousands of lines of data. After creating a backup, `ls *.backup*.html` and delete any beyond the 2 most recent.
