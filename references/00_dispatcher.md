# Dispatcher: Pixel Character Consistency

Standard operating procedure for keeping sprite characters recognisably identical across every shot, clip, video and season of a **deterministic pixel renderer** (indexed canvas, fixed palette, char-map sprites, pure-function frames). The signature use is 8-bit football video: real people drawn as 9x11 top-down sprites, where each real player must stay the *same invented person* in pixel form for a whole season.

---

## The core idea: consistency by specification, not conditioning

AI video pipelines fight identity drift with reference conditioning (see the sibling skill
`kirby-volumetric-character-consistency`). A deterministic renderer has no drift from
sampling - the same inputs always produce the same pixels. Pixel character drift comes from
**people, not models**: re-deciding a character's skin or hair while writing a clip,
letting colours wander off the palette, editing sprite maps in one video and not another,
or letting two characters collide into the same look. This skill removes drift by making
the character sheet **data**, locked before any frame is written, and by re-checking it
in the visual review gate.

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
Both skills exist because identity is decided ONCE and then enforced EVERYWHERE.
```

## 🏷️ Canonical Asset Tag Register

| Asset Variable | Data Type | Canonical Description | Role |
|---|---|---|---|
| `@palette_lock` | Constant table | The renderer's fixed indexed palette (e.g. 32 colours) plus its named registers: SKINS, HAIRS, KIT and the char-map letter register (four registers). | Every identity colour must resolve to a named palette entry. No free hex. |
| `@char_sheet` | Data (one per character) | The per-character spec: skin, hair colour, hair variant, team (KIT key), keeper flag, build, notes. The single source of truth for identity. | Decides identity once. Everything downstream reads it, never re-decides. |
| `@variant_lib` | Sprite-map rows | Named hair/head variants as char-map rows (same width and row count as the base sprite) plus big-head overlays for large sprites (their own 9x9 contract, see module 02). | Lets characters differ in silhouette without breaking sprite alignment. |
| `@roster_map` | Data (one per video or squad-season) | Assignment of each character appearing in the work to its `@char_sheet` id (and, where the engine still wants a legacy index, that index). | Prevents re-deciding looks per video and collisions inside one frame. |
| `@preview_sheet` | Image (.png) | Contact sheet of each character's sprites at final scale, rendered from the sheets before any video is built. | The human look-check that locks the roster. |

## 🧭 Operational Routing Matrix

| Module | Purpose & Scope | Target Output |
|---|---|---|
| [`01_character_sheet_format.md`](01_character_sheet_format.md) | The sheet schema, field rules, the uniqueness tuple, literal examples. | One `@char_sheet` per character. |
| [`02_palette_and_sprite_registers.md`](02_palette_and_sprite_registers.md) | SKINS / HAIRS / KIT registers, char-map letter register, the hair-variant library, big-head overlays, flip rules. | `@palette_lock` + `@variant_lib` for the project. |
| [`03_consistency_workflows.md`](03_consistency_workflows.md) | The 5-step pipeline, roster maps, kit swaps, keepers, multi-team isolation, cross-video continuity, big-sprite reuse. | Locked roster reused across all shots. |
| [`04_failure_modes_and_review.md`](04_failure_modes_and_review.md) | The eight pixel drift modes, and the eleven sprite checks to add to the project's visual review gate. | Zero-drift delivery with recorded checks. |

## 🛑 When NOT to Use This Skill

* **Photoreal or cinematic AI video**: use `kirby-volumetric-character-consistency`; there is no deterministic renderer to spec against.
* **Single static illustration**: a one-off image does not need a roster map; write the sheet only if the character will return.
* **Diffusion-generated pixel art**: if the "renderer" is actually a diffusion model making pixel-style frames, conditioning (the sibling skill) is the right tool; you may still use `@char_sheet` as the prompt spec, but the guarantees in this skill do not hold.
* **Faceless characters with no identity needs** (indistinguishable crowd extras): keep them generic; sheets are for characters a viewer must tell apart.

## 🔗 Related Skills in the Kirby Ecosystem

* [Kirby Volumetric Character Consistency](../../kirby-volumetric-character-consistency/SKILL.md) - the sibling SOP for diffusion video pipelines (turntables, dual ingestion, wardrobe swaps).
* [Kirby YT Faceless Formula](../../kirby-yt-faceless-formula/SKILL.md) - multi-shot video production pipelines.
* [Kirby Skills Collection](../../kirby-skills-collection/SKILL.md) - master registry and dispatcher for all specialized AI skills.
