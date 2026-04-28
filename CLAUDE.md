# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **Claude Code skill repository**. The `README.md` is the skill specification (effectively a `SKILL.md`) — an 887-line step-by-step template that instructs Claude how to generate self-contained, single-file mobile slide deck web apps.

There is no build system, no npm packages, no compilation step. The output of the skill is a single `.html` file with all HTML, CSS, and vanilla JavaScript inlined.

## Skill Invocation

The skill is triggered by user phrases like "make me a slide deck about X" or "create a slidedeck for Y". It collects three inputs:
- **Codename** — a short identifier for the deck (used in localStorage keys and the title)
- **Font Awesome kit URL** — for icons in the topbar
- **Orientation** — portrait (9:16, default) or landscape (16:9), inferred from context if not stated

## Generated Output Architecture

The skill produces a single HTML file structured as:

```
<html data-theme="light" data-orient="portrait">
  <body>
    <div class="deck">
      <div class="deck-top">     <!-- topbar: page counter | font size | controls -->
      <div class="slides-area">
        <div class="slides-track">   <!-- 400% wide, CSS transform-based sliding -->
          <div class="slide">...</div>
        </div>
      </div>
      <div class="deck-bot">     <!-- prev | dot indicators | next -->
    </div>
  </body>
</html>
```

**Key implementation details:**
- Navigation uses CSS `transform: translateX()` on `.slides-track`, not JavaScript scroll
- State (page, font size, theme, orientation) persists via `localStorage` + URL query params, with boot priority: URL params → localStorage → hardcoded defaults
- Touch swipe threshold: 36px; keyboard: arrow keys
- Font size range: 3–69px in steps of 3; portrait default 21px, landscape default 18px
- Theme cycles: light → dark → system (respects `prefers-color-scheme`)
- Print/PDF: `slidesToPrintPDF()` opens a new window with one page per slide for browser print

## CSS Theme System

All colors are CSS custom properties on `:root` with two sets (light/dark). Key semantic variables: `--bg`, `--text`, `--border`, `--accent`, `--ok` (green), `--warn` (amber), `--danger` (red), plus `--*-bg` tinted backgrounds for each semantic color. A `[data-theme="system"]` block uses `@media (prefers-color-scheme: dark)`.

## Visualization Library (Step 8 of the skill)

11 reusable HTML/CSS patterns defined in the skill spec:

| ID | Pattern | Use case |
|----|---------|----------|
| 8-A | Table | Feature comparison matrix |
| 8-B | Bar chart | Single-series horizontal bars |
| 8-C | Segment bar | Multi-entity ratings |
| 8-D | Yes/No grid | Binary feature matrix |
| 8-E | Pick list | Ordered benefits |
| 8-F | Tier ladder | Loyalty/subscription tiers |
| 8-G | Stat strip | 2–4 KPI numbers |
| 8-H | KPI Insight card | One dramatic headline number |
| 8-I | Quote callout | Reviews or warnings |
| 8-J | SVG Radar chart | 6-axis hexagon (overflow-safe, SVG viewBox) |
| 8-K | Brand accent card | Platform/product overview |
| 8-L | Caption/note box | Footnotes |

## Output Formatting Rules (Step 0.5)

Generated HTML must use **zero indentation** and **no comments**. These are intentional constraints to keep output compact — do not add indentation or inline comments when generating or editing slide HTML files.

## Updating the Skill Spec

When modifying `README.md` (the skill spec), update the version number and changelog at the top of the file. Current version: **1.7.1** (2026-04-22).
