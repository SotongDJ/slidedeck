---
name: mobile-slidedeck
description: >
  Create a single-page Card Box web app with vanilla JS and CSS — no external frameworks.
  Three-level hierarchy: Box → Set → Card. A Set with one Card is equivalent to a classic slide.
  Default orientation is portrait (9:16); use landscape (16:9) for "landscape", "widescreen", "desktop", or "16:9".
  Supports a runtime orientation-toggle.
  Navigation: horizontal swipe/arrows/keyboard moves between Sets; vertical moves between Cards within a Set.
  Cover card (first card of a multi-card Set) expands on tap/click/SPACE.
  Use this skill whenever the user asks to turn content into a slide deck, presentation, card box, or swipeable mobile web app.
  Trigger on: "make slides", "mobile slide deck", "swipeable presentation", "card box", "one slide per chapter",
  or any request to present structured content as navigable cards.
  Also trigger for data comparisons, ratings, rankings, scorecards, tier progressions, or content that benefits
  from charts, radar plots, stat strips, or visual matrices.
---

# Mobile Card Box Skill

Converts structured content into a polished single-page web app with 2D card navigation.
Output: one self-contained `.html` file. No external JS or CSS frameworks.

**Three-level hierarchy:**
- **Box** — the whole app (one per file); the HTML body is the desktop surface
- **Set** — a group of thematically related Cards; Sets are arranged horizontally
- **Card** — an individual content unit within a Set; Cards within a Set are stacked vertically

A Set with one Card renders identically to a classic slide (no cover mechanic).
A Set with multiple Cards shows Card 0 as a **cover**; tap / click / SPACE expands downward to reveal Cards 1…N.
Navigating UP back to Card 0 collapses the Set.

**Version: 2.0.1**

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| **2.0.1** | 2026-05-02 | Fix `.set` layout bug: `align-items:flex-start` (was `center`). Centering a multi-card `card-track` (CPR×100dvh) inside a 100dvh set pushed card 0 above the viewport, making cover cards invisible. |
| **2.0.0** | 2026-05-01 | Layout replaced: slide deck → Card Box. Three-level hierarchy (Box → Set → Card). 2D navigation: horizontal between Sets (←→), vertical between Cards (↑↓). Cover card expand/collapse on tap/click/SPACE. Directional nav buttons (4 edges) replace bottom dot bar. Page indicator: `SET 01/N · Y/M` (card suffix omitted for single-card Sets). `card-track` height = `numCards×100dvh`; `set-track` width = `numSets×100vw`; both translated by percentage. Font-size controlled via `--card-font` CSS var set on `.box`. |
| **1.7.1** | 2026-04-22 | Radar chart overflow-safe layout; align-self:stretch + margin:0 on landscape slide; height-driven sizing in landscape; preserveAspectRatio; viewBox formula; 6-axis pre-computed vectors |
| **1.7.0** | 2026-04-22 | Visualization library: SVG radar chart, segment bar, stat strip, KPI insight card, quote callout, tier ladder, brand accent card; semantic CSS vars |
| **1.6.0** | 2026-04-11 | Print/PDF button; slidesToPrintPDF(); orientation-aware |
| **1.5.0** | 2026-04-11 | Codename + FA icons; localStorage + URL query string persistence; Share + Reset |
| **1.4.1** | 2026-04-10 | Restored Step 0 heading; removed stale body margin |
| **1.4.0** | 2026-04-10 | Deck always centred; safe-area insets; viewport-fit=cover; 100dvh |
| **1.3.0** | 2026-04-10 | Output HTML/CSS/JS: zero indentation, no comments |
| **1.2.0** | 2026-04-10 | Font size buttons centred; range 3-69px step 3; portrait default 21px, landscape 18px |
| **1.1.0** | 2026-04-10 | Orientation toggle; runtime data-orient attribute |
| **1.0.0** | 2026-04-10 | Initial release |

---

## Step 0.5 — Output formatting rules

**These rules apply to every line of HTML, CSS, and JS written into the output file. No exceptions.**

- **No indentation.** All tags, rules, and statements start at column 0.
- **No comments.** No HTML, CSS, or JS comments of any kind.
- The SKILL.md uses indented, commented code blocks only as readable reference; the actual output must strip all of that.

---

## Step 0 — Gather inputs & detect orientation

Before writing any code, collect from the user if not already provided:

1. **Codename** (required) — output filename and localStorage key prefix.
2. **Font Awesome kit URL** (required) — `https://kit.fontawesome.com/{10-digit-id}.js`.
3. **Number of Sets** (N).
4. **Cards per Set** — array `[c0, c1, …, cN-1]`; value `1` = single-card Set (no cover/expand mechanic).
5. Detect orientation: Portrait (default, 9:16) or Landscape (16:9).
6. Always include the orientation-toggle button in the topbar.

---

## Step 1 — HTML skeleton

