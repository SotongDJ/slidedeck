# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **Claude Code skill repository**. The `SKILL.md` is the skill specification — a step-by-step template that instructs Claude how to generate `.cards` JSONL deck files for the Card Box Viewer (card.trth.nl).

There is no build system, no npm packages, no compilation step. The output of the skill is a single `.cards` file — plain UTF-8 JSONL, one JSON object per line. No HTML, CSS, or JS.

> **HTML output removed in v3.0.0.** For `{codename}_desk.html` (Slidedeck) or `{codename}_box.html` (Card Box), use **v2.x** of this skill.

## Skill Invocation

The skill is triggered by user phrases like "make me a slide deck about X" or "create a cardbox for Y". It collects:
- **Codename** — short identifier used in the output filename
- **Content / topic** — the material to present
- **Language** — default English; ask if bilingual is needed and what the second language is
- **Orientation hint** — portrait (default) or landscape; affects layout choices in content schemas

## Generated Output Architecture

The skill produces a `.cards` file with this deck structure as loaded by the Cardbox Viewer:

```
JSONL set 0   →  Viewer set 0  —  Cover    (single "cover" card)
(auto)        →  Viewer set 1  —  Outline  (viewer-generated, never in file)
JSONL set 1   →  Viewer set 2  —  First content set
JSONL set N   →  Viewer set N+1
```

**Key authoring rules:**
- Set 0: single `"pattern":"cover"` card only
- Never emit `"pattern":"toc"` — viewer generates the Outline itself
- Content sets start at `"set":1` in JSONL
- Each content set's first card should be `"pattern":"cover"` with `tag`, `title`, `subline`
- All output is compact single-line JSON (no pretty-printing, no comments)

## Cardbox Viewer Features (v2.0.0)

- **Auto-Outline**: viewer inserts Outline at viewer set 1 automatically
- **Checklist** (`"pattern":"checklist"`): three-state items (`empty`/`check`/`strikethrough`), `localStorage` persistence, stable `id` fields required
- **Jump URL**: `https://card.trth.nl/?jump={viewer-set}_{card}` — uses viewer indices (Cover=0, Outline=1, Content=2+)
- **Save as .cards**: viewer can export live deck state with baked checklist statuses

## Visualization Library (Step P2-2)

14 patterns available as JSONL content objects:

| Pattern | Use case |
|---|---|
| `cover` | Deck cover and content-set cover cards |
| `table` | Feature comparison matrix |
| `bar` | Single-series horizontal bars |
| `segment` | Multi-entity coverage ratings |
| `yn-grid` | Binary feature matrix |
| `pick` | Ordered benefit lists |
| `tier` | Loyalty/subscription tiers |
| `stat` | 2–4 KPI numbers |
| `kpi` | One dramatic headline number per entity |
| `quote` | Reviews or warnings (ok/warn/danger) |
| `radar` | Multi-axis radar chart |
| `brand` | Platform/product overview cards |
| `note` | Footnotes / caption box |
| `checklist` | Interactive three-state checklist |

## Output Formatting Rules

Generated `.cards` output must use **one JSON object per line** with **no extra whitespace** and **no comments**. Code blocks in the skill spec are for readability only — strip all formatting in actual output.

## Updating the Skill Spec

When modifying `SKILL.md` (the skill spec), bump the version string in `SKILL.md` and add a changelog row to `CHANGELOG`. Current version: **3.2.1** (2026-05-26).

### Version release workflow

1. Edit `SKILL.md`: bump version string
2. Edit `CHANGELOG`: add changelog row at the top of the table
3. `git add SKILL.md CHANGELOG`
4. `git commit -S -m "<message>"` — GPG-signed commit
5. `git push`
6. `git tag -s vX.Y.Z -m "vX.Y.Z: <short description>"` — GPG-signed tag
7. `git push origin vX.Y.Z`
