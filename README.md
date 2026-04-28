---
name: mobile-slidedeck
description: >
  Create a single-page mobile slide deck web app with vanilla JS and CSS — no external frameworks.
  Default orientation is portrait (9:16); use landscape (16:9) when the user says "landscape", "horizontal slides",
  "widescreen", "desktop slides", or "16:9". Supports a runtime orientation-toggle button so users can switch in-browser.
  Use this skill whenever the user asks to turn content, chapters, or sections into a slide deck, presentation, or swipeable mobile web app.
  Trigger on phrases like "make slides", "mobile slide deck", "swipeable presentation", "one slide per chapter", or any request to present structured content as vertical or horizontal slides.
  Always use this skill when the user asks for a slide-based layout optimised for phones, portrait devices, or widescreen displays.
  Also trigger when the content includes data comparisons, ratings, rankings, scorecards, tier progressions, or any structured data that benefits from charts, radar plots, stat strips, or visual matrices.
---

# Mobile Slide Deck Skill

Converts structured content (chapters / sections) into a polished single-page web app.
Output: one self-contained `.html` file. No external JS or CSS frameworks.

**Version: 1.7.1**

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| **1.7.1** | 2026-04-22 | 8-J Radar chart rewritten with overflow-safe layout: landscape slide gets align-self:stretch + margin:0; radar-wrap switches column/row per orientation; radar-svg-box is height-driven in landscape via height:100% + aspect-ratio + max-width:58%; SVG gains preserveAspectRatio; viewBox formula documented; 6-axis pre-computed vectors table added |
| **1.7.0** | 2026-04-22 | New visualization library in Step 8: SVG radar chart, segment bar, stat strip, KPI insight card, quote callout (ok/warn/danger), tier ladder, brand accent card; new CSS vars --accent, --accent-bg, --ok, --ok-bg, --warn, --warn-bg, --danger, --danger-bg defined in both light and dark; Composition guide table added; print function updated to inline new semantic vars |
| **1.6.0** | 2026-04-11 | Print/PDF button (fa-print); slidesToPrintPDF() opens a new window with all slides as individual print pages, inlines live CSS vars + theme, scales font size proportionally, triggers window.print(); right-side order: orient · theme · print · share · reset |
| **1.5.0** | 2026-04-11 | Codename + Font Awesome kit required; FA icons replace text labels; personalization persisted to localStorage + URL query string via history.pushState(); Share (Web Share API) + Reset buttons |
| **1.4.1** | 2026-04-10 | Fix: restored missing Step 0 heading; removed stale body margin from spacing philosophy and checklist |
| **1.4.0** | 2026-04-10 | Deck always centred via body flex; safe-area insets prevent cropping on notched/rounded devices; viewport-fit=cover; 100dvh replaces 100vh |
| **1.3.0** | 2026-04-10 | Output HTML/CSS/JS: zero indentation, no comments |
| **1.2.0** | 2026-04-10 | Font size buttons moved to topbar centre; range 3-69px step 3; portrait default 21px, landscape 18px |
| **1.1.0** | 2026-04-10 | Added orientation toggle; runtime data-orient attribute; orientation-aware font-size defaults |
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
3. Number of slides.
4. Detect orientation: Portrait (default, 9:16) or Landscape (16:9). Keywords: "landscape", "horizontal", "widescreen", "desktop", "16:9".
5. Always include the orientation-toggle button in the topbar.

---

## Step 1 — HTML skeleton

```html
<!DOCTYPE html>
<html lang="..." data-theme="light" data-orient="portrait">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<title>...</title>
<script src="https://kit.fontawesome.com/{10-digit-id}.js" crossorigin="anonymous"></script>
</head>
<body>
<div class="deck" id="deck">
<div class="deck-top"> ... </div>
<div class="slides-area">
<div class="slides-track" id="track">
</div>
</div>
<div class="deck-bot"> ... </div>
</div>
</body>
```

---

## Step 2 — CSS rules (must follow exactly)