```html
<!DOCTYPE html>
<html lang="..." data-theme="light" data-orient="portrait">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<title>...</title>
<link href="https://fonts.googleapis.com/..." rel="stylesheet">
<script src="https://kit.fontawesome.com/{10-digit-id}.js" crossorigin="anonymous"></script>
</head>
<body>
<div class="box" id="box">

  <!-- Topbar -->
  <div class="box-top" id="boxTop">
    <div class="top-left"><span id="pageNum"></span></div>
    <div class="top-center">
      <button id="btnMinus">A-</button>
      <button id="btnPlus">A+</button>
    </div>
    <div class="top-right">
      <button id="orientBtn"></button>
      <button id="themeBtn"></button>
      <button id="printBtn"><i class="fa-solid fa-fw fa-print"></i></button>
      <button id="shareBtn"><i class="fa-solid fa-fw fa-share"></i></button>
      <button id="resetBtn"><i class="fa-solid fa-fw fa-arrow-rotate-left"></i></button>
    </div>
  </div>

  <!-- Set track — horizontal rail -->
  <div class="set-track" id="setTrack">

    <!-- Multi-card Set example (Set 0, 3 Cards) -->
    <div class="set" data-set="0">
      <div class="card-track" id="ct0">
        <div class="card">
          <div class="card-face cover-card">
            <!-- cover card content: slide-tag, h2.slide-title, etc. -->
            <div class="expand-hint"><i class="fa-solid fa-fw fa-chevron-down"></i></div>
          </div>
        </div>
        <div class="card">
          <div class="card-face"><!-- card 1 content --></div>
        </div>
        <div class="card">
          <div class="card-face"><!-- card 2 content --></div>
        </div>
      </div>
    </div>

    <!-- Single-card Set example (Set 1) -->
    <div class="set" data-set="1">
      <div class="card-track" id="ct1">
        <div class="card">
          <div class="card-face"><!-- single card content --></div>
        </div>
      </div>
    </div>

  </div>

  <!-- Directional nav buttons -->
  <button class="nav-btn nav-left"  id="navLeft"><i class="fa-solid fa-fw fa-chevron-left"></i></button>
  <button class="nav-btn nav-right" id="navRight"><i class="fa-solid fa-fw fa-chevron-right"></i></button>
  <button class="nav-btn nav-up"   id="navUp"><i class="fa-solid fa-fw fa-chevron-up"></i></button>
  <button class="nav-btn nav-down" id="navDown"><i class="fa-solid fa-fw fa-chevron-down"></i></button>

</div>
</body>
```

**Authoring rules:**
- `.cover-card` class only on the first `.card-face` of a multi-card Set (CPR[s] > 1). Never on single-card Sets.
- `.expand-hint` only inside `.cover-card`.
- Each `.card-track` has `id="ct{s}"` where s = Set index (0-based).
- Each `.set` has `data-set="{s}"`.

---

## Step 2 — CSS rules (must follow exactly)

```css
html, body, h2, table, th, td, hr { margin: 0; padding: 0; }
div { padding: 0; margin: 0; }
*, *::before, *::after { box-sizing: border-box; }

body {
width: 100vw;
height: 100dvh;
overflow: hidden;
background: var(--bg-outer);
}

.box {
position: fixed;
top: 0; left: 0;
width: 100vw;
height: 100dvh;
overflow: hidden;
background: var(--bg-outer);
--top-h: 40px;
}

.box-top {
position: absolute;
top: 0; left: 0; right: 0;
z-index: 200;
height: var(--top-h);
display: flex;
align-items: center;
justify-content: space-between;
padding: 0 0.5em;
background: var(--bg);
border-bottom: 1px solid var(--border);
font-size: 14px;
}

.set-track {
display: flex;
flex-direction: row;
height: 100dvh;
will-change: transform;
transition: transform 0.55s cubic-bezier(0.77,0,0.175,1);
}

.set {
flex: 0 0 100vw;
height: 100dvh;
overflow: hidden;
display: flex;
align-items: flex-start;
justify-content: center;
}

.card-track {
display: flex;
flex-direction: column;
will-change: transform;
transition: transform 0.55s cubic-bezier(0.77,0,0.175,1);
}

.card {
flex: 0 0 100dvh;
width: 100vw;
display: flex;
align-items: center;
justify-content: center;
}

.card-face {
--aw: calc(100vw - 8vw - env(safe-area-inset-left) - env(safe-area-inset-right));
--ah: calc(100dvh - var(--top-h) - 8vw - env(safe-area-inset-top) - env(safe-area-inset-bottom));
width: min(var(--aw), calc(var(--ah) * 9 / 16));
height: min(var(--ah), calc(var(--aw) * 16 / 9));
font-size: var(--card-font, clamp(16px, 7vw, 28px));
background: var(--bg);
display: flex;
flex-direction: column;
gap: 0.5em;
overflow: hidden;
flex-shrink: 0;
position: relative;
}

[data-orient="landscape"] .card-face {
width: min(var(--aw), calc(var(--ah) * 16 / 9));
height: min(var(--ah), calc(var(--aw) * 9 / 16));
font-size: var(--card-font, clamp(12px, 3.5vw, 22px));
}

.cover-card { cursor: pointer; }

.expand-hint {
position: absolute;
bottom: 0.5em;
left: 50%;
transform: translateX(-50%);
color: var(--text-muted);
font-size: 0.8em;
pointer-events: none;
animation: expandBounce 1.8s ease-in-out infinite;
}
@keyframes expandBounce {
0%,100% { opacity:0.45; transform:translateX(-50%) translateY(0); }
50% { opacity:1; transform:translateX(-50%) translateY(4px); }
}

.nav-btn {
position: absolute;
z-index: 150;
background: var(--surface);
border: 1px solid var(--border);
color: var(--text-muted);
cursor: pointer;
display: flex;
align-items: center;
justify-content: center;
width: 2em;
height: 2em;
border-radius: 50%;
font-size: 13px;
opacity: 0.65;
transition: opacity 0.2s;
padding: 0;
}
.nav-btn:hover { opacity: 1; }
.nav-btn:disabled { opacity: 0.15; pointer-events: none; }
.nav-left  { left: 0.4em; top: 50%; transform: translateY(-50%); }
.nav-right { right: 0.4em; top: 50%; transform: translateY(-50%); }
.nav-up   { top: calc(var(--top-h) + 0.4em); left: 50%; transform: translateX(-50%); }
.nav-down  { bottom: calc(env(safe-area-inset-bottom) + 0.4em); left: 50%; transform: translateX(-50%); }
```

**Width/height rules:**
- `set-track` width = `SETS * 100vw` — set by JS on init.
- Each `card-track` height = `CPR[s] * 100dvh` — set by JS on init.
- `.card-face` dimensions are viewport-based (independent of parent padding).
- Translations are percentage-based: `translateX(-(curSet/SETS)*100%)` and `translateY(-(curCard[s]/CPR[s])*100%)`.

---

## Step 3 — Theme system

