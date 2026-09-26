---
name: feedback-grzybomapa-ocr-check
description: Always verify mushroom species data against the OCR guide before adding or changing any GrzyboMapa species entry
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8ae9d567-6eb8-498c-96ad-ff8f657586ca
  modified: 2026-09-19T17:38:39.744Z
---

Always check the OCR guide before editing or adding species data in GrzyboMapa.

**Why:** Two entries (sarniak/kolczakowka) had wrong Latin names — app said both were Hydnum repandum but sarniak is actually Sarcodon imbricatus (completely different genus). The OCR guide immediately revealed this. Also: species that share the same Latin name in the app might have a data error on one of them, not be a true duplicate — always verify before removing.

**How to apply:**
- Use `grep -n "Latin name" C:\Users\abibl\Desktop\Claude\grzyby_guide_ocr_full.txt` to find the guide entry
- Use `sed -n 'Xp'` with surrounding lines to read the full description
- Check: Polish name, habitat, season, edibility classification, any protection status
- When two app entries share a Latin name: confirm they're the same species before removing; one may just have a wrong Latin name
- Polish name conventions: "Kolczakówka"=Hydnellum (inedible), "Kolczak"=Hydnum, "Sarniak"=Sarcodon, "Wieruszka"=Entoloma — genus names differ by common name family

See [[project-grzybomapa]] for the full guide file path and known-good species fixes.
