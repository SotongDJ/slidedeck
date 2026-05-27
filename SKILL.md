---
name: mobile-slidedeck
description: >
  Generate a `cards` deck file for the Card Box Viewer.
  Use this skill for the user to present structured content as a cardbox: a viewer-ready JSONL file
  Phase 1: gather codename, lang (mono/bi), orientation, topic.
  Phase 2: generate {codename}_box.cards — UTF-8 JSONL, one JSON object per line, no HTML.
  Orientation hint (portrait default) guides content density.
  Default language is English only. Ask user if bilingual is needed and what the second language is.
  For HTML output, use v2.x of this skill instead.
  Format v2 adds 17 SVG data visualisation: line, area, pie, donut, scatter, histogram,
  stacked-bar, boxplot, heatmap, waterfall, funnel, candlestick, bubble, violin, gantt, treemap, sankey.
  Format v3 adds two-level patterns (layout + pattern) with 6 layouts (title, full, dual, fulldual,
  dual-hybrid, fulldual-hybrid) and 4 new content patterns (text, image, embed, multimedia).
  Max 10 data points per figure; downsample larger datasets via quartile method.
  Trigger on: make slidedeck
---

# Mobile Slidedeck & Card Box Skill

Generates a `.cards` JSONL deck file for the Card Box Viewer. No HTML, CSS, or JS output.

> **HTML output removed in v3.0.0.** To generate `{codename}_desk.html` (Slidedeck) or `{codename}_box.html` (Card Box), use **v2.x** of this skill.

**Output:** `{codename}_box.cards` — UTF-8 JSONL, one JSON object per line; line 1 = metadata, lines 2..N = cards. **Version: 3.3.0**

---