Three themes: `light` (default), `dark`, `system`.

```css
:root {
--bg: #F9F9F6;  --bg-outer: #ECEAE5;
--text: #0F0F0D;  --text-muted: #7A7A72;
--border: #DDDDD6;  --surface: #F1F1EE;  --surface-2: #E5E5E0;
--dot-on: #0F0F0D;  --dot-off: #C8C8C0;
--yes: #1E7A3C;  --no: #B83030;
--bar: #0F0F0D;  --bar-bg: #E0E0DA;
--shadow: rgba(0,0,0,0.14);
--accent: #1E7A3C;  --accent-bg: #E6F0E8;
--ok: #063E18;  --ok-bg: #F6FAF7;
--warn: #5C2E00;  --warn-bg: #FEFAF1;
--danger: #5C0D0D;  --danger-bg: #FDF4F3;
}
[data-theme="dark"] {
--bg: #0D0D0B;  --bg-outer: #070705;
--text: #EEEEE8;  --text-muted: #888880;
--border: #252520;  --surface: #141412;  --surface-2: #1C1C18;
--dot-on: #EEEEE8;  --dot-off: #3A3A34;
--yes: #4DC870;  --no: #E06060;
--bar: #EEEEE8;  --bar-bg: #1C1C18;
--shadow: rgba(0,0,0,0.5);
--accent: #4DC870;  --accent-bg: #0A1F0E;
--ok: #4DC870;  --ok-bg: #071208;
--warn: #E8A44A;  --warn-bg: #1A1000;
--danger: #E06060;  --danger-bg: #1A0808;
}
```

Cycle: `light → dark → system → light`. `system` reads `prefers-color-scheme`.

---

## Step 4 — Topbar (`box-top`)

Layout: `[ page-num ] | [ A− ] [ A+ ] | [ orient ] [ theme ] [ print ] [ share ] [ reset ]`

```html
<div class="box-top" id="boxTop">
<div class="top-left"><span id="pageNum"></span></div>
<div class="top-center">
<button id="btnMinus">A-</button>
<button id="btnPlus">A+</button>
</div>
<div class="top-right">
<button id="orientBtn"></button>
<button id="themeBtn"></button>
<button id="printBtn"><i class="fa-solid fa-fw fa-print"></i></button>
<button id="shareBtn"><i class="fa-solid fa-fw fa-share"></i></button>
<button id="resetBtn"><i class="fa-solid fa-fw fa-arrow-rotate-left"></i></button>
</div>
</div>
```

FA icons: orient portrait=`fa-up-down` / landscape=`fa-left-right`; theme light=`fa-sun` / dark=`fa-moon` / system=`fa-cloud`.
A-/A+ range: 3–69px, step 3px; portrait default 21px, landscape 18px.
Share button: hidden (`display:none`) when `!navigator.share`.

**Page indicator:**
- Single-card Set: `01/N`
- Multi-card Set: `01/N · Y/M` where Y = curCard[curSet]+1, M = CPR[curSet]

---

## Step 5 — Directional nav buttons

Four edge-positioned buttons replace the old bottom dots bar.

```html
<button class="nav-btn nav-left"  id="navLeft"><i class="fa-solid fa-fw fa-chevron-left"></i></button>
<button class="nav-btn nav-right" id="navRight"><i class="fa-solid fa-fw fa-chevron-right"></i></button>
<button class="nav-btn nav-up"   id="navUp"><i class="fa-solid fa-fw fa-chevron-up"></i></button>
<button class="nav-btn nav-down" id="navDown"><i class="fa-solid fa-fw fa-chevron-down"></i></button>
```

Disabled states:
- `navLeft`: `curSet === 0`
- `navRight`: `curSet === SETS - 1`
- `navUp`: `curCard[curSet] === 0`
- `navDown`: `CPR[curSet] === 1` OR `curCard[curSet] === CPR[curSet] - 1`

---

## Step 6 — JavaScript (vanilla)

localStorage keys: `{CODENAME}.set`, `.cards` (JSON array), `.font`, `.theme`, `.orient`.
URL params: `?set=&cards=&font=&theme=&orient=` kept in sync via `history.pushState`.
Boot order: URL params → localStorage → hardcoded defaults.

