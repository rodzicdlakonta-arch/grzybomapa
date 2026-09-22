# Navigation Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the broken 3-tab bottom nav + 80vh bottom-sheet with a top bar + chip row + full-screen list overlay + landscape pull-tab handle.

**Architecture:** All changes are inline edits to `index.html`. New HTML elements added before `</body>`. New CSS rules appended before `</style>`. New JS appended at end of existing `<script>` block (before mob-nav script at line 43171). Existing elements are patched in-place using Edit tool.

**Tech Stack:** Vanilla JS, Leaflet 1.9.4, CSS custom properties already defined in `:root`. No new dependencies.

**Spec:** `docs/superpowers/specs/2026-09-21-navigation-redesign.md`

## Global Constraints

- Single `index.html` file — NO build step, NO npm, NO new files created
- Version bump required: find `<span style="letter-spacing:.04em">v1.29.77</span>` and change to `v1.30.0`
- NO functional changes to scanner logic, heatmap, BDL, filtering/sorting algorithms
- Scanner draw buttons may be visually restyled only — `_btnStyle()` function at line 41658 is the target
- `#mob-nav` stays in DOM (hidden) — its `#mob-map`, `#mob-list`, `#mob-filters` are referenced by existing handlers
- `window._closeSidebar` must remain callable after all changes (scanner uses it at line 41646)
- Git commit after every task: `git add index.html && git commit`

## Review Focus

1. `window._closeSidebar` must be defined by the time scanner opens — landscape IIFE overrides the portrait version; scanner must work in both orientations
2. `#filters-panel` teleported to `#filter-modal-body` must still trigger `renderList()` + `_updateFilterBadge()` — the existing `.fb[data-cat]` handlers stay attached since we move the element not clone it
3. Rotating from portrait to landscape after page load — `#filters-panel` would be inside `#filter-modal-body` but landscape sidebar expects it in `#sb-top`; test in portrait only per scope
4. `#ap.open{bottom:56px}` currently clears mob-nav height — Task 2 changes this to `bottom:0`, test scanner panel doesn't overlap portrait controls
5. Desktop `sbt-filtry` body class hides `#sw` — verify existing search handler (line 42177) still opens list when desktop tab switches back

---

### Task 1: Desktop Sidebar Tabs

**Files:**
- Modify: `index.html` — HTML around line 375, CSS around line 367, JS before line 43186

**Interfaces:**
- Produces: body classes `sbt-grzyby` / `sbt-filtry`, CSS rules that hide/show sidebar sections by body class

- [ ] **Step 1: Add HTML for tab row**

Find the closing `</div>` of `#logo` (the one before `<div id="sw">`, around line 384). Insert after it:

```html
  <div id="sb-tabs">
    <button class="sb-tab active" data-tab="grzyby">Grzyby</button>
    <button class="sb-tab" data-tab="filtry">Filtry</button>
  </div>
```

Exact old string to replace (line ~384):
```
  </div>
  <div id="sw">
```
Replace with:
```
  </div>
  <div id="sb-tabs">
    <button class="sb-tab active" data-tab="grzyby">Grzyby</button>
    <button class="sb-tab" data-tab="filtry">Filtry</button>
  </div>
  <div id="sw">
```

- [ ] **Step 2: Add CSS for tabs**

Append before `</style>` (line 367):

```css
/* ── Desktop sidebar tabs ── */
#sb-tabs{display:flex;border-bottom:1px solid var(--border);flex-shrink:0;padding:0 12px}
.sb-tab{flex:1;background:none;border:none;border-bottom:2px solid transparent;padding:9px 0 8px;font-size:11px;font-weight:600;letter-spacing:.05em;text-transform:uppercase;color:var(--muted);cursor:pointer;transition:color .15s,border-color .15s;font-family:var(--f-body)}
.sb-tab.active{color:var(--gold);border-bottom-color:var(--gold)}
.sb-tab:hover:not(.active){color:var(--text)}
@media(min-width:768px){
  body.sbt-filtry #sw,body.sbt-filtry #ml{display:none!important}
  body.sbt-grzyby #filters-panel,body.sbt-grzyby #season-bar{display:none!important}
  #ml-resize{display:none!important}
}
@media(max-width:767px){#sb-tabs{display:none!important}}
```

