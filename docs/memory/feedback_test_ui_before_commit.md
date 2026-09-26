---
name: feedback-test-ui-before-commit
description: Always test UI changes in the browser before committing GrzyboMapa
metadata: 
  node_type: memory
  type: feedback
  originSessionId: abb55c48-8857-458c-a7e9-84bfb38e7568
  modified: 2026-09-19T23:13:54.977Z
---

Always open the GrzyboMapa preview in the browser and verify UI changes look correct before committing and pushing.

**Why:** User explicitly reminded about this rule in 2026-09-20 session after a bad commit broke the whole map (L.drawLocal crash halted all JS, GPS and draw buttons disappeared).
**How to apply:** After editing grzybomapa/index.html, use the in-app browser to navigate to the live site (or reload it), verify the feature looks correct AND check read_console_messages for errors BEFORE running git commit + push. Do not rely on logic review alone — test interactively.

**Specific lesson (2026-09-20):** When setting nested object properties like `L.drawLocal.draw.toolbar.actions.cancel.text`, first dump the actual object structure via `JSON.stringify(L.drawLocal.draw)` in the browser console, then write code. Wrong paths throw TypeError that silently kills all code after it.
