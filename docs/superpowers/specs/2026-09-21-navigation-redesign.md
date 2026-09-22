# GrzyboMapa v2.0 — Navigation Redesign Spec

Date: 2026-09-21  
Version: draft  
Backup: `index.backup-v1.29.77.html`

---

## Problem Summary

Three separate layout failures, all rooted in the same cause: filters and the species list share the same scrollable container with no clear separation.

| Viewport | Root Failure |
|---|---|
| Desktop | Sidebar is one undivided column — scroll past 350px of filters to reach the list; resize handle is not discoverable |
| Mobile portrait | Bottom sheet 80vh: logo+search+filters eat 535px, leaving 106px for 10 000px of species content |
| Mobile landscape | Sidebar hidden by default, only reachable via an unlabeled floating 📋 button |

---

## Design Decisions

### Reference pattern
AllTrails mobile (nearest comparable app): slim fixed top bar + horizontal filter chip row + full-screen map + single floating `Lista / Mapa` toggle pill. Species list takes the full screen when active. No bottom tab bar for primary navigation.

NN/g confirms: nonmodal bottom sheet + tab bar = best for map apps. Gesture-only drawers have poor discoverability. Three tabs that all open the same bottom sheet (current situation) is the worst of both worlds.

---

## Section 1 — Desktop Sidebar Tabs

### What changes
`#sb-top` gains a tab row between the logo and the content area.

```
┌─────────────────────────┐
│  GrzyboMapa  🍄         │  ← #logo (unchanged)
├─────────────────────────┤
│  [ Grzyby ] [ Filtry ]  │  ← NEW #sb-tabs (tab row)
├─────────────────────────┤
│  Szukaj...       🔍     │  ← #sw search (visible in Grzyby tab only)
│  ─────────────────────  │
│  species list (#ml)     │  ← fills remaining height, Grzyby tab only
│   …                     │
│   …                     │
└─────────────────────────┘

When "Filtry" tab active:
┌─────────────────────────┐
│  GrzyboMapa  🍄         │
├─────────────────────────┤
│  [ Grzyby ] [ Filtry ]  │
├─────────────────────────┤
│  KATEGORIA              │  ← #filters-panel fills remaining height
│  SEZON                  │
│  SIEDLISKO              │
│  SORTUJ                 │
│  …                      │
└─────────────────────────┘
```

### Rules
- Default active tab: `Grzyby`
- Active tab indicator: 2px gold underline (`var(--gold)`)
- `Grzyby` active → show `#sw` + `#ml`; hide `#filters-panel` + `#season-bar`
- `Filtry` active → show `#filters-panel` + `#season-bar`; hide `#sw` + `#ml`
- `#ml-resize` drag handle → hidden (tabs replace it; it cluttered the UX)
- `#season-bar` → moved into the info panel header only (removed from sidebar top)
- When a filter changes while on `Filtry` tab → no side effect; list rerenders normally
- Tab state NOT persisted to localStorage (always starts on Grzyby)

### Elements touched
- HTML: add `<div id="sb-tabs">` inside `#sb-top`, two `<button>` children
- CSS: tab row styles, active indicator, show/hide rules via `.sb-tab-grzyby` / `.sb-tab-filtry` body class (or direct style toggle via JS)
- JS: click handlers on tab buttons, toggle which section is visible

---

## Section 2 — Mobile Portrait: New Navigation Structure

### What changes
Remove the 3-tab bottom nav (`#mob-nav`). Replace with:
1. A slim fixed top bar (`#mob-top`)
2. A horizontal filter chip row (`#mob-chips`)
3. A single floating toggle pill (`#mob-pill`)
4. A full-screen list overlay (repurposed `#sidebar`)
5. A slide-up filter modal (`#filter-modal`, new element)

### Layout

