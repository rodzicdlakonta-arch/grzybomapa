---
name: feedback-pixel9-pointer-coarse
description: pointer:coarse media feature unreliable on Pixel 9 Pro XL — use max-width:1366px instead to discriminate phones from desktops in landscape MQ
metadata:
  node_type: memory
  type: feedback
  modified: 2026-09-21T21:57:26.511Z
  originSessionId: 996fcca4-52e4-4e3f-ae23-32147383b3ce
---

Never use `pointer:coarse` to discriminate mobile phones from desktop laptops in GrzyboMapa's landscape media query.

**Why:** Pixel 9 Pro XL running Chrome Android does NOT match `pointer:coarse` reliably. The device likely reports a fine pointer due to its AI pen/stylus accessory or a Chrome version quirk. Adding `and (pointer:coarse)` to `@media(max-height:700px) and (orientation:landscape)` caused the entire landscape MQ to stop applying on the device — sidebar reverted to flex layout (map narrowed), BDL unreachable, all landscape fixes invisible. Discovered in v1.30.11, reverted in v1.30.13.

**How to apply:** Use `max-width:1366px` instead:
```css
@media (max-height:700px) and (orientation:landscape) and (max-width:1366px) { … }
```
And in JS `isLargeLS()`:
```js
const isLargeLS=()=>window.innerWidth>window.innerHeight&&window.innerHeight<700&&window.innerWidth<=1366;
```
Rationale: Pixel 9 Pro XL in landscape ≈ 1040px CSS width (matches); most desktops/laptops ≥ 1440px (excluded). Both CSS MQ and JS function must stay in sync.