```css
div { padding: 0.1%; margin: 0; }
html, body, h2, table, th, td, hr { margin: 0; padding: 0; }
*, *::before, *::after { box-sizing: border-box; }
body {
min-height: 100dvh;
display: flex;
align-items: center;
justify-content: center;
overflow: hidden;
padding: max(4vw, env(safe-area-inset-top))
         max(4vw, env(safe-area-inset-right))
         max(4vw, env(safe-area-inset-bottom))
         max(4vw, env(safe-area-inset-left));
}
.deck {
--aw: calc(100vw - 8vw - env(safe-area-inset-left) - env(safe-area-inset-right));
--ah: calc(100dvh - 8vw - env(safe-area-inset-top)  - env(safe-area-inset-bottom));
width:  min(var(--aw), calc(var(--ah) * 9 / 16));
height: min(var(--ah), calc(var(--aw) * 16 / 9));
font-size: clamp(16px, 7vw, 28px);
background: var(--bg);
display: flex;
flex-direction: column;
flex-shrink: 0;
position: relative;
overflow: hidden;
}
[data-orient="landscape"] .deck {
width:  min(var(--aw), calc(var(--ah) * 16 / 9));
height: min(var(--ah), calc(var(--aw) * 9 / 16));
font-size: clamp(12px, 3.5vw, 22px);
}
.slides-track {
display: flex;
align-items: center;
width: 400%;
height: 100%;
will-change: transform;
transition: transform 0.55s cubic-bezier(0.77, 0, 0.175, 1);
}
.slide {
flex: 0 0 25%;
height: auto;
margin-top: auto;
margin-bottom: auto;
display: flex;
flex-direction: column;
gap: 0.5em;
overflow: hidden;
}
```

For n slides: `width: n*100%; each .slide flex: 0 0 calc(100%/n)`.

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

## Step 4 — Topbar (`deck-top`)

Layout: `[ page-num ] | [ A− ] [ A+ ] | [ orient ] [ theme ] [ print ] [ share ] [ reset ]`

