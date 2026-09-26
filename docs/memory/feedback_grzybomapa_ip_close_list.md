---
name: feedback-grzybomapa-ip-close-list
description: "GrzyboMapa portrait: closing info panel (#pcls) must not open the list when scanner panel (#ap) is open"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4e82ccda-048f-4b67-b928-3077b0387fba
  modified: 2026-09-21T23:01:17.739Z
---

In `#pcls` onclick (the X button on the info panel), the mob list auto-open block must check whether `#ap` is open first.

**Why:** If the user scans an area (#ap open with percentages) then taps a mushroom (#ip opens), closing #ip should return to the map — not forcibly open the species list on top of the scanner results.

**How to apply:**
```javascript
const _apOpen = document.getElementById('ap')?.classList.contains('open');
// only activate mob-list tab and open sidebar when scanner is NOT showing
if (!_apOpen) document.getElementById('mob-list').classList.add('active');
if (window.innerWidth <= 767 && !_apOpen) {
  setTimeout(() => { window._openMobList?.() || ... }, 80);
}
```
