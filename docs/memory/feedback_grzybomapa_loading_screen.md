---
name: feedback-grzybomapa-loading-screen
description: "GrzyboMapa loading screen: always use background-size:cover, never a fixed percentage"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4e82ccda-048f-4b67-b928-3077b0387fba
  modified: 2026-09-21T23:01:10.261Z
---

`#map-loader` background image must use `center/cover no-repeat`, not `50% 44%/145% no-repeat` or any fixed percentage.

**Why:** A fixed width percentage (145%) fills landscape viewports but leaves black bars top/bottom on portrait phones because the image height doesn't cover the taller viewport.

**How to apply:** Both `#map-loader` and `[data-theme="light"] #map-loader` use the same `center/cover` value. No portrait-specific media query needed — cover handles all orientations automatically.