- [ ] **Step 3: Add JS for tab switching**

Find `// ── Landscape list-toggle` (line 43113) and insert BEFORE it:

```js
// ── Desktop sidebar tabs ──────────────────────────────────────
(()=>{
  document.body.classList.add('sbt-grzyby');
  document.querySelectorAll('.sb-tab').forEach(btn=>{
    btn.onclick=()=>{
      document.querySelectorAll('.sb-tab').forEach(b=>b.classList.remove('active'));
      btn.classList.add('active');
      const t=btn.dataset.tab;
      document.body.classList.toggle('sbt-grzyby',t==='grzyby');
      document.body.classList.toggle('sbt-filtry',t==='filtry');
    };
  });
})();

```

- [ ] **Step 4: Verify in browser**

Open `index.html` in browser. At ≥768px wide:
- Two tabs appear below the logo: `GRZYBY` and `FILTRY`
- `GRZYBY` active by default — species list visible, filters hidden
- Click `FILTRY` — filters visible, species list hidden, active underline moves
- Click `GRZYBY` again — list returns

At ≤767px: tabs must be invisible.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: desktop sidebar tabs (Grzyby/Filtry) — v1.30.0-wip

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

### Task 2: Mobile Top Bar + Chips

**Files:**
- Modify: `index.html` — HTML after `</div>` of `#mob-nav` (line 43184), CSS in `@media(max-width:767px)` block

**Interfaces:**
- Produces: `#mob-top`, `#mob-chips`, `.mchip` elements; `syncMobChips()` function for Task 3 to call
- Consumes: `activeCats` Set (line 42319), `renderList()`, `_updateFilterBadge()`

- [ ] **Step 1: Add top bar + chips HTML**

Add after `</div>` closing `#mob-nav` (line 43184), before `<script>`:

```html
<div id="mob-top">
  <div id="mob-logo">🍄 <span>GrzyboMapa</span></div>
  <button id="mob-srch" aria-label="Szukaj">🔍</button>
</div>
<div id="mob-chips">
  <button class="mchip on" data-mcat="all">Wszystkie</button>
  <button class="mchip" data-mcat="edible">Jadalne</button>
  <button class="mchip" data-mcat="dangerous">Trujące</button>
  <button class="mchip" data-mcat="rare">Rzadkie</button>
  <button class="mchip mchip-more" id="mob-more">🎛️ Filtry</button>
</div>
```

- [ ] **Step 2: Add CSS (new rules block)**

Append before `</style>`:

```css
/* ── Mobile top bar + chips ── */
#mob-top{display:none;position:fixed;top:0;left:0;right:0;height:52px;background:var(--sb);border-bottom:1px solid var(--border);z-index:1350;align-items:center;justify-content:space-between;padding:0 14px}
#mob-logo{font-size:15px;font-weight:700;color:var(--gold)}
#mob-logo span{font-size:13px;opacity:.85}
#mob-srch{background:none;border:none;font-size:20px;cursor:pointer;color:var(--muted);padding:4px;line-height:1}
#mob-srch:hover{color:var(--text)}
#mob-chips{display:none;position:fixed;top:52px;left:0;right:0;height:44px;background:var(--sb);border-bottom:1px solid var(--border);z-index:1350;align-items:center;padding:0 12px;gap:7px;overflow-x:auto;overflow-y:hidden;scrollbar-width:none}
#mob-chips::-webkit-scrollbar{display:none}
.mchip{padding:5px 11px;border-radius:16px;border:1px solid var(--goldb,#a0845c55);background:transparent;color:var(--text);font-size:11.5px;font-weight:500;cursor:pointer;transition:all .15s;white-space:nowrap;flex-shrink:0;font-family:var(--f-body)}
.mchip:hover{border-color:var(--gold);background:var(--goldd,rgba(180,130,60,.15))}
.mchip.on{background:var(--gold);border-color:var(--gold);color:#1a1208;font-weight:600}
.mchip-more{border-style:dashed}
```

- [ ] **Step 3: Update existing `@media(max-width:767px)` CSS block**

Patch these lines inside the existing `@media(max-width:767px){...}` block:

Change line 295 `#mw{padding-bottom:56px}` → `#mw{padding-bottom:0}`

