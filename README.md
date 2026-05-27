# Mobile Slidedeck & Card Box Skill

[![Pipeline Status](https://gitlab.com/djtratoh/card.trth.nl/badges/main/pipeline.svg)](https://gitlab.com/djtratoh/card.trth.nl/-/commits/main)
[![Release Skill](https://github.com/SotongDJ/slidedeck/actions/workflows/release.yml/badge.svg?branch=main)](https://github.com/SotongDJ/slidedeck/actions/workflows/release.yml)

A [Claude Code](https://claude.ai/code) skill that generates `.cards` JSONL deck files for the [Card Box Viewer](https://card.trth.nl).

**Current version: 3.5.0**

---

## What It Does

Trigger the skill with phrases like *"make me a slide deck about X"* or *"create a cardbox for Y"*. Claude gathers your content and outputs a single `.cards` file — UTF-8 JSONL, one JSON object per line — ready to load in the Card Box Viewer.

No HTML, CSS, or JS is generated. The viewer handles all rendering.

> **HTML output removed in v3.0.0.** For `{codename}_desk.html` (Slidedeck) or `{codename}_box.html` (Card Box), use **v2.x** of this skill.

---

## Skill Spec

The full step-by-step skill specification is in [`SKILL.md`](SKILL.md).

---

## Releases

Each tagged release publishes a `mobile-slidedeck-vX.Y.Z.skill` zip on the [Releases](../../releases) page. Download and install it in Claude Code as a custom skill.

---

## Changelog

See [CHANGELOG](CHANGELOG) for full version history.
