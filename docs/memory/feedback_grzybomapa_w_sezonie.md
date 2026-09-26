---
name: grzybomapa-w-sezonie-design
description: "W sezonie sidebar section is crowd-sourced observations, NOT derived from MUSHROOMS — differs from the W sezonie filter"
metadata: 
  node_type: memory
  type: feedback
  modified: 2026-09-22T05:39:16.214Z
  originSessionId: 8b4f41e2-4205-40f2-b2bf-5d93b28b53ce
---

The sidebar "W sezonie" section (season calendar chips in the left panel) is designed to show **crowd-sourced mushroom observations** for the current month/area. It is a PLACEHOLDER showing a static curated list until the feature is implemented.

**This is different from** the "W sezonie" filter badge in the species list, which uses `MUSHROOMS[mid].season[]` to filter by current month.

**Why:** User explicitly corrected a dynamic MUSHROOMS-derived calendar that was added in v1.30.43. It was reverted in v1.30.44. "the in seazon thing from the website was suppose to be crowed sorced by the way its diffrent from the filter in seazon"

**How to apply:** Never replace the static S{} map with MUSHROOMS-derived data. The sidebar calendar is a crowd-sourced placeholder — future implementation should fetch iNaturalist/GBIF filtered to current month + visible map bbox.