Change line 297 `#ap.open{bottom:56px}` → `#ap.open{bottom:0}`

Change line 320 `#mob-nav{display:flex}` → `#mob-nav{display:none!important}`

Add these lines inside the block (append just before the closing `}`):
```css
  #mob-top{display:flex}
  #mob-chips{display:flex}
  .leaflet-top.leaflet-left{top:96px!important}
  #logo{display:none!important}
```

- [ ] **Step 4: Add JS for chips + search button**

Add to the new JS section (before `// ── Landscape list-toggle`):

```js
// ── Mobile top chips + search ─────────────────────────────────
(()=>{
  function syncMobChips(){
    document.querySelectorAll('.mchip[data-mcat]').forEach(c=>{
      const cat=c.dataset.mcat;
      if(cat==='all') c.classList.toggle('on',activeCats.size===0);
      else c.classList.toggle('on',activeCats.has(cat));
    });
  }
  window.syncMobChips=syncMobChips;

  document.querySelectorAll('.mchip[data-mcat]').forEach(chip=>{
    chip.onclick=()=>{
      // delegate to existing .fb filter button logic
      const cat=chip.dataset.mcat;
      document.querySelector(`.fb[data-cat="${cat}"]`)?.click();
      syncMobChips();
    };
  });

  // Search icon → open list (pill must exist — Task 3 adds it)
  document.getElementById('mob-srch')?.addEventListener('click',()=>{
    window._openMobList?.();
    setTimeout(()=>document.getElementById('si')?.focus(),100);
  });

  syncMobChips();
})();
```

Also hook `syncMobChips()` after existing filter button clicks. Find the existing filter JS block (line ~42395):
```js
           renderList();_updateFilterBadge();
         };
```
Replace with:
```js
           renderList();_updateFilterBadge();
           if(typeof syncMobChips==='function')syncMobChips();
         };
```
(This is inside the `.forEach(btn=>{btn.onclick=()=>{...}})` for `.fb[data-cat]` buttons.)

- [ ] **Step 5: Verify in browser**

At ≤767px:
- Old bottom nav gone, top bar visible with mushroom logo and 🔍
- Chip row below it: Wszystkie | Jadalne | Trujące | Rzadkie | Filtry
- `Wszystkie` chip highlighted gold
- Click `Jadalne` → chip turns gold, species list filters
- Click `Wszystkie` → all species, chip syncs
- Leaflet zoom +/− buttons are below the bars (not behind them)
- 🔍 button — clicking it should open the list overlay (needs Task 3 first; currently no-op is OK)

At ≥768px: top bar and chips must be invisible, existing desktop layout unchanged.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: mobile top bar + filter chips replace bottom nav

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

### Task 3: Mobile Full-Screen List Overlay + Pill

**Files:**
- Modify: `index.html` — HTML after `#mob-chips`, CSS patch in `@media(max-width:767px)`, JS

**Interfaces:**
- Produces: `window._openMobList()`, `window._closeMobList()`, redefines `window._closeSidebar`
- Consumes: `#mob-pill` (new element), `#sidebar`, `.mob-open` class

- [ ] **Step 1: Add pill HTML**

Add after `</div>` closing `#mob-chips` (just before `<script>` tag at 43185):

```html
<button id="mob-pill" aria-label="Lista gatunków">📋 Lista</button>
```

- [ ] **Step 2: Add pill CSS**

Append before `</style>`:

```css
/* ── Mobile pill ── */
#mob-pill{display:none;position:fixed;bottom:calc(16px + env(safe-area-inset-bottom));left:50%;transform:translateX(-50%);z-index:1310;padding:10px 22px;border-radius:20px;background:var(--sb);border:1px solid var(--border);box-shadow:0 3px 12px rgba(0,0,0,.45);cursor:pointer;font-size:13px;font-weight:600;color:var(--text);white-space:nowrap;transition:opacity .15s}
#mob-pill:active{opacity:.7}
@media(max-width:767px){#mob-pill{display:block}}
```

- [ ] **Step 3: Patch sidebar CSS for full-screen overlay**

Inside the existing `@media(max-width:767px)` block, replace the `#sidebar` rule (line 292):