```
Mobile portrait (375×812) — Map view:
┌────────────────────────┐  ← viewport
│ 🍄 GrzyboMapa    [🔍] │  52px  #mob-top (fixed)
├────────────────────────┤
│ Wszyst│Jadal│Truj│+Fil │  44px  #mob-chips (fixed, horizontal scroll)
├────────────────────────┤
│                        │
│        MAP             │  fills remaining space
│                        │  (padding-top: 96px to clear bars)
│                        │
│   [📋 Lista]           │  40px pill, fixed, bottom-center + 16px margin
└────────────────────────┘

Mobile portrait (375×812) — List view (after tapping Lista):
┌────────────────────────┐
│ 🍄 GrzyboMapa    [🔍] │  52px  #mob-top (fixed, still visible)
├────────────────────────┤
│ Wszyst│Jadal│Truj│+Fil │  44px  #mob-chips (fixed, still visible)
├────────────────────────┤
│  110 gatunków          │
│  ─ JADALNE ──────────  │
│  🌲 Borowik szlachetny │  full-screen #ml, scrollable
│  🌲 Borowik sosnowy    │
│  …                     │
│   [🗺️ Mapa]           │  pill switches back
└────────────────────────┘

Filter modal (after tapping + Filtry chip):
┌────────────────────────┐
│ 🍄 GrzyboMapa    [🔍] │
│ Wszyst│Jadal│Truj│+Fil │
│░░░░░░░░░░░░░░░░░░░░░░░│  scrim
│ ┌────────────────────┐ │
│ │ Filtry           ✕ │ │  slide-up modal
│ │  SEZON            │ │
│ │  SIEDLISKO        │ │
│ │  SORTUJ           │ │
│ └────────────────────┘ │
└────────────────────────┘
```

### #mob-top (new element)
- Height: 52px, background `var(--sb)`, border-bottom `var(--border)`
- Left: compact logo text "GrzyboMapa" in `var(--gold)` + small mushroom emoji
- Right: search icon button `🔍` — tapping opens `#sw` as a full-width overlay (same search logic as current, just triggered differently)
- Position: fixed, top: 0, z-index: 1350

### #mob-chips (new element)
- Height: 44px, background `var(--sb)`, border-bottom `var(--border)`
- Horizontal scroll, no scrollbar visible
- Chips: `Wszystkie | Jadalne | Trujące | Rzadkie | + Filtry`
  - First 4 chips map directly to the KATEGORIA filter (activeCats)
  - Active chip: gold background (`var(--gold)`, dark text)
  - `+ Filtry` chip: always secondary style, opens `#filter-modal`
- Position: fixed, top: 52px, z-index: 1350

### #mob-pill (new element)
- Position: fixed, bottom: calc(16px + env(safe-area-inset-bottom)), left: 50%, transform: translateX(-50%)
- z-index: 1310
- Pill button: border-radius: 20px, padding: 10px 20px, background `var(--sb)`, border `var(--border)`
- Icon + label: `📋 Lista` (map view) → `🗺️ Mapa` (list view)
- Box shadow for elevation

### #sidebar on mobile (repurposed)
- Position: fixed, top: 96px (below both bars), left: 0, right: 0, bottom: 0
- `transform: translateY(100%)` → `translateY(0)` when list active
- `transition: transform .3s`
- NO border-radius (no longer a bottom sheet)
- Inside: ONLY `#ml` visible. `#sb-top` hidden on mobile via CSS.
- `#ml` padding-bottom: calc(60px + env(safe-area-inset-bottom)) — room for pill

### #filter-modal (new element)
- Position: fixed, inset: 0, z-index: 1400
- Scrim: `background: rgba(0,0,0,.5)`, tap to close
- Sheet: position absolute, bottom: 0, left: 0, right: 0, border-radius: 16px 16px 0 0, background `var(--sb)`
- Max-height: 70vh, overflow-y: auto
- Header: "Filtry i sortowanie" + ✕ button
- Content: `#filters-panel` + `#season-bar` — teleported here via JS at page load when mobile (`document.getElementById('filter-modal-body').append(filtersPanel, seasonBar)` — 2 lines)
- On desktop: `#filters-panel` + `#season-bar` remain in `#sb-top`, `#filter-modal` stays `display:none`
- Slides up: `transform: translateY(100%)` → `translateY(0)` with transition

### Removed
- `#mob-nav` element → `display:none` on mobile (kept in DOM for landscape/legacy)
- Logo from inside `#sidebar` on mobile → `#mob-top` replaces it
- Bottom sheet border-radius from sidebar on mobile