```js
(function () {
const CODENAME = '{codename}';
const SETS = N;
const CPR = [c0, c1];
const FONT_PORTRAIT = 21;
const FONT_LANDSCAPE = 18;
const MIN_FONT = 3, MAX_FONT = 69, STEP = 3;
const THEMES = ['light', 'dark', 'system'];
const mq = window.matchMedia('(prefers-color-scheme: dark)');
const sp = new URLSearchParams(location.search);
const ls = k => localStorage.getItem(CODENAME + '.' + k);
let curSet = parseInt(sp.get('set') ?? ls('set') ?? 0, 10);
curSet = Math.max(0, Math.min(SETS - 1, curSet));
let curCard;
try { curCard = JSON.parse(sp.get('cards') ?? ls('cards') ?? 'null'); } catch(_) { curCard = null; }
if (!Array.isArray(curCard) || curCard.length !== SETS) curCard = new Array(SETS).fill(0);
curCard = curCard.map((v, s) => Math.max(0, Math.min(CPR[s] - 1, v)));
let baseFontPx = parseInt(sp.get('font') ?? ls('font') ?? FONT_PORTRAIT, 10);
let ti = THEMES.indexOf(sp.get('theme') ?? ls('theme') ?? 'light');
if (ti < 0) ti = 0;
const initOrient = sp.get('orient') ?? ls('orient') ?? 'portrait';
function getOrient() { return document.documentElement.dataset.orient || 'portrait'; }
function persist() {
localStorage.setItem(CODENAME + '.set', curSet);
localStorage.setItem(CODENAME + '.cards', JSON.stringify(curCard));
localStorage.setItem(CODENAME + '.font', baseFontPx);
localStorage.setItem(CODENAME + '.theme', THEMES[ti]);
localStorage.setItem(CODENAME + '.orient', getOrient());
const p = new URLSearchParams({ set: curSet, cards: JSON.stringify(curCard), font: baseFontPx, theme: THEMES[ti], orient: getOrient() });
history.pushState(null, '', '?' + p.toString());
}
function updatePageNum() {
const s = String(curSet + 1).padStart(2, '0') + '/' + String(SETS).padStart(2, '0');
const c = CPR[curSet] > 1 ? ' · ' + (curCard[curSet] + 1) + '/' + CPR[curSet] : '';
pageNum.textContent = s + c;
}
function updateNavState() {
updatePageNum();
navLeft.disabled = curSet === 0;
navRight.disabled = curSet === SETS - 1;
navUp.disabled = curCard[curSet] === 0;
navDown.disabled = CPR[curSet] === 1 || curCard[curSet] === CPR[curSet] - 1;
}
function goSet(i) {
curSet = Math.max(0, Math.min(SETS - 1, i));
setTrack.style.transform = 'translateX(' + (-(curSet / SETS) * 100) + '%)';
updateNavState();
persist();
}
function goCard(s, i) {
curCard[s] = Math.max(0, Math.min(CPR[s] - 1, i));
const ct = document.getElementById('ct' + s);
ct.style.transform = 'translateY(' + (-(curCard[s] / CPR[s]) * 100) + '%)';
updateNavState();
persist();
}
function doNavLeft()  { goSet(curSet - 1); }
function doNavRight() { goSet(curSet + 1); }
function doNavUp()    { goCard(curSet, curCard[curSet] - 1); }
function doNavDown()  { if (CPR[curSet] > 1) goCard(curSet, curCard[curSet] + 1); }
function toggleExpand() {
if (CPR[curSet] > 1) { curCard[curSet] === 0 ? doNavDown() : goCard(curSet, 0); }
}
function setFont(px) {
baseFontPx = Math.max(MIN_FONT, Math.min(MAX_FONT, px));
box.style.setProperty('--card-font', baseFontPx + 'px');
btnMinus.disabled = baseFontPx <= MIN_FONT;
btnPlus.disabled = baseFontPx >= MAX_FONT;
persist();
}
function applyTheme(t) {
document.documentElement.dataset.theme = t === 'system' ? (mq.matches ? 'dark' : 'light') : t;
const icons = { light: 'fa-sun', dark: 'fa-moon', system: 'fa-cloud' };
themeBtn.innerHTML = '<i class="fa-solid fa-fw ' + icons[t] + '"></i>';
}
function setOrient(o) {
document.documentElement.dataset.orient = o;
orientBtn.innerHTML = o === 'portrait'
? '<i class="fa-solid fa-fw fa-up-down"></i>'
: '<i class="fa-solid fa-fw fa-left-right"></i>';
}
setTrack.style.width = (SETS * 100) + 'vw';
for (let s = 0; s < SETS; s++) {
const ct = document.getElementById('ct' + s);
ct.style.height = (CPR[s] * 100) + 'dvh';
}
setOrient(initOrient);
setFont(baseFontPx);
applyTheme(THEMES[ti]);
goSet(curSet);
for (let s = 0; s < SETS; s++) { if (curCard[s] > 0) goCard(s, curCard[s]); }
updateNavState();
btnMinus.addEventListener('click', () => setFont(baseFontPx - STEP));
btnPlus.addEventListener('click',  () => setFont(baseFontPx + STEP));
orientBtn.addEventListener('click', () => {
const next = getOrient() === 'portrait' ? 'landscape' : 'portrait';
setOrient(next);
setFont(next === 'portrait' ? FONT_PORTRAIT : FONT_LANDSCAPE);
persist();
});
themeBtn.addEventListener('click', () => { ti = (ti + 1) % THEMES.length; applyTheme(THEMES[ti]); persist(); });
mq.addEventListener('change', () => { if (THEMES[ti] === 'system') applyTheme('system'); });
if (!navigator.share) { shareBtn.style.display = 'none'; }
else { shareBtn.addEventListener('click', () => navigator.share({ title: document.title, url: location.href })); }
resetBtn.addEventListener('click', () => {
['set', 'cards', 'font', 'theme', 'orient'].forEach(k => localStorage.removeItem(CODENAME + '.' + k));
history.replaceState(null, '', location.pathname);
location.reload();
});
printBtn.addEventListener('click', slidesToPrintPDF);
navLeft.addEventListener('click', doNavLeft);
navRight.addEventListener('click', doNavRight);
navUp.addEventListener('click', doNavUp);
navDown.addEventListener('click', doNavDown);
document.querySelectorAll('.cover-card').forEach(cc => {
const s = parseInt(cc.closest('.set').dataset.set);
cc.addEventListener('click', e => {
if (s === curSet && curCard[s] === 0 && !e.target.closest('button,a')) doNavDown();
});
});
let tx = 0, ty = 0;
box.addEventListener('touchstart', e => { tx = e.touches[0].clientX; ty = e.touches[0].clientY; }, { passive: true });
box.addEventListener('touchend', e => {
const dx = e.changedTouches[0].clientX - tx;
const dy = e.changedTouches[0].clientY - ty;
if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) > 36) {
dx < 0 ? doNavRight() : doNavLeft();
} else if (Math.abs(dy) > Math.abs(dx) && Math.abs(dy) > 36) {
dy < 0 ? doNavDown() : doNavUp();
}
}, { passive: true });
document.addEventListener('keydown', e => {
if (e.key === 'ArrowLeft')  { e.preventDefault(); doNavLeft(); }
if (e.key === 'ArrowRight') { e.preventDefault(); doNavRight(); }
if (e.key === 'ArrowUp')    { e.preventDefault(); doNavUp(); }
if (e.key === 'ArrowDown')  { e.preventDefault(); doNavDown(); }
if (e.key === ' ')          { e.preventDefault(); toggleExpand(); }
});
})();
```