Old:
```css
  #sidebar{position:fixed;bottom:0;left:0;right:0;width:100%!important;min-width:0!important;height:80vh;border-right:none;border-top:1px solid var(--border);border-radius:16px 16px 0 0;transform:translateY(100%);transition:transform .3s cubic-bezier(.4,0,.2,1)!important;z-index:1200}
```
New:
```css
  #sidebar{position:fixed;top:96px;left:0;right:0;bottom:0;width:100%!important;min-width:0!important;height:auto!important;border-right:none;border-top:none;border-radius:0;transform:translateY(100%);transition:transform .3s cubic-bezier(.4,0,.2,1)!important;z-index:1200}
```

Also update `#ml` padding (line 313). Change:
```css
  #ml{padding-bottom:calc(60px + env(safe-area-inset-bottom))}
```
to:
```css
  #ml{padding-bottom:calc(70px + env(safe-area-inset-bottom))}
```
(extra room for the pill at bottom)

- [ ] **Step 4: Add pill JS**

Add to the new JS section (BEFORE `// ── Mobile top chips + search`):

```js
// ── Mobile portrait list overlay + pill ───────────────────────
(()=>{
  const pill=document.getElementById('mob-pill');
  const sb=document.getElementById('sidebar');
  if(!pill||!sb)return;

  function _openMobList(){
    sb.classList.add('mob-open');
    pill.textContent='🗺️ Mapa';
  }
  function _closeMobList(){
    sb.classList.remove('mob-open');
    pill.textContent='📋 Lista';
    if(typeof syncMobChips==='function')syncMobChips();
  }

  window._openMobList=_openMobList;
  window._closeMobList=_closeMobList;
  // portrait version — landscape IIFE will override this for landscape phones
  window._closeSidebar=_closeMobList;

  pill.onclick=()=>{
    if(sb.classList.contains('mob-open')) _closeMobList();
    else _openMobList();
  };
})();
```

- [ ] **Step 5: Update search handler to use `_openMobList`**

Find (line 42177-42183):
```js
  if(window.innerWidth<=767){
    const sb=document.getElementById('sidebar');
    const tabList=document.getElementById('mob-list');
    if(!sb.classList.contains('mob-open')){
      sb.classList.add('mob-open');
      if(tabList)document.querySelectorAll('.mob-tab').forEach(t=>{t.classList.toggle('active',t===tabList);});
    }
  }
```
Replace with:
```js
  if(window.innerWidth<=767){
    if(typeof window._openMobList==='function') window._openMobList();
    else document.getElementById('sidebar')?.classList.add('mob-open');
  }
```

- [ ] **Step 6: Update pcls (info panel close) to use `_openMobList`**

Find (line 42696-42699):
```js
  if(window.innerWidth<=767){
    const sb=document.getElementById('sidebar');
    setTimeout(()=>{sb.classList.add('mob-open');},80);
  }
```
Replace with:
```js
  if(window.innerWidth<=767){
    setTimeout(()=>{if(typeof window._openMobList==='function') window._openMobList(); else document.getElementById('sidebar')?.classList.add('mob-open');},80);
  }
```

- [ ] **Step 7: Verify in browser**

At ≤767px:
- Map view: floating pill `📋 Lista` at bottom center
- Tap pill → sidebar slides up from 96px to bottom, fills screen, pill changes to `🗺️ Mapa`
- In list view: search bar (`#sw`) at top, then species list (`#ml`) scrollable
- Tap `🗺️ Mapa` → sidebar slides down, map returns
- `#logo` inside sidebar: invisible (body CSS hides it)
- Scanner: opening scan panel (draw a polygon) → sidebar closes, pill shows `📋 Lista` again

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: mobile full-screen list overlay + pill toggle

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

### Task 4: Mobile Filter Modal

**Files:**
- Modify: `index.html` — HTML, CSS, JS

**Interfaces:**
- Consumes: `#filters-panel`, `#season-bar` (teleported from `#sb-top` on mobile load)
- Produces: `#filter-modal` element, `#mob-more` click opens it

- [ ] **Step 1: Add modal HTML**

Add after the `#mob-pill` button HTML (before `<script>` at line 43185):

```html
<div id="filter-modal">
  <div id="filter-scrim"></div>
  <div id="filter-sheet">
    <div id="filter-sheet-head">
      <span>Filtry i sortowanie</span>
      <button id="filter-close">✕</button>
    </div>
    <div id="filter-modal-body"></div>
  </div>
</div>
```