### JS changes
- `mob-list` / `mob-filters` / `mob-map` click handlers → replaced by `#mob-pill` and `#mob-chips` handlers
- Opening list: add class `.mob-list-open` on body; sidebar slides up, pill shows Mapa
- Closing list: remove `.mob-list-open`; sidebar slides down, pill shows Lista
- Filter modal: `#mob-chips` button for `+ Filtry` opens `#filter-modal`; ✕ and scrim close it
- Category chips in `#mob-chips` call the same `activeCats` toggle logic as current filter buttons

---

## Section 3 — Mobile Landscape: Sidebar Handle

### What changes
Replace the floating `📋` button with a visible pull-tab on the left edge of the screen.

```
Landscape (812×375):
┌──────────────────────────────────────────┐
│ 🍄 GrzyboMapa              [🔍]          │  44px #mob-top
├──────────────────────────────────────────┤
│ Wszyst│Jadal│Truj│+Fil                   │  38px #mob-chips
├──┬───────────────────────────────────────┤
│◄ │                                        │  map
│  │           MAP                          │
│  │                                        │
│  └───────────────────────────────────────┘
 ↑ 
 #mob-sb-handle: fixed left edge pull-tab, vertically centered
 14px wide × 48px tall, border-radius 0 8px 8px 0
 Shows ◄ (closed) or ► (open)
```

When handle tapped → sidebar slides in from left (300px overlay), handle shows ►.

### #mob-sb-handle (new element)
- Position: fixed, left: 0, top: 50%, transform: translateY(-50%)
- z-index: 1300
- Width: 14px, Height: 48px
- Background: `var(--sb)`, border: right+top+bottom `var(--border)`, border-radius: 0 8px 8px 0
- Text: `‹` / `›` in `var(--muted)`, font-size: 10px
- Hidden on portrait, visible on landscape

### Sidebar in landscape
- When `#mob-sb-handle` clicked: sidebar `.mob-open` class added, sidebar slides in from left (same as current but triggered by handle not the old floating button)
- `#mob-top` and `#mob-chips` visible in landscape too (provides consistent branding + category access)
- `#ls-toggle` floating button → removed (replaced by handle)

---

## Section 4 — Visual Polish (all viewports)

These are secondary and can be deferred to a follow-up, but included here for completeness.

### Map control buttons
- Leaflet `+`/`−` zoom buttons: override default CSS, give `var(--card)` background + `var(--border)` border, match sidebar card aesthetic
- GPS button + heatmap button: same treatment — border `var(--border)`, subtle box-shadow

### Species cards
- Increase `padding` from current `10px 12px` to `11px 14px`
- `line-height: 1.4` on card text (currently implicit/tight)

### Filter chips (mob-chips and the existing .fb buttons)
- No design change to existing `.fb` chips — they already use gold active state
- `#mob-chips` uses same visual language as `.fb`

---

## Section 5 — What Does NOT Change

- All data: MUSHROOMS, FORESTS, LANDUSE, URBAN_DATA, bdlStore — untouched
- All logic: heatmap, scanner, BDL, filtering, sorting — untouched
- Info panel (`#ip`) — no change (slides in from right on desktop, full-screen on mobile)
- Analysis panel (`#ap`) — no change
- Layers panel (`#mc`) — no change
- Version bump rule: bump version string after implementation
- Git workflow: `git add index.html && git commit && git push`

---

## Implementation Notes

- Single `index.html` file — all CSS, HTML, JS inline
- New elements (`#mob-top`, `#mob-chips`, `#mob-pill`, `#mob-sb-handle`, `#filter-modal`) added to HTML body
- CSS changes isolated to `@media(max-width:767px)` and `@media(max-height:700px) and (orientation:landscape) and (max-width:1024px)` blocks plus new desktop tab rules
- JS: ~40 lines new code for tab/pill/modal handlers; existing mob-tab handlers replaced
- `#filters-panel` stays in DOM in its current position; on mobile it's also rendered inside `#filter-modal` via JS cloneNode or by moving it dynamically

---

## Out of Scope

- No new species added
- No heatmap/scanner changes
- No backend
- No new dependencies