---

## Step 6.5 — `slidesToPrintPDF()` (print / Save-as-PDF)

Iterates every `.card-face` across all Sets and Cards in DOM order.

| Mode | @page size | px ref |
|---|---|---|
| Portrait | 210mm 373mm | 794 × 1417 px |
| Landscape | 297mm 167mm | 1122 × 630 px |

```js
function slidesToPrintPDF() {
const html = document.documentElement;
const orient = getOrient();
const isLandscape = orient === 'landscape';
const pageW = isLandscape ? '297mm' : '210mm';
const pageH = isLandscape ? '167mm' : '373mm';
const pageWpx = isLandscape ? 1122 : 794;
const pageHpx = isLandscape ? 630 : 1417;
const firstFace = document.querySelector('.card-face');
const faceRect = firstFace.getBoundingClientRect();
const faceFontPx = parseFloat(getComputedStyle(firstFace).fontSize);
const scale = Math.min(pageWpx / faceRect.width, pageHpx / faceRect.height);
const printFontPx = Math.round(faceFontPx * scale);
const cs = getComputedStyle(html);
const cssVarNames = [
'--bg','--bg-outer','--text','--text-muted','--border',
'--surface','--surface-2','--dot-on','--dot-off',
'--yes','--no','--bar','--bar-bg','--shadow',
'--accent','--accent-bg','--ok','--ok-bg','--warn','--warn-bg','--danger','--danger-bg'
];
const rootVars = cssVarNames.map(v => v + ':' + cs.getPropertyValue(v)).join(';');
let sheets = '';
for (const s of document.styleSheets) {
try { sheets += [...s.cssRules].map(r => r.cssText).join(''); } catch (_) {}
}
const googleFontsHref = [...document.querySelectorAll('link[rel=stylesheet]')]
.map(l => l.href).find(h => h.includes('fonts.googleapis.com')) || '';
const overrides = '@page{size:' + pageW + ' ' + pageH + ';margin:0}' +
'html,body{display:block!important;margin:0!important;padding:0!important}' +
'.print-page{display:block!important;width:' + pageWpx + 'px;height:' + pageHpx + 'px;page-break-after:always;break-after:page}' +
'.print-page:last-child{page-break-after:avoid;break-after:avoid}' +
'.print-page .card-face{flex:none!important;width:' + pageWpx + 'px!important;height:' + pageHpx + 'px!important;font-size:' + printFontPx + 'px!important;display:flex!important;flex-direction:column!important;gap:0.45em!important;overflow:hidden!important}';
const faces = [...document.querySelectorAll('.card-face')];
const pages = faces.map(f => '<div class="print-page">' + f.outerHTML + '</div>').join('');
const pw = window.open('', '_blank');
pw.document.write('<!DOCTYPE html><html><head>'
+ (googleFontsHref ? '<link rel="stylesheet" href="' + googleFontsHref + '">' : '')
+ '<style>:root{' + rootVars + '}' + sheets + overrides + '</style>'
+ '</head><body>' + pages + '</body></html>');
pw.document.close();
pw.focus();
pw.onload = () => pw.print();
}
```

---

## Step 7 — Typography

| Role | Font |
|---|---|
| Card title | Playfair Display (700) |
| Labels / data / mono | DM Mono (400, 500) |
| Body / CJK | Noto Serif TC (300, 400, 600) |

```html
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&family=DM+Mono:wght@400;500&family=Noto+Serif+TC:wght@300;400;600&display=swap" rel="stylesheet">
```

---

## Step 8 — Visualization library

Each card face: `slide-tag` (mono, muted) → `h2.slide-title` (Playfair, 1.82em) → content → optional `note-block`.

**Choose the visualization that matches the data type. See Composition guide at the end of this section.**

---

### 8-A · Table (comparison matrix)
Use for: feature grids, benefit matrices, attribute comparisons.

```css
.mat{width:100%;border-collapse:collapse;}
.mat th,.mat td{padding:0.55em 0.65em;border-bottom:1px solid var(--border);}
.mat th{font-family:'DM Mono',monospace;font-size:0.68em;letter-spacing:0.08em;text-transform:uppercase;color:var(--text-muted);font-weight:500;border-bottom:2px solid var(--border);}
.mat td.has{color:var(--yes);font-weight:500;}
.mat td.no{color:var(--border);}
```

```html
<table class="mat">
<thead><tr><th>Feature</th><th>A</th><th>B</th></tr></thead>
<tbody>
<tr><td>Breakfast</td><td class="has">v</td><td class="no">-</td></tr>
<tr><td>Lounge</td><td class="no">-</td><td class="has">2x/yr</td></tr>
</tbody>
</table>
```

---

### 8-B · Simple bar chart
Use for: single-series value comparisons (scores, percentages).

```css
.bar-track{background:var(--bar-bg);border-radius:2px;height:0.55em;width:100%;}
.bar-fill{height:100%;background:var(--bar);border-radius:2px;}
```

```html
<div style="display:flex;flex-direction:column;gap:0.6em;">
<div>
<div style="display:flex;justify-content:space-between;margin-bottom:0.2em;"><span>Category A</span><span>72%</span></div>
<div class="bar-track"><div class="bar-fill" style="width:72%"></div></div>
</div>
</div>
```

---

### 8-C · Segment bar (multi-series strength indicator) NEW v1.7.0
Use for: service coverage strength, multi-entity ratings on a 3–5 segment scale.
Solid segment = strong; half-opacity = partial; empty = unsupported.
Set brand colour via `--cell` on `.rate-bar`.

```css
.seg-table{width:100%;border-collapse:collapse;}
.seg-table th,.seg-table td{padding:0.55em 0.65em;border-bottom:1px solid var(--border);}
.seg-table th{font-family:'DM Mono',monospace;font-size:0.68em;letter-spacing:0.08em;text-transform:uppercase;color:var(--text-muted);font-weight:500;border-bottom:2px solid var(--border);}
.rate-bar{display:inline-flex;align-items:center;gap:2px;}
.rate-bar .seg{width:0.85em;height:1.3em;background:var(--surface-2);border-radius:1px;}
.rate-bar .seg.on{background:var(--cell,var(--bar));}
.rate-bar .seg.mid{background:var(--cell,var(--bar));opacity:0.45;}
```