- [ ] **Step 2: Add modal CSS**

Append before `</style>`:

```css
/* ── Mobile filter modal ── */
#filter-modal{display:none;position:fixed;inset:0;z-index:1400;flex-direction:column;justify-content:flex-end}
#filter-modal.open{display:flex}
#filter-scrim{position:absolute;inset:0;background:rgba(0,0,0,.5)}
#filter-sheet{position:relative;background:var(--sb);border-radius:16px 16px 0 0;max-height:75vh;overflow-y:auto;z-index:1;padding-bottom:env(safe-area-inset-bottom)}
#filter-sheet-head{display:flex;align-items:center;justify-content:space-between;padding:14px 16px 12px;border-bottom:1px solid var(--border);font-weight:600;font-size:14px;flex-shrink:0;position:sticky;top:0;background:var(--sb);z-index:1}
#filter-close{background:none;border:none;cursor:pointer;font-size:20px;color:var(--muted);padding:2px 8px;line-height:1}
#filter-modal-body{padding:0 8px 16px}
```

- [ ] **Step 3: Add filter modal JS**

Add to new JS section (after pill JS, before chips JS):

```js
// ── Mobile filter modal ───────────────────────────────────────
(()=>{
  const modal=document.getElementById('filter-modal');
  const body=document.getElementById('filter-modal-body');
  if(!modal||!body)return;

  // Teleport filters to modal on portrait mobile load only
  if(window.innerWidth<=767){
    const fp=document.getElementById('filters-panel');
    const sb=document.getElementById('season-bar');
    if(fp)body.appendChild(fp);
    if(sb)body.appendChild(sb);
  }

  function openModal(){modal.classList.add('open');}
  function closeModal(){modal.classList.remove('open');}

  document.getElementById('mob-more')?.addEventListener('click',openModal);
  document.getElementById('filter-close')?.addEventListener('click',closeModal);
  document.getElementById('filter-scrim')?.addEventListener('click',closeModal);
})();
```

- [ ] **Step 4: Verify in browser**

At ≤767px:
- Tap `🎛️ Filtry` chip in top bar → modal slides up with scrim
- Modal shows KATEGORIA + SEZON + SIEDLISKO + SORTUJ sections (the original `#filters-panel` + `#season-bar`)
- Changing a filter inside modal → species list in background updates (existing handlers still attached)
- Tap ✕ or scrim → modal closes
- `syncMobChips()` in chip JS will sync chips to reflect active filters after modal closes

At ≥768px: modal never shown, filters visible in sidebar under `FILTRY` tab (Task 1 confirms this).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: mobile filter modal with teleported filter panel

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

### Task 5: Landscape Handle (replace ls-toggle)

**Files:**
- Modify: `index.html` — HTML (add `#mob-handle`), CSS in landscape block, JS landscape IIFE

**Interfaces:**
- Consumes: existing landscape IIFE at line 43114 (refactors it to use `#mob-handle`)
- Produces: `#mob-handle` pull-tab, visible only in large-phone landscape

- [ ] **Step 1: Add handle HTML**

Add before `<div id="mob-top">`:

```html
<button id="mob-handle" style="display:none" aria-label="Pokaż listę">‹</button>
```

- [ ] **Step 2: Add handle CSS to landscape block**

Inside `@media(max-height:700px) and (orientation:landscape) and (max-width:1024px)` (line 337), add before its closing `}`:

```css
  #mob-handle{display:flex!important;position:fixed;left:0;top:50%;transform:translateY(-50%);z-index:1300;width:14px;height:48px;background:var(--sb);border:1px solid var(--border);border-left:none;border-radius:0 8px 8px 0;font-size:11px;color:var(--muted);cursor:pointer;align-items:center;justify-content:center;padding:0}
  #ls-toggle{display:none!important}
  /* mob-top stays visible in landscape but reduced height */
  #mob-top{display:flex!important;height:44px}
  #mob-chips{display:flex!important;top:44px;height:38px}
  .leaflet-top.leaflet-left{top:82px!important}
  #mw{padding-bottom:0;padding-right:0}
```

- [ ] **Step 3: Refactor landscape IIFE**

