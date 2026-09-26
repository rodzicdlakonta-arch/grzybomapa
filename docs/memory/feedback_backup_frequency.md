---
name: feedback-backup-frequency
description: "Only create .html backups for big/drastic GrzyboMapa changes, not every commit"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 32d749b0-8d6e-480f-870f-25e4bfc347de
  modified: 2026-09-22T03:19:36.228Z
---

Skip making an `index.backup-vX.Y.Z.html` file for routine patches and data additions. Only back up before:
- Full UI redesigns
- Replacing large data arrays (FORESTS, MUSHROOMS, WYR_TREES)
- Any change that could be hard to roll back via git

**Why:** User said "we don't need to back up that much, only on big updates or drastic changes." Normal git history is sufficient rollback for small changes.

**How to apply:** No backup file on tooltip fixes, version bumps, adding ≤1000 polygons. Do back up before replacing thousands of lines of data.
