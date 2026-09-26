---
name: feedback-use-playwright
description: Use Playwright MCP browser for GrzyboMapa visual testing instead of relying on user to test on device
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4327bfd7-0ccf-48f3-96e2-329601ace473
  modified: 2026-09-21T22:16:55.556Z
---

Use Playwright (`mcp__playwright__*`) to test GrzyboMapa UI before asking the user to check on device.

**Why:** Much more efficient than repeated manual device testing. Playwright can resize viewport to landscape phone dimensions (e.g. 900×400), click elements, take screenshots, and evaluate JS to read computed state — far faster feedback loop.

**How to apply:**
- Start a local HTTP server first: `python -m http.server 8765 --directory "C:\Users\abibl\Desktop\Claude\grzybomapa"`
- Resize to landscape phone: `browser_resize(900, 400)`
- Navigate: `browser_navigate("http://localhost:8765/index.html")`
- Use `browser_click`, `browser_evaluate`, `browser_take_screenshot` to verify behavior
- Use `SendUserFile` to show screenshots inline rather than asking user to check
- Use `browser_evaluate` to read computed CSS values (e.g. `getBoundingClientRect()`) — catches discrepancies between authored CSS and actual rendered dimensions (e.g. discovered sidebar was 320px not 300px due to `min-width` override)