See [CHANGELOG](https://github.com/SotongDJ/slidedeck/blob/main/CHANGELOG) for full version history.

---

## Phase 1 — Input Gathering

Before generating, collect:

| Input | Required | Notes |
|---|---|---|
| **Codename** | Yes | Short identifier; used in the output filename and `localStorage` keying |
| **Content / topic** | Yes | The material to present |
| **Language** | Ask | Default: English only. Ask: "Do you need bilingual output? If so, what is the second language?" |
| **Orientation hint** | Detect | `portrait` (default) or `landscape`. Infer from "widescreen", "desktop", "16:9". Affects layout choices in content schemas (`cols`, stacked vs wide layouts). |
| **Focus topic** | If source is broad | Which aspect(s) of the source to emphasise |
| **Set count preference** | Optional | User may specify N; otherwise infer from content structure |

**Bilingual mode (when user confirms bilingual + specifies L2):**
- Default language is always **English** (L1); L2 is whatever the user specifies.
- Emit two separate files: `{codename}_box.en.cards` (L1) and `{codename}_box.{l2}.cards` (L2).
- In L2 strings, technical proper nouns stay in their original English form; append the formal L2 translation in parentheses: e.g. `編碼器 (Encoder)` for ZH-TW.
- See Step P2-7 for full bilingual rules.

---

## Phase 2 — Generate Card Box Data (`{codename}_box.cards`)

Generate a structured **JSONL** file that the **Card Box Viewer** can render. One JSON object per line; no HTML, CSS, or JS.

**Output:** `{codename}_box.cards` — UTF-8, one JSON object per line, no BOM, no trailing comma.

| Step | Action |
|---|---|
| **P2-1** | Emit metadata as line 1 |
| **P2-2** | Resolve `pattern` + `variant` for each card |
| **P2-3** | Emit each Card as one line, in box reading order (Set 0 Card 0, Set 0 Card 1, …, Set N Card last) |
| **P2-4** | Author `content.*` payload by pattern schema |
| **P2-5** | Build semantic `palette` and use palette keys for all colour references |
| **P2-6** | Emit optional `theme` overrides if customising CSS variables beyond defaults |
| **P2-7** | Handle bilingual decks (one JSONL per language) |
| **P2-8** | Validate against Card Box Viewer schema |

---

### Cardbox Viewer deck structure

The Cardbox Viewer inserts an **Outline set** at viewer index 1 automatically on every load. Never author it in the file.

```
JSONL set 0   →  Viewer set 0  —  Cover    (single cover card)
(auto)        →  Viewer set 1  —  Outline  (viewer-generated, never in file)
JSONL set 1   →  Viewer set 2  —  First content set
JSONL set 2   →  Viewer set 3  —  Second content set
JSONL set N   →  Viewer set N+1
```

**Rules:**
- Set 0 must contain exactly one card: `"pattern":"cover"`.
- **Never emit `"pattern":"toc"`** — the viewer silently skips any card with that pattern and generates the Outline itself.
- Content sets start at `"set":1` in the JSONL.
- Each content set's first card should be `"pattern":"cover"` with `tag`, `title`, and `subline` — these become the Outline entry.

---

### Jump URL

Deep-link to any card using the **viewer** set index (not the JSONL set index):

```
https://card.trth.nl/?jump={viewer-set}_{card}
```

The `jump` parameter is consumed after one load; the URL updates to `?set=…&card=…`.

| JSONL `set` | Viewer set | `?jump=` example |
|---|---|---|
| 0 (Cover) | 0 | `?jump=0_0` |
| — (Outline, auto) | 1 | `?jump=1_0` |
| 1 (first content set) | 2 | `?jump=2_0` |
| 2 | 3 | `?jump=3_0` |
| N | N+1 | `?jump={N+1}_0` |

String `set` keys also work: `?jump=intro_0` targets a set with `"set":"intro"`.

---

### Step P2-1 — Metadata line (line 1)

The first line of the JSONL is a single JSON object marking the deck. **Do not pretty-print** — one line, no internal newlines.

| Field | Required | Notes |
|---|---|---|
| `type` | Yes | Always `"meta"` |
| `v` | Yes | Always `3`. The viewer's edit panel (pattern/layout/variant selectors) requires `v:3`; all v1 and v2 patterns work identically under v3 |
| `title` | Yes | Deck title (matches the cover card title) |
| `subtitle` | If present | From cover-slide subtitle |
| `codename` | Yes | Same value as Phase 1; viewer uses for `localStorage` keying |
| `version` | If present | e.g. `"v2.1.3"` |
| `language` | Yes | `"en"` for monolingual; for bilingual decks see Step P2-7 |
| `palette` | Yes | Semantic name → hex map; see Step P2-5 |
| `theme` | Optional | `{"light":{"--accent":"#…"}, "dark":{…}}` — only if customising beyond viewer defaults |

Example:

```json
{"type":"meta","v":3,"title":"Visualization Library","subtitle":"35 patterns","codename":"vizdemo","version":"v3.2.0","language":"en","palette":{"planA":"#0A2766","planB":"#B83030","tripcom":"#1E7A3C"}}
```

---

### Step P2-2 — Pattern + variant resolution

Map each card to the visualisation library. Every card in a multi-card Set shares the same `pattern` (the cover's pattern); only `variant` differs.

| Visualization type | JSONL `pattern` | Documented variants |
|---|---|---|
| Table | `"table"` | `default` · `transposed` · `sorted` |
| Bar chart | `"bar"` | `default` · `grouped` · `ascending` |
| Segment bar | `"segment"` | `table` · `stacked` · `transposed` |
| Yes/No grid | `"yn-grid"` | `default` · `split` · `sorted` |
| Pick list | `"pick"` | `list` · `grid` · `numbered` |
| Tier ladder | `"tier"` | `vertical` · `horizontal` · `descending` |
| Stat strip | `"stat"` | `strip` · `grid` · `vertical` |
| KPI insight | `"kpi"` | `strip` · `stacked` · `inline` |
| Quote callout | `"quote"` | `default` · `tight` · `severity-first` |
| Radar chart | `"radar"` | `filled` · `outline` · `side-by-side` |
| Brand card | `"brand"` | `grid` · `stacked` · `compact` |
| Note box | `"note"` | `default` · `numbered` · `grid` |
| Cover card (Set 0) | `"cover"` | `default` · `stat-first` · `centered` |
| Checklist | `"checklist"` | `default` · `numbered` · `grid` |
| **Format v2 — data visualisation plots (SVG)** |||
| Line chart | `"line"` | `default` · `smooth` · `stepped` |
| Area chart | `"area"` | `default` · `stacked` · `normalized` |
| Pie chart | `"pie"` | `default` · `exploded` · `half` |
| Donut chart | `"donut"` | `default` · `thin` · `half` |
| Scatter plot | `"scatter"` | `default` · `labeled` · `trend` |
| Histogram | `"histogram"` | `default` · `cumulative` · `density` |
| Stacked bar chart | `"stacked-bar"` | `default` · `normalized` · `horizontal` |
| Box plot | `"boxplot"` | `default` · `horizontal` · `notched` |
| Heatmap | `"heatmap"` | `default` · `annotated` · `clustered` |
| Waterfall chart | `"waterfall"` | `default` · `horizontal` · `colored` |
| Funnel chart | `"funnel"` | `default` · `horizontal` · `proportional` |
| Candlestick chart | `"candlestick"` | `default` · `hollow` · `volume` |
| Bubble chart | `"bubble"` | `default` · `labeled` · `packed` |
| Violin plot | `"violin"` | `default` · `with-box` · `split` |
| Gantt chart | `"gantt"` | `default` · `grouped` · `minimal` |
| Treemap | `"treemap"` | `default` · `flat` · `labeled` |
| Sankey diagram | `"sankey"` | `default` · `vertical` · `colored` |
| **Format v3 — content patterns** |||
| Text (paragraphs) | `"text"` | _(no variants)_ |
| Image (remote) | `"image"` | _(no variants)_ |
| Embed (HTML element) | `"embed"` | _(no variants)_ |
| Multimedia (audio/video) | `"multimedia"` | _(no variants)_ |

> **`"pattern":"toc"` is prohibited.** The viewer silently skips any card with that pattern — the Outline is auto-generated on every load and is never stored in the file.

**Rules:**
- **Card 0 of every Set** is the cover/default — omit `variant` (or use `"default"`).
- **Cards 1+ are variants** — pick the documented variant string that best describes the layout. If no exact match, pick the closest; the viewer falls back to default rendering.
- For Set 0 (the deck cover), `pattern:"cover"`. For all other sets, `pattern` is one of the visualization types from the table above.

---

### Step P2-3 — Card line shape

| Field | Required | Notes |
|---|---|---|
| `set` | Yes | Set index from the box (`0`, `1`, … as integers preferred; strings accepted) |
| `layout` | v3 only | Layout mode: `title` (default), `full`, `dual`, `fulldual`, `dual-hybrid`, `fulldual-hybrid`. Omit for `title`. |
| `pattern` | Yes | One of the pattern keys from P2-2 |
| `variant` | If not default | One of the documented variants for this pattern |
| `tag` | If present | The card's top-left tag (e.g. `"Pattern A · Table"` or `"Variant 1 · Transposed"`) |
| `title` | If present | The card's title text |
| `subline` | If present | The card's `.subline` text |
| `note` | If present | The card's `.note-block` footer text |
| `content` | Yes for non-cover | Pattern-specific payload; see Step P2-4 |
| `left` | v3 hybrid only | `{pattern, content, variant}` for the left column in `dual-hybrid` / `fulldual-hybrid` |
| `right` | v3 hybrid only | `{pattern, content, variant}` for the right column in `dual-hybrid` / `fulldual-hybrid` |

**Round-trip rule:** emit cards in box reading order (Set 0 Card 0 → Set 0 Card 1 → … → Set 0 Card last → Set 1 Card 0 → …). The viewer buckets by `set`, so contiguous integer set keys preserve original positions.

---

### Step P2-4 — Content schemas (per pattern)

Author the values in these shapes. Numbers stay as numbers; never wrap them as strings unless the viewer does so (e.g. `"∞"`, `"99.9"`).

**Inline HTML in text fields.** Strings inside `content.*` fields (`text`, `description`, `notes`, `subline`, `note`, `foot`, `sub`, etc.) accept a small safe subset of inline tags only: `<strong>`, `<em>`, `<br>`, `<sup>`, `<sub>`. **Do not emit `<span>`** in any form, including `<span style="color:var(--yes);">`. The Card Box Viewer renders these strings without applying CSS variables from a stylesheet, so coloured spans surface as literal text. Use leading signs (`+`, `-`) to convey polarity rather than colour, or wrap in `<strong>` for emphasis. Hex/colour intent for non-text elements belongs in `accent` / `color` palette keys, not inline styles.

#### `cover`
```json
{
  "description": "...",
  "stats": [{"val":"12","lbl":"Patterns"}],
  "footer": "..."
}
```

#### `table`
```json
{
  "headers": ["Feature","Free","Pro"],
  "rows": [
    ["Projects","3","∞"]
  ]
}
```
Cells can also be objects: `{"text":"...","class":"has|no|num"}` when the box used explicit styling classes. The viewer auto-classifies dashes (`-`, `−`, `–`, `—`) as `no` and other strings as `has`.

#### `bar`
```json
{
  "max": 100,
  "items": [
    {"label":"Python","value":41,"color":"planA"}
  ]
}
```
For the `grouped` variant, replace `items` with `groups` (each group's `items` follows the same shape as above):
```json
{
  "max": 100,
  "groups": [
    {"label":"Above 20%","items":[]},
    {"label":"Below 20%","items":[]}
  ]
}
```

#### `segment`
```json
{
  "columns": ["Service","Plan A","Plan B"],
  "rows": [
    {"label":"Hotels","ratings":[
      {"on":4,"total":5,"color":"planA"},
      {"on":5,"total":5,"color":"planB"}
    ]}
  ]
}
```
`ratings[].mid` defaults to 0 when omitted.

#### `yn-grid`
```json
{
  "columns": ["Plan A","Plan B"],
  "rows": [
    {"label":"SSO login","values":[true,true]}
  ]
}
```
Values: booleans (preferred) or `"v"` / `"−"`.

#### `pick`
```json
{
  "items": [
    {"text":"Free breakfast at 4★+ properties"}
  ],
  "numbered": false,
  "lastSpan": false
}
```
Set `numbered:true` for the numeric-prefix variant. `lastSpan:true` only for `grid` variant when the item count is odd.

#### `tier`
```json
{
  "tiers": [
    {"name":"Bronze","color":"bronze","desc":"Earn-only","req":"0 nights"}
  ]
}
```
`color` may be a built-in tier name (`bronze`, `silver`, `gold`, `platinum`, `diamond`, `black` — recognised by the viewer) or a palette key.

#### `stat`
```json
{
  "items": [{"val":"142","lbl":"Hotels"}],
  "description": "..."
}
```

#### `kpi`
```json
{
  "items": [
    {
      "num":"87%",
      "delta":"↑ 4pp vs last quarter",
      "direction":"up",
      "label":"Member satisfaction"
    }
  ]
}
```
`direction`: `"up"` (green) or `"dn"` (red).

#### `quote`
```json
{
  "items": [
    {"text":"...","attr":"— Q3 retention review","tone":"ok"}
  ]
}
```
`tone`: `"ok"` | `"warn"` | `"danger"`.

#### `radar`
```json
{
  "axes": ["Speed","Cost","Coverage","Support","Quality","Flexibility"],
  "max": 5,
  "series": [
    {"name":"Plan A","values":[4,4,3,3,4,4],"total":"22/30","color":"planA"},
    {"name":"Plan B","values":[3,4,5,4,1,2],"total":"21/30","color":"planB"}
  ]
}
```
`series[].values.length` **must equal** `axes.length`.

#### `brand`
```json
{
  "columns": 3,
  "cards": [
    {
      "meta":"Platform",
      "name":"Agoda",
      "sub":"5 tiers · 24-mo",
      "foot":"Top: Diamond\nValidity: 6 mo",
      "accent":"agoda"
    }
  ]
}
```
For the `compact` variant, use `subs` array instead of `sub`/`foot`:
```json
{"name":"Agoda","subs":["5 tiers","Diamond","24-mo · 6mo valid"],"accent":"agoda"}
```

#### `note`
```json
{
  "notes": [
    "All figures from Q3 2024 internal data..."
  ],
  "numbered": false,
  "footer": "End of demo · 12 / 12 patterns"
}
```

#### `checklist`
```json
{
  "variant": "default",
  "items": [
    {"text":"Reserve hotels","status":"check","id":"hotels"},
    {"text":"Book flights","id":"flights"},
    {"text":"Old task","status":"strikethrough","id":"task-cancel"}
  ],
  "footer": "Tap to cycle: empty → ✓ → crossed → empty"
}
```

Item fields: `text` (required; `<strong>` and `<em>` allowed), `status` (`"empty"` default / `"check"` / `"strikethrough"`), `id` (required — prevents state misalignment on reorder), `variant` (`"default"` / `"numbered"` / `"grid"`), `footer` (optional instruction note).

File-baked `status` values load only when no `localStorage` record already exists for that item. Omit `status` (or use `"empty"`) for items that start unchecked. Always include `"id"` on every item.

---

### Step P2-4b — Data density rule (all patterns)

Every figure shows **at most 10 data points**. When the source dataset has more than 10 points, downsample to **9 representative points** using quartiles:

| Position | Percentile |
|---|---|
| 1 | Q0 (min, 0%) |
| 2 | P12.5 |
| 3 | Q1 (25%) |
| 4 | P37.5 |
| 5 | Q2 (median, 50%) |
| 6 | P62.5 |
| 7 | Q3 (75%) |
| 8 | P87.5 |
| 9 | Q4 (max, 100%) |

This applies to all data arrays: `series[].values`, `points[]`, `bins[]`, `slices[]`, `stages[]`, `candles[]`, `tasks[]`, `groups[]`, `items[]`, `tiers[]`, `rows[]`, etc. — across **both** format-v1 and format-v2 patterns.

---

### Step P2-4c — Format v2 plot content schemas

All plots are rendered as inline SVG by the viewer. Always set `"v":3` in the metadata line (not `"v":2`) — v3 is a superset that enables the viewer's full editing capabilities.

#### `line`
```json
{
  "labels": ["Jan","Feb","Mar","Apr","May"],
  "series": [
    {"name":"Revenue","values":[10,15,12,18,22],"color":"planA"}
  ],
  "xLabel": "Month",
  "yLabel": "$ (thousands)",
  "max": 25
}
```
`series[].values.length` must equal `labels.length`. `max` and `min` are optional axis overrides.

#### `area`
```json
{
  "labels": ["Jan","Feb","Mar","Apr","May"],
  "series": [
    {"name":"Users","values":[100,250,400,700,950],"color":"planA"}
  ],
  "stacked": false,
  "xLabel": "Week",
  "yLabel": "Users"
}
```
For `stacked` variant, set `"stacked":true`. For `normalized` variant, values are auto-normalized to 100%.

#### `pie`
```json
{
  "slices": [
    {"label":"Desktop","value":55,"color":"planA"},
    {"label":"Mobile","value":35,"color":"planB"},
    {"label":"Tablet","value":10,"color":"planC"}
  ]
}
```
Best limited to 2–5 slices. Labels are rendered inside slices when the slice is ≥5% of total.

#### `donut`
```json
{
  "slices": [
    {"label":"Eng","value":45,"color":"planA"},
    {"label":"Sales","value":30,"color":"planB"}
  ],
  "center": {"value":"$2M","label":"Total"}
}
```
`center` is optional; shown as large text inside the ring.

#### `scatter`
```json
{
  "series": [
    {"name":"Group A","points":[{"x":1,"y":5},{"x":3,"y":8}],"color":"planA"}
  ],
  "xLabel": "Height",
  "yLabel": "Weight"
}
```
For `labeled` variant, add `"label":"..."` to each point. For `trend` variant, a regression line is auto-computed.

#### `histogram`
```json
{
  "bins": [
    {"range":"0–20","count":5},
    {"range":"20–40","count":12},
    {"range":"40–60","count":25}
  ],
  "xLabel": "Score",
  "yLabel": "Frequency"
}
```
For `cumulative` variant, a cumulative line is overlaid.

#### `stacked-bar`
```json
{
  "labels": ["Q1","Q2","Q3","Q4"],
  "series": [
    {"name":"Product A","values":[30,35,40,45],"color":"planA"},
    {"name":"Product B","values":[20,25,15,30],"color":"planB"}
  ],
  "max": 100
}
```
`series[].values.length` must equal `labels.length`. For `normalized` variant, values are auto-normalized to 100%.

#### `boxplot`
```json
{
  "groups": [
    {"label":"Eng","min":60,"q1":75,"median":90,"q3":110,"max":140,"outliers":[45,160]},
    {"label":"Sales","min":40,"q1":55,"median":65,"q3":80,"max":100}
  ]
}
```
`outliers` is optional. All five-number summary fields (`min`, `q1`, `median`, `q3`, `max`) are required.

#### `heatmap`
```json
{
  "xLabels": ["Mon","Tue","Wed","Thu","Fri"],
  "yLabels": ["AM","PM","Eve"],
  "values": [[5,3,8,2,7],[9,4,6,3,5],[2,7,4,8,1]],
  "colorScale": {"low":"#e8e8e0","high":"planA"},
  "annotated": true
}
```
`values` is a 2D array: `values[y][x]`. Dimensions must match `yLabels.length × xLabels.length`. `annotated` shows values in cells.

#### `waterfall`
```json
{
  "items": [
    {"label":"Revenue","value":100,"type":"total"},
    {"label":"COGS","value":-30,"type":"decrease"},
    {"label":"OpEx","value":-25,"type":"decrease"},
    {"label":"Net","value":35,"type":"total"}
  ]
}
```
`type`: `"total"` (absolute bar from zero), `"increase"` (positive delta), `"decrease"` (negative delta). For `colored` variant, increases are green and decreases are red.

#### `funnel`
```json
{
  "stages": [
    {"label":"Visitors","value":1000,"color":"planA"},
    {"label":"Leads","value":600},
    {"label":"Qualified","value":300},
    {"label":"Won","value":100}
  ]
}
```
Stages should be ordered from largest to smallest. For `proportional` variant, widths scale exactly to values.

#### `candlestick`
```json
{
  "candles": [
    {"label":"Mon","open":100,"high":110,"low":95,"close":105},
    {"label":"Tue","open":105,"high":115,"low":100,"close":98}
  ],
  "bullColor": "planA",
  "bearColor": "planB"
}
```
All four OHLC fields required per candle. `bullColor`/`bearColor` default to green/red.

#### `bubble`
```json
{
  "series": [
    {"name":"Products","points":[
      {"x":20,"y":80,"r":30,"label":"A"},
      {"x":50,"y":60,"r":15,"label":"B"}
    ],"color":"planA"}
  ],
  "xLabel": "Price",
  "yLabel": "Quality",
  "rLabel": "Market Share"
}
```
`r` encodes the third variable as bubble radius. For `packed` variant, no axes — bubbles are packed in a circle layout.

#### `violin`
```json
{
  "groups": [
    {"label":"API A","density":[[10,0.02],[20,0.1],[30,0.25],[40,0.15],[50,0.03]],"color":"planA"}
  ]
}
```
`density` is an array of `[value, frequency]` pairs defining the kernel density estimate. For `with-box` variant, add `q1`, `median`, `q3`, `min`, `max` fields to each group.

#### `gantt`
```json
{
  "tasks": [
    {"label":"Design","start":0,"end":3,"color":"planA"},
    {"label":"Dev","start":2,"end":7,"color":"planB"}
  ],
  "labels": ["W1","W2","W3","W4","W5","W6","W7","W8"],
  "milestones": [{"label":"Alpha","at":5}]
}
```
`start`/`end` are numeric positions on the timeline axis. `labels` maps positions to display text. `milestones` are diamond markers.

#### `treemap`
```json
{
  "items": [
    {"label":"System","value":50,"color":"planA","children":[
      {"label":"OS","value":30},{"label":"Drivers","value":20}
    ]},
    {"label":"Apps","value":30,"color":"planB"}
  ]
}
```
`children` is optional for hierarchical layout. For `flat` variant, children are ignored. For `labeled` variant, values are shown in cells.

#### `sankey`
```json
{
  "nodes": ["Organic","Paid","Social","Landing","Signup"],
  "links": [
    {"from":0,"to":3,"value":40,"color":"planA"},
    {"from":1,"to":3,"value":25},
    {"from":3,"to":4,"value":50}
  ]
}
```
`from`/`to` are zero-based indices into `nodes`. `value` determines ribbon width. For `colored` variant, links inherit the source node's colour.

---

### Step P2-4d — Format v3 layout system and content schemas

When using two-level patterns (layout + pattern), set `"v":3` in the metadata line.

#### Layout modes

| Layout | Title header | Columns | Content source |
|---|---|---|---|
| `title` (default) | Yes (tag + title + subline + note) | 1 | `content` |
| `full` | No | 1 | `content` |
| `dual` | Yes | 2 (same pattern) | `content.left`, `content.right` |
| `fulldual` | No | 2 (same pattern) | `content.left`, `content.right` |
| `dual-hybrid` | Yes | 2 (different patterns) | `left.{pattern,content}`, `right.{pattern,content}` |
| `fulldual-hybrid` | No | 2 (different patterns) | `left.{pattern,content}`, `right.{pattern,content}` |

**Dual (same pattern)** — both columns use the card's `pattern` field:
```json
{"set":1,"layout":"dual","pattern":"bar","title":"Comparison","content":{"left":{"max":100,"items":[...]},"right":{"max":100,"items":[...]}}}
```

**Hybrid (different patterns)** — each column specifies its own pattern:
```json
{"set":2,"layout":"dual-hybrid","title":"Mixed","left":{"pattern":"text","content":{"paragraphs":["..."]}},"right":{"pattern":"bar","content":{"max":100,"items":[...]}}}
```

**Full (no title)** — body fills entire card face:
```json
{"set":3,"layout":"full","pattern":"image","content":{"src":"https://example.com/photo.jpg","fit":"cover"}}
```

#### V3 content patterns

##### `text`
```json
{
  "paragraphs": ["First paragraph with <strong>rich</strong> text.", "Second paragraph."]
}
```
Renders stacked `<p>` elements with `richEsc` (safe inline HTML: `<strong>`, `<em>`, `<br>`, `<sup>`, `<sub>`, `<code>`, `<b>`, `<i>`, `<u>`).

##### `image`
```json
{
  "src": "https://example.com/photo.jpg",
  "fit": "contain",
  "alt": "Description",
  "caption": "Photo credit"
}
```
| Field | Required | Notes |
|---|---|---|
| `src` | Yes | Remote image URL |
| `fit` | No | `"contain"` (default) or `"cover"` |
| `alt` | No | Alt text for accessibility |
| `caption` | No | Caption rendered below the image |

##### `embed`
```json
{
  "html": "<iframe src=\"https://example.com/embed\" width=\"100%\" height=\"300\"></iframe>"
}
```
Only allowlisted HTML tags are rendered: `<iframe>`, `<video>`, `<audio>`, `<canvas>`. Safe attributes only: `src`, `width`, `height`, `controls`, `autoplay`, `sandbox`, `allow`, `frameborder`, `style`, `poster`, `preload`, `loop`, `muted`. Iframes automatically receive `sandbox="allow-scripts allow-same-origin"`.

##### `multimedia`
```json
{
  "src": "https://example.com/track.mp3",
  "type": "audio"
}
```
| Field | Required | Notes |
|---|---|---|
| `src` | Yes | Remote audio/video URL |
| `type` | Yes | `"audio"` or `"video"` |
| `poster` | No | Poster image URL (video only) |

---

### Step P2-5 — Palette extraction (semantic names)

Walk every card in the deck and collect distinct hex values. Map each hex to a **semantic name derived from the content**, never a slot index.

**Naming rules** (in priority order):

1. **Tier names** — if the hex matches a built-in tier (`bronze` / `silver` / `gold` / `platinum` / `diamond` / `black`), use that name. The viewer recognises these as a fallback even without a metadata entry, but emitting the explicit entry is preferred for clarity.
2. **Entity names** — when a colour belongs to a named entity in the deck (Plan A, Plan B, Agoda, Booking, Trip.com), use camelCase or kebab-case of the entity: `planA`, `planB`, `agoda`, `booking`, `tripcom`.
3. **Functional names** — for severity/state colours (`ok`, `warn`, `danger`) use the role name; for accent/highlight, use `accent` / `highlight`.
4. **Last resort** — single-letter slots (`a`, `b`, `c`) only if the entity is genuinely unnamed (e.g. abstract demo data).

**Example extraction** (from a real deck):

```
Box hex set:        Entity association:                   Palette key:
  #0A2766           Plan A series                         planA
  #B83030           Plan B series                         planB
  #1E7A3C           Trip.com brand card                   tripcom
  #A06830           Bronze tier badge                     bronze (built-in fallback)
  #C9A84C           Gold tier badge                       gold   (built-in fallback)
```

Resulting metadata fragment:
```json
"palette":{"planA":"#0A2766","planB":"#B83030","tripcom":"#1E7A3C","bronze":"#A06830","gold":"#C9A84C"}
```

**Critical:** inside `content.*.color` and `content.*.accent`, always emit the **palette key**, never the raw hex. This way a future re-skin only edits metadata.

✓ Right: `{"label":"Python","value":41,"color":"planA"}`
✗ Wrong: `{"label":"Python","value":41,"color":"#0A2766"}`

---

### Step P2-6 — Theme overrides (optional)

To customise CSS variables (e.g. change `--accent`, `--ok`, etc. beyond the viewer's defaults), serialise the deltas into `meta.theme`:

```json
"theme":{
  "light":{"--accent":"#1E7A3C"},
  "dark":{"--accent":"#4DC870"}
}
```

Only emit variables that **differ from the viewer's defaults**. Omit `meta.theme` entirely when using default CSS variables.

---

### Step P2-7 — Bilingual decks

The Card Box Viewer renders one language at a time — it does not understand `{en, l2}` objects. For bilingual decks:

- **Default**: serialise the **L1 (English)** strings only. Set `"language":"en"` in metadata.
- **L2 only**: if the user wants the L2 file, serialise the L2 strings and set `"language"` to the BCP-47 code (e.g. `"zh-TW"`, `"ja"`, `"es"`).
- **Both**: emit two JSONL files — `{codename}_box.en.cards` and `{codename}_box.{l2}.cards`. Each is a complete deck on its own.

Do not embed `{en, l2}` objects inside `content.*` — the viewer treats every string as opaque text and would render the JSON shape verbatim.

---

### Step P2-8 — Validation

Before delivering the file, verify it would pass the **Card Box Validator**. Cross-check against this list:

- [ ] Line 1 is a JSON object with `type:"meta"` (or no `pattern` field — the validator is lenient on the marker but emits a warning)
- [ ] Metadata has `title` and `codename`
- [ ] Every line parses as JSON; no trailing comma, no `//` comments, no extra whitespace between lines
- [ ] No `"pattern":"toc"` in any card line — viewer generates the Outline
- [ ] Every card has a `set` (number or string) and `pattern` (one of the documented pattern keys)
- [ ] Every `pattern` resolves to a known variant or the default
- [ ] Every `color` / `accent` reference resolves: hex, palette key, tier name, or `var(...)`
- [ ] Checklist `items[].id` present on every item; `status` ∈ `{empty, check, strikethrough}` (or absent for default `empty`); `variant` ∈ `{default, numbered, grid}` (or absent)
- [ ] Required content keys present per pattern:
  - `table` → `headers`, `rows`
  - `bar` → `items` or `groups`
  - `segment` → `columns`/`headers`, `rows`
  - `yn-grid` → `columns`, `rows`
  - `pick` → `items`
  - `tier` → `tiers`
  - `stat` → `items` or `stats`
  - `kpi` → `items`
  - `quote` → `items`
  - `radar` → `axes`, `series`
  - `brand` → `cards`
  - `note` → `notes`
  - `checklist` → `items`
- [ ] Radar `series[].values.length === axes.length`; `max` is a positive number
- [ ] KPI `direction` ∈ `{up, dn}`; quote `tone` ∈ `{ok, warn, danger}`
- [ ] `meta.v` is always `3` — required for the viewer's edit panel (pattern/layout/variant selectors)
- [ ] Format v2 required keys per plot pattern:
  - `line` / `area` → `labels`, `series`; `series[].values.length === labels.length`
  - `pie` / `donut` → `slices`
  - `scatter` / `bubble` → `series` with `points[]`
  - `histogram` → `bins`
  - `stacked-bar` → `labels`, `series`; `series[].values.length === labels.length`
  - `boxplot` → `groups` with `min`, `q1`, `median`, `q3`, `max`
  - `heatmap` → `xLabels`, `yLabels`, `values`; `values[y].length === xLabels.length`
  - `waterfall` → `items` with `type` ∈ `{total, increase, decrease}`
  - `funnel` → `stages`
  - `candlestick` → `candles` with `open`, `high`, `low`, `close`
  - `violin` → `groups` with `density` (array of `[value, freq]` pairs)
  - `gantt` → `tasks` with `start`, `end`
  - `treemap` → `items` with `value`
  - `sankey` → `nodes`, `links` with `from`, `to`, `value`; indices valid into `nodes`
- [ ] Format v3 layout validation (layout field available because `meta.v` is always `3`):
  - `layout` ∈ `{title, full, dual, fulldual, dual-hybrid, fulldual-hybrid}` (or absent for default `title`)
  - `dual` / `fulldual` → `content.left` and `content.right` both present
  - `dual-hybrid` / `fulldual-hybrid` → `left` and `right` top-level fields with `pattern` and `content`
  - `full` / `fulldual` / `fulldual-hybrid` → `title`, `tag`, `subline`, `note` are ignored (not rendered)
- [ ] Format v3 required content keys:
  - `text` → `paragraphs` (array of strings)
  - `image` → `src` (URL string)
  - `embed` → `html` (string with allowlisted tags only: iframe, video, audio, canvas)
  - `multimedia` → `src` (URL string), `type` ∈ `{audio, video}`
- [ ] No data array exceeds 10 elements (see P2-4b data density rule)
- [ ] Set indices form a contiguous sequence starting at 0 (preferred for round-trip ordering)
- [ ] Card 0 of each Set has no `variant` (or `variant:"default"`)
- [ ] No raw hex inside `content.*.color` / `content.*.accent` — palette keys only

---

### Step P2-9 — File naming

Follow the codename convention:

| Output | File |
|---|---|
| Monolingual | `{codename}_box.cards` |
| Bilingual (both languages) | `{codename}_box.en.cards` and `{codename}_box.{l2}.cards` |

UTF-8 encoding. One JSON object per line. No BOM. The trailing newline at end-of-file is optional (both forms accepted by the viewer).

---

## Phase 2 design checklist

- [ ] First line: object with `type:"meta"`, `title`, `codename`, `palette`, `"v":3` (always v3 for viewer edit compatibility)
- [ ] No `"pattern":"toc"` cards in file (viewer auto-generates Outline at viewer set 1)
- [ ] Cards in box reading order: Set 0 Card 0 → Set 0 Card 1 → … → Set N Card last
- [ ] Card 0 of each Set has no `variant` field (it is the cover/default)
- [ ] Pattern shared across all cards in a Set; only `variant` differs (exception: hybrid layouts use different patterns per column)
- [ ] All hex colours in `content.*.color` and `content.*.accent` rewritten as palette keys
- [ ] Palette names are semantic (`planA`, `agoda`, `bronze`) — never slot indices unless entity is unnamed
- [ ] Variants chosen from the documented list per pattern (see P2-2) or omitted for default
- [ ] Numeric values stay numeric (e.g. `"value":41`); display strings stay strings (e.g. `"display":"41%"`)
- [ ] `series[].values.length === axes.length` for every radar card
- [ ] Bilingual → one language per file; `meta.language` matches the language used; `{en, l2}` objects never embedded in content
- [ ] `meta.theme` emitted only if customising CSS variables beyond viewer defaults
- [ ] No `<span>` tags in any `content.*` text field; allowed inline subset is `<strong>`, `<em>`, `<br>`, `<sup>`, `<sub>` only
- [ ] Checklist items have `"id"` fields; `status` ∈ `{empty, check, strikethrough}`; `variant` ∈ `{default, numbered, grid}`
- [ ] v3 layout: `dual`/`fulldual` cards have `content.left` + `content.right`; `dual-hybrid`/`fulldual-hybrid` have top-level `left` + `right` with `{pattern, content}`
- [ ] v3 patterns: `text` has `paragraphs`; `image` has `src`; `embed` has `html`; `multimedia` has `src` + `type`
- [ ] One self-contained `.cards` file
- [ ] UTF-8 encoding, no BOM, no trailing comma, no comments
- [ ] No data array exceeds 10 items; datasets >10 downsampled via quartile method (P2-4b)
- [ ] Validates against the Card Box Validator with zero errors