```html
<div class="deck-top">
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

---

## Step 5 — Bottom bar (`deck-bot`)

`[ ‹ ] [ dots ] [ › ]`. Dots `.on` = current slide. Keyboard: ArrowLeft/Up=prev, ArrowRight/Down=next. Touch swipe >36px.

---

## Step 6 — JavaScript (vanilla)

localStorage keys: `{CODENAME}.page`, `.font`, `.theme`, `.orient`.
URL params: `?page=&font=&theme=&orient=` kept in sync via `history.pushState`.
Boot order: URL params → localStorage → hardcoded defaults.

```js
(function () {
const CODENAME = '{codename}';
const TOTAL = n;
const FONT_PORTRAIT = 21;
const FONT_LANDSCAPE = 18;
const MIN_FONT = 3, MAX_FONT = 69, STEP = 3;
const THEMES = ['light', 'dark', 'system'];
const mq = window.matchMedia('(prefers-color-scheme: dark)');
const sp = new URLSearchParams(location.search);
const ls = k => localStorage.getItem(CODENAME + '.' + k);
let cur = parseInt(sp.get('page') ?? ls('page') ?? 0, 10);
let baseFontPx = parseInt(sp.get('font') ?? ls('font') ?? FONT_PORTRAIT, 10);
let ti = THEMES.indexOf(sp.get('theme') ?? ls('theme') ?? 'light');
if (ti < 0) ti = 0;
const initOrient = sp.get('orient') ?? ls('orient') ?? 'portrait';
function getOrient() { return document.documentElement.dataset.orient || 'portrait'; }
function persist() {
const p = { page: cur, font: baseFontPx, theme: THEMES[ti], orient: getOrient() };
localStorage.setItem(CODENAME + '.page', p.page);
localStorage.setItem(CODENAME + '.font', p.font);
localStorage.setItem(CODENAME + '.theme', p.theme);
localStorage.setItem(CODENAME + '.orient', p.orient);
history.pushState(null, '', '?' + new URLSearchParams(p).toString());
}
function setSlide(i) {
cur = Math.max(0, Math.min(TOTAL - 1, i));
track.style.transform = 'translateX(' + (-cur * (100 / TOTAL)) + '%)';
pageNum.textContent = String(cur + 1).padStart(2, '0') + ' / ' + String(TOTAL).padStart(2, '0');
dots.forEach((d, j) => d.classList.toggle('on', j === cur));
prevBtn.disabled = cur === 0;
nextBtn.disabled = cur === TOTAL - 1;
persist();
}
function setFont(px) {
baseFontPx = Math.max(MIN_FONT, Math.min(MAX_FONT, px));
deck.style.fontSize = baseFontPx + 'px';
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
setOrient(initOrient);
setFont(baseFontPx);
applyTheme(THEMES[ti]);
setSlide(cur);
btnMinus.addEventListener('click', () => setFont(baseFontPx - STEP));
btnPlus.addEventListener('click', () => setFont(baseFontPx + STEP));
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
['.page', '.font', '.theme', '.orient'].forEach(k => localStorage.removeItem(CODENAME + k));
history.replaceState(null, '', location.pathname);
location.reload();
});
prevBtn.addEventListener('click', () => setSlide(cur - 1));
nextBtn.addEventListener('click', () => setSlide(cur + 1));
dots.forEach((d, i) => d.addEventListener('click', () => setSlide(i)));
printBtn.addEventListener('click', slidesToPrintPDF);
let tx = 0;
deck.addEventListener('touchstart', e => { tx = e.touches[0].clientX; }, { passive: true });
deck.addEventListener('touchend', e => {
const dx = e.changedTouches[0].clientX - tx;
if (Math.abs(dx) > 36) setSlide(cur + (dx < 0 ? 1 : -1));
}, { passive: true });
document.addEventListener('keydown', e => {
if (['ArrowRight', 'ArrowDown'].includes(e.key)) setSlide(cur + 1);
if (['ArrowLeft', 'ArrowUp'].includes(e.key)) setSlide(cur - 1);
});
})();
```

---

## Step 6.5 — `slidesToPrintPDF()` (print / Save-as-PDF)

| Mode | @page size | px ref |
|---|---|---|
| Portrait | 210mm 373mm | 794 × 1417 px |
| Landscape | 297mm 167mm | 1122 × 630 px |

Font scale: `scale = Math.min(pageWpx/deckRect.width, pageHpx/deckRect.height); printFontPx = Math.round(deckFontPx * scale)`

```js
function slidesToPrintPDF() {
const html = document.documentElement;
const orient = getOrient();
const isLandscape = orient === 'landscape';
const pageW = isLandscape ? '297mm' : '210mm';
const pageH = isLandscape ? '167mm' : '373mm';
const pageWpx = isLandscape ? 1122 : 794;
const pageHpx = isLandscape ? 630 : 1417;
const deckRect = deck.getBoundingClientRect();
const deckFontPx = parseFloat(getComputedStyle(deck).fontSize);
const scale = Math.min(pageWpx / deckRect.width, pageHpx / deckRect.height);
const printFontPx = Math.round(deckFontPx * scale);
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
'.print-page .slide{flex:none!important;width:' + pageWpx + 'px!important;height:' + pageHpx + 'px!important;font-size:' + printFontPx + 'px!important;display:flex!important;flex-direction:column!important;gap:0.45em!important;overflow:hidden!important}';
const slides = [...document.querySelectorAll('.slide')];
const pages = slides.map(s => '<div class="print-page">' + s.outerHTML + '</div>').join('');
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
| Slide title | Playfair Display (700) |
| Labels / data / mono | DM Mono (400, 500) |
| Body / CJK | Noto Serif TC (300, 400, 600) |

```html
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&family=DM+Mono:wght@400;500&family=Noto+Serif+TC:wght@300;400;600&display=swap" rel="stylesheet">
```

---

## Step 8 — Visualization library

Each slide: `slide-tag` (mono, muted) → `h2.slide-title` (Playfair, 1.82em) → content → optional `note-block`.

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
Each row = one tier. Highlight top tier with brand-colour background inline.
`.perm` = green (lifetime); `.limit` = amber (expiring).

```css
.ladder{display:flex;flex-direction:column;gap:0;}
.ladder-row{display:grid;grid-template-columns:2em 1fr auto;gap:0.6em;align-items:center;padding:0.55em 0;border-bottom:1px solid var(--border);}
.ladder-row:last-child{border-bottom:none;}
.ladder-n{font-family:'DM Mono',monospace;font-size:0.72em;color:var(--text-muted);}
.ladder-l{font-weight:700;font-size:1.05em;}
.ladder-l em{font-family:'DM Mono',monospace;font-size:0.65em;color:var(--text-muted);font-style:normal;display:inline-block;margin-left:0.4em;}
.ladder-v{font-family:'DM Mono',monospace;font-size:0.72em;color:var(--text-muted);text-transform:uppercase;text-align:right;}
.ladder-v.perm{color:var(--yes);}
.ladder-v.limit{color:var(--warn);}
```

```html
<div class="ladder">
<div class="ladder-row">
<span class="ladder-n">01</span>
<span class="ladder-l">Silver <em>sign-up</em></span>
<span class="ladder-v perm">inf</span>
</div>
<div class="ladder-row">
<span class="ladder-n">02</span>
<span class="ladder-l">Gold <em>5 stays / 2yr</em></span>
<span class="ladder-v perm">inf</span>
</div>
<div class="ladder-row" style="background:#0A2766;color:#fff;margin:0 -0.5em;padding-left:0.5em;padding-right:0.5em;">
<span class="ladder-n" style="color:rgba(255,255,255,0.6);">03</span>
<span class="ladder-l" style="color:#fff;">Platinum <em style="color:rgba(255,255,255,0.7);">15 stays / 2yr</em></span>
<span class="ladder-v" style="color:#fff;">inf</span>
</div>
</div>
```

---

### 8-G · Stat strip NEW v1.7.0
Use for: 2–4 hero KPI numbers side by side (usage stats, quick facts).
Each cell: big value + label + optional detail line.

```css
.stat-strip{display:grid;grid-template-columns:repeat(3,1fr);border:1px solid var(--border);background:var(--surface);}
.stat-cell{padding:0.75em 0.85em;border-right:1px solid var(--border);}
.stat-cell:last-child{border-right:none;}
.stat-cell .v{font-family:'Playfair Display',serif;font-weight:700;font-size:2.2em;line-height:1;color:var(--accent);letter-spacing:-0.015em;}
.stat-cell .l{font-family:'DM Mono',monospace;font-size:0.65em;color:var(--text-muted);letter-spacing:0.08em;text-transform:uppercase;margin-top:0.4em;}
.stat-cell .d{font-size:0.75em;color:var(--text-muted);margin-top:0.2em;line-height:1.4;}
```

```html
<div class="stat-strip">
<div class="stat-cell">
<div class="v">0</div>
<div class="l">Breakfasts redeemed</div>
<div class="d">Long-term Platinum users report zero successful claims</div>
</div>
<div class="stat-cell">
<div class="v">6 mo</div>
<div class="l">Renewal window</div>
<div class="d">Must re-qualify or be downgraded</div>
</div>
<div class="stat-cell">
<div class="v">$141</div>
<div class="l">Real benefit / yr</div>
<div class="d">Coins + lounge + eSIM combined</div>
</div>
</div>
```

Use `repeat(2,1fr)` for portrait slides with longer detail text.

---

### 8-H · KPI Insight card NEW v1.7.0
Use for: one dramatic number per entity, laid out 2–3 cards per slide.
Brand colour bar at top via `--brand-accent`; number colour class (`.red`, `.blue`, `.teal`) for branding.

```css
.insight-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:0.6em;flex:1;}
.insight-card{border:1px solid var(--border);padding:0.7em 0.8em;display:flex;flex-direction:column;gap:0.3em;position:relative;}
.insight-card::before{content:'';position:absolute;top:0;left:0;right:0;height:3px;background:var(--brand-accent,var(--accent));}
.insight-plat{font-family:'DM Mono',monospace;font-size:0.65em;color:var(--text-muted);letter-spacing:0.08em;text-transform:uppercase;}
.insight-num{font-family:'Playfair Display',serif;font-weight:700;font-size:3em;line-height:0.95;letter-spacing:-0.02em;color:var(--accent);}
.insight-num.red{color:var(--no);}
.insight-num.blue{color:#0A2766;}
.insight-num.teal{color:#0E6E6E;}
.insight-unit{font-family:'DM Mono',monospace;font-size:0.7em;color:var(--text-muted);}
.insight-claim{font-weight:600;font-size:0.85em;line-height:1.3;}
.insight-claim em{font-style:italic;font-weight:400;color:var(--text-muted);display:block;font-size:0.85em;margin-top:0.15em;}
.insight-src{font-family:'DM Mono',monospace;font-size:0.6em;color:var(--text-muted);line-height:1.4;margin-top:auto;padding-top:0.3em;border-top:1px solid var(--border);}
```

```html
<div class="insight-grid">
<div class="insight-card" style="--brand-accent:#B83030;">
<div class="insight-plat">Platform A</div>
<div class="insight-num red">0</div>
<div class="insight-unit">x free breakfast redeemed</div>
<div class="insight-claim">Benefit on paper only<em>No successful redemption reported</em></div>
<div class="insight-src">Independent review 2026</div>
</div>
<div class="insight-card" style="--brand-accent:#0A2766;">
<div class="insight-plat">Platform B</div>
<div class="insight-num blue">+/-0%</div>
<div class="insight-unit">price delta L2 vs L3</div>
<div class="insight-claim">Higher tier does not mean better price<em>Real value is priority support</em></div>
<div class="insight-src">User A/B test, Reddit</div>
</div>
</div>
```

For portrait: use `repeat(2,1fr)` or `flex-direction:column`.

---

### 8-I · Quote callout (ok / warn / danger) NEW v1.7.0
Use for: user reviews, key warnings, editorial evidence. Stack vertically.
Three variants: `ok` (green), `warn` (amber), `danger` (red).

```css
.qitem{padding:0.6em 0.75em;border-left:3px solid var(--border);background:var(--surface);margin-bottom:0.35em;}
.qitem.ok{border-left-color:var(--accent);background:var(--accent-bg);}
.qitem.warn{border-left-color:var(--warn);background:var(--warn-bg);}
.qitem.danger{border-left-color:var(--no);background:var(--danger-bg);}
.qitem .lbl{font-family:'DM Mono',monospace;font-size:0.62em;letter-spacing:0.1em;text-transform:uppercase;color:var(--text-muted);margin-bottom:0.25em;}
.qitem.ok .lbl{color:var(--accent);}
.qitem.warn .lbl{color:var(--warn);}
.qitem.danger .lbl{color:var(--no);}
.qitem .txt{font-size:0.82em;line-height:1.5;}
.qitem .src{font-family:'DM Mono',monospace;font-size:0.6em;color:var(--text-muted);margin-top:0.25em;}
```

```html
<div class="qitem danger">
<div class="lbl">X Renewal trap</div>
<div class="txt">Must re-qualify every 6 months or be downgraded. Users call it a "punishment design".</div>
<div class="src">Reddit community</div>
</div>
<div class="qitem warn">
<div class="lbl">! Limited coverage</div>
<div class="txt">Discounts only apply to undisclosed partner hotels.</div>
<div class="src">Independent review</div>
</div>
<div class="qitem ok">
<div class="lbl">+ Silver lining</div>
<div class="txt">Base prices in Southeast Asia are competitive and stackable with credit-card codes.</div>
<div class="src">User consensus</div>
</div>
```

---

### 8-J · SVG Radar chart NEW v1.7.0 · overflow-safe v1.7.1
Use for: multi-dimensional comparison of 3+ entities across 5–8 axes.
Pure SVG — zero libraries.

> **The core sizing trap:** A radar SVG has a nearly-square viewBox; landscape slides are wide and short (16:9).
> Sizing by width causes the derived height to overflow the slide. Follow the steps below exactly.

**Step 1 — Fix height inheritance on landscape slides**

The default `.slide` uses `margin-top:auto; margin-bottom:auto` for centering.
`align-self:stretch` silently fails because auto margins consume free space first.
Must zero the margins so stretch actually takes effect:

```css
.slide{height:auto;margin-top:auto;margin-bottom:auto;display:flex;flex-direction:column;gap:0.5em;overflow:hidden;}
[data-orient="landscape"] .slide{align-self:stretch;margin-top:0;margin-bottom:0;}
```

**Step 2 — Wrapper layout: column (portrait) vs row (landscape)**

```css
.radar-wrap{display:flex;flex-direction:column;flex:1;min-height:0;overflow:hidden;align-items:center;}
[data-orient="landscape"] .radar-wrap{flex-direction:row;align-items:stretch;gap:0.5em;}
[data-orient="landscape"] .radar-legend-box{flex:1;min-width:0;display:flex;flex-direction:column;justify-content:center;padding-left:0.5em;border-left:1px solid var(--border);}
```

**Step 3 — SVG container: width-driven (portrait) vs height-driven (landscape)**

Portrait slides are tall — width-driven is fine.
Landscape slides are short — drive from height, derive width via `aspect-ratio`:

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

Landscape slides are wider and shorter — prefer horizontal layouts (columns, wide tables, radar+legend) over tall stacked content.

---

## Design checklist

- [ ] Codename collected; output file `{codename}.html`
- [ ] Font Awesome kit in `<head>`
- [ ] `viewport-fit=cover` in meta viewport
- [ ] `body` flex-centred, `min-height:100dvh`, safe-area padding all four sides
- [ ] `.deck` `--aw`/`--ah` subtract padding + safe-area insets; `flex-shrink:0`
- [ ] `div { padding: 0.1%; margin: 0; }` global
- [ ] `.deck` portrait `clamp(16px,7vw,28px)` / landscape `clamp(12px,3.5vw,22px)`
- [ ] `[data-orient="landscape"]` overrides width/height to 16:9
- [ ] `.slide` height `auto`, `margin-top/bottom: auto`
- [ ] `.slides-track` `align-items: center`
- [ ] A-/A+ centred; range 3–69px step 3; portrait default 21px, landscape 18px
- [ ] Right side order: orient · theme · print · share · reset
- [ ] FA icons correct per state (see Step 4)
- [ ] Share button hidden when `!navigator.share`
- [ ] `slidesToPrintPDF()` orientation-aware; font scaled; NEW semantic vars inlined; stylesheets copied; `pw.print()` in onload
- [ ] Print contains only `.print-page > .slide`
- [ ] Reset clears all `{codename}.*` localStorage keys then reloads
- [ ] Boot: URL params → localStorage → defaults
- [ ] `persist()` called after every state change
- [ ] Theme light/dark/system all defined; NEW semantic vars (`--accent`, `--ok`, `--warn`, `--danger`, `--*-bg`) in both themes
- [ ] Touch swipe, keyboard, dot nav working
- [ ] No external JS/CSS framework
- [ ] One self-contained `.html` file
- [ ] Zero indentation in output
- [ ] Zero comments in output
- [ ] Visualization pattern(s) from Step 8 chosen to match content type
- [ ] Radar (8-J): `[data-orient=landscape] .slide` has `align-self:stretch; margin-top:0; margin-bottom:0`
- [ ] Radar (8-J): `.radar-wrap` is `flex-direction:column` portrait / `flex-direction:row` landscape
- [ ] Radar (8-J): `.radar-svg-box` portrait is width-driven (`width:100%`); landscape is height-driven (`height:100%; width:auto; aspect-ratio:W/H; max-width:58%`)
- [ ] Radar (8-J): SVG has `preserveAspectRatio="xMidYMid meet"`; viewBox includes label padding beyond outer ring
- [ ] Radar (8-J): polygon points computed via trigonometry (angle = -90 + 360/N*i), no library