```html
<table class="seg-table">
<thead><tr><th>Service</th><th>A</th><th>B</th><th>Notes</th></tr></thead>
<tbody>
<tr>
<td>Hotels</td>
<td><span class="rate-bar" style="--cell:#B83030;"><span class="seg on"></span><span class="seg on"></span><span class="seg on"></span><span class="seg on"></span><span class="seg"></span></span></td>
<td><span class="rate-bar" style="--cell:#0A2766;"><span class="seg on"></span><span class="seg on"></span><span class="seg on"></span><span class="seg on"></span><span class="seg on"></span></span></td>
<td>B has wider coverage</td>
</tr>
<tr>
<td>Trains</td>
<td><span class="rate-bar"><span class="seg"></span><span class="seg"></span><span class="seg"></span><span class="seg"></span><span class="seg"></span></span></td>
<td><span class="rate-bar" style="--cell:#0A2766;"><span class="seg on"></span><span class="seg mid"></span><span class="seg"></span><span class="seg"></span><span class="seg"></span></span></td>
<td>B partial support</td>
</tr>
</tbody>
</table>
```

---

### 8-D · Yes/No grid
Use for: binary feature availability.

```css
.yn-row{display:grid;grid-template-columns:1fr repeat(2,5em);align-items:center;padding:0.5em 0;border-bottom:1px solid var(--border);}
.yn-row:last-child{border-bottom:none;}
.yes{color:var(--yes);font-weight:600;}
.no{color:var(--border);}
```

---

### 8-E · Pick list
Use for: ordered/unordered benefit lists.

```css
.pick-item{display:flex;gap:0.5em;padding:0.45em 0;border-bottom:1px solid var(--border);}
.pick-item:last-child{border-bottom:none;}
```

```html
<div style="display:flex;flex-direction:column;">
<div class="pick-item"><span style="color:var(--text-muted);">-&gt;</span><span>Free breakfast at select hotels</span></div>
<div class="pick-item"><span style="color:var(--text-muted);">-&gt;</span><span>Priority customer support</span></div>
</div>
```

---

### 8-F · Tier ladder NEW v1.7.0
Use for: loyalty tiers, level progressions, qualification thresholds.

```css
.tier-row{display:flex;align-items:center;gap:0.7em;padding:0.5em 0;border-bottom:1px solid var(--border);}
.tier-row:last-child{border-bottom:none;}
.tier-badge{font-family:'DM Mono',monospace;font-size:0.65em;font-weight:500;letter-spacing:0.06em;text-transform:uppercase;padding:0.2em 0.55em;border-radius:2px;white-space:nowrap;}
.tier-req{font-family:'DM Mono',monospace;font-size:0.72em;color:var(--text-muted);margin-left:auto;white-space:nowrap;}
```

```html
<div style="display:flex;flex-direction:column;">
<div class="tier-row">
<span class="tier-badge" style="background:#C9A84C22;color:#C9A84C;">Gold</span>
<span style="font-size:0.88em;">Lounge access + upgrade priority</span>
<span class="tier-req">40 nights</span>
</div>
</div>
```

---

### 8-G · Stat strip NEW v1.7.0
Use for: 2–4 hero stats side by side.

```css
.stat-strip{display:flex;gap:0.5em;flex:1;}
.stat-cell{flex:1;background:var(--surface);padding:0.6em 0.5em;display:flex;flex-direction:column;gap:0.25em;}
.stat-val{font-family:'DM Mono',monospace;font-size:1.6em;font-weight:500;line-height:1;}
.stat-lbl{font-family:'DM Mono',monospace;font-size:0.6em;color:var(--text-muted);text-transform:uppercase;letter-spacing:0.08em;}
```

```html
<div class="stat-strip">
<div class="stat-cell">
<span class="stat-val">142</span>
<span class="stat-lbl">Hotels</span>
</div>
<div class="stat-cell">
<span class="stat-val">5</span>
<span class="stat-lbl">Tiers</span>
</div>
</div>
```

---

### 8-H · KPI Insight card NEW v1.7.0
Use for: one dramatic metric per entity, with delta and label.

```css
.kpi-card{background:var(--surface);padding:0.8em 1em;display:flex;flex-direction:column;gap:0.3em;flex:1;}
.kpi-num{font-family:'DM Mono',monospace;font-size:2.4em;font-weight:500;line-height:1;}
.kpi-delta{font-family:'DM Mono',monospace;font-size:0.72em;}
.kpi-delta.up{color:var(--yes);}
.kpi-delta.dn{color:var(--no);}
.kpi-lbl{font-size:0.78em;color:var(--text-muted);}
```

```html
<div style="display:flex;gap:0.5em;flex:1;">
<div class="kpi-card">
<div class="kpi-num">87%</div>
<div class="kpi-delta up">↑ 4pp vs last quarter</div>
<div class="kpi-lbl">Member satisfaction</div>
</div>
</div>
```

---

### 8-I · Quote callout NEW v1.7.0
Use for: user quotes, evidence snippets, warnings, confirmations.
Variants: `ok` (green), `warn` (amber), `danger` (red).

```css
.quote-box{border-left:3px solid var(--qc,var(--border));background:var(--qbg,var(--surface));padding:0.6em 0.8em;font-size:0.85em;line-height:1.55;}
.quote-box.ok{--qc:var(--ok);--qbg:var(--ok-bg);}
.quote-box.warn{--qc:var(--warn);--qbg:var(--warn-bg);}
.quote-box.danger{--qc:var(--danger);--qbg:var(--danger-bg);}
.quote-attr{font-family:'DM Mono',monospace;font-size:0.75em;color:var(--text-muted);margin-top:0.4em;}
```

