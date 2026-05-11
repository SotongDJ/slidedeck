---
name: mobile-slidedeck
description: >
  Generate a .cards JSONL deck file for the Card Box Viewer (card.trth.nl).
  Use this skill whenever the user wants to present structured content as a swipeable deck,
  card box, or viewer-ready data file — even if they don't say "slide deck" explicitly.
  Phase 1: gather codename, language (mono/bilingual), orientation hint, content/topic.
  Phase 2: generate {codename}_box.cards — UTF-8 JSONL, one JSON object per line, no HTML.
  Orientation hint (portrait default, landscape for widescreen/16:9) guides content density.
  Default language is English only. Ask user if bilingual is needed and what the second language is.
  For HTML output (Slidedeck .html or Card Box .html), use v2.x of this skill instead.
  Trigger on: make slides, slidedeck, card box, swipeable presentation, one slide per chapter,
  data comparisons, ratings, rankings, scorecards, tier progressions, charts, radar plots,
  JSONL export, viewer-ready data, cardbox data file, .cards file, card box viewer.
---

# Mobile Slidedeck & Card Box Skill

[![Pipeline Status](https://gitlab.com/djtratoh/card.trth.nl/badges/main/pipeline.svg)](https://gitlab.com/djtratoh/card.trth.nl/-/commits/main)
[![Release Skill](https://github.com/SotongDJ/slidedeck/actions/workflows/release.yml/badge.svg?branch=main)](https://github.com/SotongDJ/slidedeck/actions/workflows/release.yml)

Generates a `.cards` JSONL deck file for the Card Box Viewer. No HTML, CSS, or JS output.

> **HTML output removed in v3.0.0.** To generate `{codename}_desk.html` (Slidedeck) or `{codename}_box.html` (Card Box), use **v2.x** of this skill.

**Output:** `{codename}_box.cards` — UTF-8 JSONL, one JSON object per line; line 1 = metadata, lines 2..N = cards. **Version: 3.0.1**

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
| `title` | Yes | Deck title (matches the cover card title) |
| `subtitle` | If present | From cover-slide subtitle |
| `codename` | Yes | Same value as Phase 1; viewer uses for `localStorage` keying |
| `version` | If present | e.g. `"v2.1.3"` |
| `language` | Yes | `"en"` for monolingual; for bilingual decks see Step P2-7 |
| `palette` | Yes | Semantic name → hex map; see Step P2-5 |
| `theme` | Optional | `{"light":{"--accent":"#…"}, "dark":{…}}` — only if customising beyond viewer defaults |

Example:

```json
{"type":"meta","title":"Visualization Library","subtitle":"Twelve patterns","codename":"vizdemo","version":"v2.1.3","language":"en","palette":{"planA":"#0A2766","planB":"#B83030","tripcom":"#1E7A3C"}}
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
| `pattern` | Yes | One of the 14 keys from P2-2 |
| `variant` | If not default | One of the documented variants for this pattern |
| `tag` | If present | The card's top-left tag (e.g. `"Pattern A · Table"` or `"Variant 1 · Transposed"`) |
| `title` | If present | The card's title text |
| `subline` | If present | The card's `.subline` text |
| `note` | If present | The card's `.note-block` footer text |
| `content` | Yes for non-cover | Pattern-specific payload; see Step P2-4 |

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
- [ ] Every card has a `set` (number or string) and `pattern` (one of the 14 keys)
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

- [ ] First line: object with `type:"meta"`, `title`, `codename`, `palette`
- [ ] No `"pattern":"toc"` cards in file (viewer auto-generates Outline at viewer set 1)
- [ ] Cards in box reading order: Set 0 Card 0 → Set 0 Card 1 → … → Set N Card last
- [ ] Card 0 of each Set has no `variant` field (it is the cover/default)
- [ ] Pattern shared across all cards in a Set; only `variant` differs
- [ ] All hex colours in `content.*.color` and `content.*.accent` rewritten as palette keys
- [ ] Palette names are semantic (`planA`, `agoda`, `bronze`) — never slot indices unless entity is unnamed
- [ ] Variants chosen from the documented list per pattern (see P2-2) or omitted for default
- [ ] Numeric values stay numeric (e.g. `"value":41`); display strings stay strings (e.g. `"display":"41%"`)
- [ ] `series[].values.length === axes.length` for every radar card
- [ ] Bilingual → one language per file; `meta.language` matches the language used; `{en, l2}` objects never embedded in content
- [ ] `meta.theme` emitted only if customising CSS variables beyond viewer defaults
- [ ] No `<span>` tags in any `content.*` text field; allowed inline subset is `<strong>`, `<em>`, `<br>`, `<sup>`, `<sub>` only
- [ ] Checklist items have `"id"` fields; `status` ∈ `{empty, check, strikethrough}`; `variant` ∈ `{default, numbered, grid}`
- [ ] One self-contained `.cards` file
- [ ] UTF-8 encoding, no BOM, no trailing comma, no comments
- [ ] Validates against the Card Box Validator with zero errors