Find the landscape IIFE at line 43114. Replace it entirely with:

```js
// ── Landscape list-toggle (large phones only — portrait uses pill) ──
(()=>{
  const handle=document.getElementById('mob-handle');
  const sb=document.getElementById('sidebar');
  const ip=document.getElementById('ip');
  if(!handle||!sb||!ip)return;
  const isLargeLS=()=>window.innerWidth>window.innerHeight&&window.innerWidth>767&&window.innerWidth<=1024;
  let _sbOpen=false,_ipWasClosed=false;
  function _restoreIp(){if(_ipWasClosed){_ipWasClosed=false;ip.classList.add('open');}}
  function _update(){
    if(isLargeLS()){
      handle.style.display='flex';
      handle.textContent=_sbOpen?'›':'‹';
      handle.setAttribute('aria-label',_sbOpen?'Ukryj listę':'Pokaż listę');
    }else{
      handle.style.display='none';
      if(_sbOpen){_sbOpen=false;sb.classList.remove('mob-open');}
      _restoreIp();
    }
  }
  handle.onclick=e=>{
    e.stopPropagation();
    _sbOpen=!_sbOpen;
    sb.classList.toggle('mob-open',_sbOpen);
    if(!_sbOpen&&ip.classList.contains('open')){
      _ipWasClosed=true;ip.classList.remove('open');ip.classList.remove('ip-mini');
    }else if(_sbOpen){
      _restoreIp();
    }
    _update();
    map.invalidateSize();
  };
  new MutationObserver(_update).observe(sb,{attributes:true,attributeFilter:['class']});
  window.addEventListener('orientationchange',()=>{_sbOpen=false;sb.classList.remove('mob-open');_restoreIp();setTimeout(_update,350);});
  window.addEventListener('resize',_update);
  // Override portrait _closeSidebar for landscape phones
  window._closeSidebar=()=>{_sbOpen=false;sb.classList.remove('mob-open');_update();};
  _update();
})();
```

- [ ] **Step 4: Verify in browser**

In landscape (emulate 812×375 in DevTools):
- Left edge shows `‹` pull-tab, vertically centered
- Tap `‹` → sidebar slides in from left (300px), arrow becomes `›`
- Tap `›` → sidebar closes
- `📋 ls-toggle` floating button is gone

In portrait (375×812): handle invisible.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: landscape sidebar handle replaces ls-toggle button

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

### Task 6: Visual Polish + Version Bump

**Files:**
- Modify: `index.html` — CSS, version string, scanner button style

**Interfaces:** None — pure visual changes, no API surface

- [ ] **Step 1: Species card spacing**

Find in CSS (around line 200, the `.ml-item` rule):

```css
.ml-item{
```
Look for its `padding` declaration and increase from current value by 2px each side. Typical current: `padding:10px 12px` → change to `padding:12px 14px`.

Also add `line-height:1.45` to `.ml-item` text elements if not set.

- [ ] **Step 2: Scanner draw buttons visual**

Find `_btnStyle` function (line 41658):
```js
function _btnStyle(a){a.style.fontSize='16px';a.style.lineHeight='30px';a.style.width='30px';a.style.display='block';a.style.textAlign='center';}
```
Replace with:
```js
function _btnStyle(a){a.style.fontSize='15px';a.style.lineHeight='28px';a.style.width='28px';a.style.display='block';a.style.textAlign='center';a.style.borderRadius='6px';}
```
(Only font/size/radius changes — NO logic changes.)

- [ ] **Step 3: Version bump**

Find version string:
```html
<span style="letter-spacing:.04em">v1.29.77</span>
```
Replace with:
```html
<span style="letter-spacing:.04em">v1.30.0</span>
```

- [ ] **Step 4: Final visual verify**

Open browser preview, check all viewports:
- Desktop ≥768px: tabs work, species cards slightly roomier
- Mobile portrait ≤767px: top bar + chips + pill, list overlay, filter modal
- Mobile landscape: handle visible, mob-top + chips at top

- [ ] **Step 5: Final commit + push**

```bash
git add index.html
git commit -m "feat: visual polish + v1.30.0

- scanner draw buttons: smaller, border-radius added
- species cards: padding +2px
- version: 1.29.77 → 1.30.0

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
git push
```