```html
<div class="quote-box warn">
Points expiry is the #1 complaint from surveyed members.
<div class="quote-attr">— 2024 loyalty survey, N=1,200</div>
</div>
```

---

### 8-J · SVG Radar chart NEW v1.7.0
Use for: multi-dimension entity scoring (3–8 axes).

**Step 1 — Landscape slide fix**

In landscape, the `.card-face` holding a radar must stretch to fill its slot:

```css
[data-orient="landscape"] .card-face {
align-self: stretch;
margin-top: 0;
margin-bottom: 0;
}
```

**Step 2 — Radar wrap**

```css
.radar-wrap{display:flex;flex:1;min-height:0;min-width:0;}
[data-orient="portrait"] .radar-wrap{flex-direction:column;}
[data-orient="landscape"] .radar-wrap{flex-direction:row;align-items:stretch;}
```

**Step 3 — SVG box sizing (critical — prevents overflow)**

```css
.radar-svg-box{flex:1;min-height:0;min-width:0;width:100%;overflow:hidden;}
[data-orient="landscape"] .radar-svg-box{flex:0 0 auto;height:100%;width:auto;aspect-ratio:620/530;max-width:58%;overflow:hidden;}
[data-orient="landscape"] .radar-svg-box svg{width:100%;height:100%;}
```

`aspect-ratio` values must match your SVG viewBox W/H (e.g. `viewBox="-310 -265 620 530"` → `aspect-ratio:620/530`).
`max-width:58%` ensures the legend column always has room.

**Step 4 — SVG markup**

Always include `preserveAspectRatio="xMidYMid meet"` so the SVG letterboxes without clipping.

ViewBox formula — include padding beyond outer ring for axis labels:
```
R = outer ring radius (e.g. 220)
pad = ~30px for label text
viewBox = "-(R+pad) -(R+pad) (R+pad)*2 (R+pad)*2"
```
Adjust height slightly if labels only appear top/bottom (non-symmetric axes).

```html
<svg viewBox="-310 -265 620 530"
preserveAspectRatio="xMidYMid meet"
xmlns="http://www.w3.org/2000/svg"
font-family="DM Mono,monospace" font-size="12">
```

**Grid rings (one per score level):**
```html
<g fill="none" stroke="var(--border)" stroke-width="1">
<circle r="44"></circle><circle r="88"></circle>
<circle r="132"></circle><circle r="176"></circle>
<circle r="220" stroke-width="1.5"></circle>
</g>
```

**Axes (one line per axis from center):**
```html
<g stroke="var(--border)" stroke-width="1">
<line x1="0" y1="0" x2="0" y2="-220"></line>
</g>
```

**Data polygon + dots per entity:**
```html
<polygon fill="#0A2766" fill-opacity="0.12" stroke="#0A2766" stroke-width="2.2"
points="x1,y1 x2,y2 ..."></polygon>
<g fill="#0A2766">
<circle cx="x1" cy="y1" r="3.5"></circle>
</g>
```

**Axis labels** — text-anchor: `middle` top/bottom; `start` right-side; `end` left-side:
```html
<g fill="var(--text)" font-size="12" font-weight="500">
<text x="0" y="-238" text-anchor="middle">Axis name</text>
<text x="0" y="-252" font-size="10" fill="var(--text-muted)" text-anchor="middle">sublabel</text>
</g>
```

**Geometry formulas (N axes, max score S, outer radius R=220):**
```
angle[i] = -90deg + (360/N)*i  → radians for Math.cos/sin
point[i].x = cos(angle[i]) * score[i] * (R/S)
point[i].y = sin(angle[i]) * score[i] * (R/S)
grid ring radius for score k = k*(R/S)   e.g. S=5,R=220 → step=44
label position = cos(angle[i])*(R+pad), sin(angle[i])*(R+pad)
```

Pre-computed unit vectors for 6-axis hexagon (R=220, S=5):

| Axis | Angle | x at score 5 | y at score 5 |
|---|---|---|---|
| 0 top | -90° | 0 | -220 |
| 1 top-right | -30° | +190.5 | -110 |
| 2 bot-right | +30° | +190.5 | +110 |
| 3 bottom | +90° | 0 | +220 |
| 4 bot-left | +150° | -190.5 | +110 |
| 5 top-left | +210° | -190.5 | -110 |

Scale each by `score/S` for actual data.

**Legend markup:**
```css
.radar-legend{display:flex;flex-direction:column;gap:0.7em;}
.radar-legend-row{display:flex;align-items:center;gap:0.6em;font-size:0.85em;}
.radar-swatch{width:1.2em;height:0.18em;}
```

```html
<div class="radar-legend-box">
<div class="radar-legend">
<div class="radar-legend-row">
<span class="radar-swatch" style="background:#0A2766;"></span>
<strong style="color:#0A2766;">Entity A</strong> 24/35
</div>
</div>
</div>
```

**Full HTML structure:**
```html
<div class="radar-wrap">
<div class="radar-svg-box">
<svg viewBox="-310 -265 620 530" preserveAspectRatio="xMidYMid meet" xmlns="http://www.w3.org/2000/svg">
...rings, axes, polygons, labels...
</svg>
</div>
<div class="radar-legend-box">
...legend rows...
</div>
</div>
```

---

### 8-K · Brand accent card NEW v1.7.0
Use for: platform / product overviews with a coloured accent bar. Works in 2-col or 3-col grids.

```css
.brand-card{border:1px solid var(--border);background:var(--bg);padding:1em;display:flex;flex-direction:column;gap:0.55em;position:relative;}
.brand-card::before{content:'';position:absolute;top:0;left:0;right:0;height:3px;background:var(--card-accent,var(--accent));}
```

