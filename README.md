# Pixel Character Consistency 🧱🎥

> **Deterministic standard operating procedure (SOP) for keeping sprite characters recognisably identical across every shot, clip, video and season of a pixel/8-bit renderer.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Architecture: Hollow Shell](https://img.shields.io/badge/Architecture-Hollow%20Shell-brightgreen.svg)](SKILL.md)
[![Sibling: Volumetric Character Consistency](https://img.shields.io/badge/Sibling-Volumetric%20Character%20Consistency-orange.svg)](https://github.com/markkirby125/kirby-volumetric-character-consistency)

AI video pipelines fight character drift with conditioning: turntable videos, reference sheets, prompt locks. A deterministic pixel renderer has no sampling drift - the same inputs always draw the same pixels. Its character drift is **human drift**: a skin tone re-decided from memory in video 4, a hair colour nudged off the palette, a sprite map edited in one clip and not the next, two characters colliding into the same silhouette.

This skill removes that drift the same way the volumetric skill removes its own: **identity is decided once, then enforced everywhere**. The character sheet is data - palette-locked, researched, versioned - and every clip reads from it through a roster map, with a preview contact sheet and sprite-level review-gate checks standing in front of the render.

---

## The paradigm: specification, not conditioning

```text
DIFFUSION PIPELINE (sibling skill)          DETERMINISTIC PIXEL PIPELINE (this skill)
┌────────────────────────────────┐          ┌────────────────────────────────┐
│ Identity carried by:           │          │ Identity carried by:           │
│ 360° turntable video + sheet   │          │ character sheet + palette lock │
│ Risk: model drift, melting,    │          │ Risk: human drift - re-decided │
│ mirroring, hallucinated angles │          │ looks, palette wander, sprite  │
│ Fix: dual ingestion + QA gate  │          │ map edits, look collisions     │
│                                │          │ Fix: sheet-first + review gate │
└────────────────────────────────┘          └────────────────────────────────┘
```

Signature use: **8-bit football video** - real players drawn as 9x11 top-down sprites, where each real person must stay the *same invented person* in pixel form for a whole season, across goal Shorts, loop cuts and 16:9 match reels, in home kits, away kits and keeper kits.

---

## 🪄 Universal Agent Installation

Copy and paste this "Magic Prompt" into **any** AI coding assistant (Antigravity, Cursor, Windsurf, Claude Code, Grok, Kimi, Reasonix, Codex, Roo Code):

```markdown
@agent Install the Pixel Character Consistency skill from: https://github.com/markkirby125/kirby-pixel-character-consistency
1. Read `SKILL.md` and `references/` from the repository.
2. Place into your environment's skills/rules directory preserving the dispatcher structure.
3. Confirm when installation is complete.
```

### Manual Installation (Local Symlink)
To deploy locally across your AI coding environments, clone once into a skills root, then link it into each app's skills directory (replace the `<...>` placeholders with your own paths):
```bash
git clone https://github.com/markkirby125/kirby-pixel-character-consistency.git <skills-root>/kirby-pixel-character-consistency
TARGET="<skills-root>/kirby-pixel-character-consistency"
ln -sfn "$TARGET" <app-skills-dir>/kirby-pixel-character-consistency   # repeat per app, e.g. Cursor, Windsurf, Claude Code, Grok, Kimi, Reasonix
```

---

## 📊 Drift-Mode Coverage

Eight drift modes, in the same order as [`references/04_failure_modes_and_review.md`](references/04_failure_modes_and_review.md) (the canonical list):

| # | Drift Mode | Free-form sprite work | Sheet + Roster Map + Preview Gate |
|---|---|---|---|
| 1 | Re-decided look | Frequent; invisible until side by side | Impossible by construction; map is the only assignment point |
| 2 | Palette wander | Common; off-palette skin/hair or a lighting tint | Registers resolve by name; only the CRT pass may touch pixels |
| 3 | Variant misalignment | Occasional; head floats off shirt | Variant library keeps the row/width contract; preview catches it |
| 4 | Legacy-index squeeze | Common where an engine couples skin and hair | Extend the engine to take sheet cmaps; open mismatches go in the delivery note |
| 5 | Collision | Common with big rosters | Mechanical uniqueness tuple per roster |
| 6 | Flip betrayal | Undetected until a viewer spots it | Flip-aware flags + review-gate direction check |
| 7 | Big-sprite split | Podium sprite reads as a stranger | Same sheet drives 9 px and podium sprites |
| 8 | Unsourced or stale fact | Silent inconsistency (guessed look, hair dye or cut never logged) | Two dated sources per sheet; dated `version` bump with logged reason + re-preview |

---

## 🏷️ Canonical Asset Tag Register

| Asset Variable | Data Type | Canonical Purpose |
|---|---|---|
| `@palette_lock` | Constant table | Fixed indexed palette + named SKINS / HAIRS / KIT registers + the char-map letter register. No free hex. |
| `@char_sheet` | Data (per character) | The single source of truth: skin, hair colour, variant, team (KIT key), keeper flag, build, sources, version. |
| `@variant_lib` | Sprite-map rows | Named hair/head variants at fixed width/row count, plus big-head overlays. |
| `@roster_map` | Data (per video / squad-season) | Assignment of characters to sheets; the only place looks are assigned. |
| `@preview_sheet` | Image (.png) | Contact sheet of every character at final scale; opened and reviewed before any build. |

---

## 🚀 Core Workflows

### 1. The 5-Step Squad Pipeline
Research (two dated sources per person) → write sheets → mechanical uniqueness check → render and **look at** a preview contact sheet → lock the squad file and reference it everywhere.

### 2. Roster-Driven Clips
Clip files reference sheet ids through the roster map. Where an engine still couples skin+hair into one legacy index, extend the engine to take sheet cmaps - never distort the sheet to fit the engine.

### 3. Kit Swaps (Wardrobe Decoupling)
Away/third/keeper kits recolour the body rows only. Identity rides in the head rows; clothing rides in the body rows; the two never trade places.

### 4. Big-Sprite Reuse
MOTM podium and poster sprites read the same sheet (build, hair overlay, kit) so the big moment shows the same person the viewer watched for 90 seconds.

---

## 🧭 Where to Read Next

Start at [`references/00_dispatcher.md`](references/00_dispatcher.md), the master router, then open the module for the task in hand:

| Task | Module |
|---|---|
| Write a character sheet | [`01_character_sheet_format.md`](references/01_character_sheet_format.md) |
| Define palette, registers, variants, big-head overlays | [`02_palette_and_sprite_registers.md`](references/02_palette_and_sprite_registers.md) |
| Run the 5-step pipeline, roster maps, kit swaps, big-sprite reuse | [`03_consistency_workflows.md`](references/03_consistency_workflows.md) |
| Review a build: eight drift modes and the twelve sprite review-gate checks | [`04_failure_modes_and_review.md`](references/04_failure_modes_and_review.md) |

---

## 🛠️ Tooling Notes

This skill is renderer-agnostic but tuned for deterministic Python pixel kits (indexed numpy canvas, char-map blits, 5x7 pixel font, CRT pass). It pairs naturally with an 8-bit football project spec that already defines the canvas sizes, palette, dithering and review gate; the sprite checks here bolt onto that gate unchanged.

---

## 🗂️ Repository Structure

This repository strictly implements the **Hollow Shell Architecture** (root `SKILL.md` ≤ 200 tokens) to prevent AI context window bloat:

```text
kirby-pixel-character-consistency/
├── SKILL.md                                  # Lightweight dispatcher shell (≤ 200 tokens)
├── README.md                                 # Public documentation & installation guide
├── LICENSE                                   # MIT License
└── references/
    ├── 00_dispatcher.md                      # Master router & Canonical Asset Tag Register
    ├── 01_character_sheet_format.md          # Sheet schema, field rules, uniqueness tuple
    ├── 02_palette_and_sprite_registers.md    # SKINS/HAIRS/KIT registers, letter register, variant library
    ├── 03_consistency_workflows.md           # 5-step pipeline, roster maps, kit swaps, big sprites
    └── 04_failure_modes_and_review.md        # Eight drift modes & the sprite review gate
```

---

## 📜 Primary Source & Attribution

Methodology extracted from an in-house deterministic pixel-renderer pipeline for 8-bit football highlight videos: PICO-8 palette, char-map sprite recolours and a mandatory visual review gate.

---

## 📄 License

MIT License. Copyright (c) 2026 Mark Kirby. Distributed as part of the [Kirby Skills Library](https://github.com/markkirby125/kirby-skills).