```html
<div style="display:grid;grid-template-columns:repeat(3,1fr);gap:0.6em;flex:1;">
<div class="brand-card" style="--card-accent:#B83030;">
<div style="font-family:'DM Mono',monospace;font-size:0.65em;color:var(--text-muted);text-transform:uppercase;letter-spacing:0.1em;">Platform</div>
<div style="font-weight:700;font-size:1.3em;">Agoda</div>
<div style="font-size:0.8em;color:var(--text-muted);">5 tiers, 24-month rolling</div>
<div style="font-size:0.78em;line-height:1.5;margin-top:auto;padding-top:0.4em;border-top:1px solid var(--border);">
Top tier: Diamond &nbsp;x&nbsp; Validity: 6 mo
</div>
</div>
</div>
```

---

### 8-L · Caption / note box
Use for: footnotes, methodology, caveats.

```css
.note-block{border:1px solid var(--border);border-radius:2px;padding:0.4em 0.6em;font-family:'DM Mono',monospace;font-size:0.68em;color:var(--text-muted);line-height:1.5;}
```

---

### Composition guide

| Content type | Best pattern |
|---|---|
| Multi-entity feature comparison | 8-A Table |
| Binary feature availability | 8-D Yes/No grid |
| Service coverage strength (graded) | 8-C Segment bar |
| 2-4 hero stats side by side | 8-G Stat strip |
| One dramatic metric per entity | 8-H KPI Insight card |
| Ranked tier / level progressions | 8-F Tier ladder |
| Multi-dimension entity scoring | 8-J SVG Radar chart |
| Platform / product overviews | 8-K Brand accent card |
| User quotes, warnings, evidence | 8-I Quote callout |
| Ordered benefit or step list | 8-E Pick list |
| Single-series value ranking | 8-B Simple bar chart |
| Footnotes / caveats | 8-L Caption box |

Landscape card faces are wider and shorter — prefer horizontal layouts (columns, wide tables, radar+legend) over tall stacked content.

---

## Design checklist

- [ ] Codename collected; output file `{codename}.html`
- [ ] Font Awesome kit in `<head>`
- [ ] `viewport-fit=cover` in meta viewport
- [ ] `body` and `.box` both `width:100vw; height:100dvh; overflow:hidden`
- [ ] `.box` has `--top-h: 40px`; font-size not set on box (topbar uses fixed 14px)
- [ ] `.box-top` `position:absolute; z-index:200; height:var(--top-h)`
- [ ] `.set-track` width set by JS on init: `SETS * 100vw`; height `100dvh`; horizontal transition
- [ ] `.set` `flex: 0 0 100vw; height:100dvh; overflow:hidden; display:flex; align-items:flex-start; justify-content:center`
- [ ] `.card-track` height set by JS on init: `CPR[s] * 100dvh`; id `ct{s}`; vertical transition
- [ ] `.card` `flex: 0 0 100dvh; width:100vw; display:flex; align-items/justify-content: center`
- [ ] `.card-face` portrait: `width:min(--aw,--ah*9/16); height:min(--ah,--aw*16/9); font-size:var(--card-font,clamp(16px,7vw,28px))`
- [ ] `.card-face` landscape: `width:min(--aw,--ah*16/9); height:min(--ah,--aw*9/16); font-size:var(--card-font,clamp(12px,3.5vw,22px))`
- [ ] `--aw` subtracts `8vw` and horizontal safe-area insets from `100vw`
- [ ] `--ah` subtracts `var(--top-h)`, `8vw`, and vertical safe-area insets from `100dvh`
- [ ] `.cover-card` class only on first `.card-face` of multi-card Sets; `.expand-hint` inside
- [ ] 4 directional nav buttons positioned at box edges; correct disabled states per nav rules
- [ ] `goSet`: `setTrack.style.transform = translateX(-(curSet/SETS)*100%)`
- [ ] `goCard`: `ct.style.transform = translateY(-(curCard[s]/CPR[s])*100%)`
- [ ] `curCard` is Array(SETS); boot validates length and clamps values; persisted as JSON `.cards`
- [ ] Page indicator: `01/N · Y/M` for multi-card Sets; `01/N` for single-card Sets
- [ ] Touch swipe: horizontal threshold → set nav; vertical threshold → card nav; axis with larger delta wins
- [ ] Keyboard: ← → = Sets; ↑ ↓ = Cards; SPACE = toggleExpand
- [ ] Cover card click/tap → `doNavDown()`; guard: `s === curSet && curCard[s] === 0 && !e.target.closest('button,a')`
- [ ] `setFont()` uses `box.style.setProperty('--card-font', px + 'px')`; range 3–69px step 3; portrait default 21px, landscape 18px
- [ ] Right side topbar order: orient · theme · print · share · reset
- [ ] FA icons correct per state
- [ ] Share button hidden when `!navigator.share`
- [ ] `slidesToPrintPDF()` iterates `document.querySelectorAll('.card-face')`; overrides target `.card-face` not `.slide`
- [ ] Reset clears `{codename}.set`, `.cards`, `.font`, `.theme`, `.orient` then reloads
- [ ] Boot order: URL params → localStorage → defaults for all state vars
- [ ] `persist()` called after every state change
- [ ] Theme light/dark/system all defined; semantic vars (`--accent`, `--ok`, `--warn`, `--danger`, `--*-bg`) in both themes
- [ ] No external JS/CSS framework
- [ ] One self-contained `.html` file
- [ ] Zero indentation in output
- [ ] Zero comments in output
- [ ] Visualization pattern(s) from Step 8 chosen to match content type
- [ ] Radar (8-J): landscape `.card-face` has `align-self:stretch; margin-top:0; margin-bottom:0`
- [ ] Radar (8-J): `.radar-wrap` column in portrait, row in landscape
- [ ] Radar (8-J): `.radar-svg-box` portrait width-driven; landscape height-driven with `aspect-ratio` and `max-width:58%`
- [ ] Radar (8-J): SVG has `preserveAspectRatio="xMidYMid meet"`; viewBox includes label padding beyond outer ring