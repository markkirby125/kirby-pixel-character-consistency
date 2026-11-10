# 02 - Palette Lock, Registers and the Variant Library

Identity lives in four registers. Define them once per project, freeze them, and reference
them by name everywhere. The examples below use the 32-colour PICO-8 extended palette
(the default for 8-bit football projects); the structure works for any indexed palette.
Index convention: the index columns are positions in the project's 32-entry table; entries
16-31 are the extended colours, mapped from PICO-8's secret-palette slots 128-143.

## The SKINS register

| Name | Index (PICO-8) | Hex | Reads as |
|---|---|---|---|
| `PEACH` | 15 | `FFCCAA` | fairest |
| `LSALMON` | 31 | `FF9D81` | fair, warm |
| `TAN` | 22 | `A28879` | olive / tanned |
| `BROWN` | 4 | `AB5236` | mid-deep |
| `DBROWN` | 20 | `742F29` | deep |
| `K16` | 16 | `291814` | very deep (extended colour) |

Rules:
* Skin tone is factual. Pick the register entry that reads true at sprite scale; when two
  roster members genuinely share a tone, that is fine - the uniqueness tuple still holds
  via `hair` and `variant`.
* Never invent skin tones. No blends, no off-palette hexes, no lighting tints on skin.

## The HAIRS register

Hair reads with less light than skin, so some palette entries that look odd elsewhere read
correctly as hair:

| Name | Index | Hex | Reads as |
|---|---|---|---|
| `BLACK` | 0 | `000000` | black |
| `K16` | 16 | `291814` | blue-black / near-black |
| `DBROWN` | 20 | `742F29` | dark brown |
| `BROWN` | 4 | `AB5236` | mid brown |
| `ORANGE` | 9 | `FFA300` | ginger |
| `TAN` | 22 | `A28879` | dark blond |
| `CREAM` | 23 | `F3EF7D` | blond / platinum |
| `DGREY` | 5 | `5F574F` | grey |
| `LGREY` | 6 | `C2C3C7` | silver / white |

Rules:
* Natural colour from research; dyed colours (platinum, ginger) allowed **with a dated
  source** - hair dye is the most volatile identity fact a person owns.
* `YELLOW` (`FFEC27`) is not a hair colour: it is reserved for payoff type and UI.
* `hair` must resolve to a different palette entry than `skin` (e.g. not `K16` for both),
  or the head loses its hair silhouette. If a person's true colours land on one entry,
  pick the nearest distinct entry and say so in the sheet's `notes`.

## The KIT register

Per team: shirt colour `S`, shorts colour `p`, optional `gloves` for keepers, and for a
striped kit `striped=True` plus two stripe colours `R` and `B`. Sprites always write shirt
cells as `S`; with `striped=True` the renderer paints `S` cells as 2-px vertical stripes,
`R` from the sprite's left column, then `B`, alternating (an odd last column takes the
colour its pair would have had), re-anchored after a flip. A striped entry still sets
`S` to its main colour. Plain colours only - **no crests, no
sponsor logos, no badges**; ad boards are invented or the channel's own brand.

```python
TEAMS = {
    "riverside": dict(S=WHITE, p=NAVY),
    "harbour":   dict(S=WHITE, p=DBLUE),        # kit colours sourced; away kits only when verified
    "riverside_away": dict(S=NAVY, striped=True, R=NAVY, B=WHITE, p=NAVY),  # striped away kit
    "gk_lime":   dict(S=LIME, p=BLACK, gloves=WHITE),
    "gk_orange": dict(S=ORANGE, p=BLACK, gloves=WHITE),
}
```

Keeper kits are their own `team` entries so a keeper is never recoloured from their outfield
team by accident.

## The letter register (char maps)

Sprites are lists of equal-width strings; each letter maps to a palette colour per
character. The base register:

| Letter | Meaning | Resolves to |
|---|---|---|
| `h` | hair | the sheet's `hair` colour |
| `l` | limbs/skin | the sheet's `skin` colour |
| `S` | shirt | team shirt (plain kits) |
| `R`,`B` | shirt stripes | reserved: never written in a sprite; the renderer substitutes the KIT entry's `R` and `B` for `S` cells in striped kits |
| `p` | shorts | team shorts |
| `k` | hands / keeper gloves | keepers: `extras.gloves` if set, else the keeper KIT entry's `gloves`, else a build error; everyone else: `skin` |
| `f` | boots | fixed (usually `BLACK`) |
| `b` | accent (headbands, armbands) | `extras.captain` (armband colour) if the captain flag is set, else `extras.band`; `captain` overrides `band`. Always defined in the base register; the `band` variant needs `extras.band` |
| `.` / space | transparent | - |

Extension protocol: new letters are allowed, but each must be documented here with its
resolution rule and used consistently by every sheet that needs it. One project, one
letter register.

## The variant library

Row layout comes from the project's base sprite: its head block is the rows above the
first shirt row; the body rows are the shirt, shorts and boot rows below it.

Hair variants are **the first 3 rows of the head block**, replacing the default rows, at
the same width as the sprite (9 chars here). Same row count, same width - always, so the
sprite stays on-grid and every other row keeps its alignment. Drawn in the project's
sprite style; keep them chunky and symmetric unless marked flip-aware.

Default (`short`):

```text
...hhh...
..hhhhh..
..hhhhh..
```

| Variant | Rows | Reads as | Flip-aware? |
|---|---|---|---|
| `buzz` | `..hhhhh..` / `..hhhhh..` / `..hhhhh..` | shaved close | no |
| `bald` | `..lllll..` / `..lllll..` / `..lllll..` | no hair (skin head) | no |
| `afro` | `..hhhhh..` / `.hhhhhhh.` / `.hhhhhhh.` | high volume | no |
| `long` | `...hhh...` / `..hhhhh..` / `.hhhhhhh.` | past the shoulders | no |
| `bun` | `....h....` / `...hhh...` / `..hhhhh..` | tied up on top | no |
| `twists` | `..h.h.h..` / `..hhhhh..` / `..hhhhh..` | textured / braided | no |
| `band` | `...hhh...` / `..bbbbb..` / `..hhhhh..` | headband (needs `extras.band`) | no |
| `ponytail` | `...hhh...` / `..hhhhh.h` / `..hhhhh.h` | tail behind | **yes** |

Flip rule: sprite blits mirror when a character faces the other way. Symmetric variants
are safe. `ponytail` mirrors with the sprite - acceptable, but note it on the sheet so the
review gate checks the tail stays behind the head relative to direction of travel.

Adding a variant: keep the 3-row 9-char contract, keep it readable at 1x, prefer
silhouette over detail. If a real hairstyle cannot read at this scale, pick the nearest
honest variant and say so in the sheet's notes.

## Big sprites (side view / podium)

Two different sprites, two contracts:

* **In-play side view (stickman):** identity is kit colours + build (`z`) only; the head
  is drawn in the shirt colour, with no overlay.
* **MOTM / podium / poster big sprite:** must wear the character's hair through a
  `big-head` overlay of the sheet's `variant` (module 03).

The library ships
no overlays: before the first big-sprite render, draw one per variant in use, scaled up
from the 3-row variant's silhouette, and add it to the library. Big-head contract
(separate from the variant contract above): 9 wide x 9 tall, replacing the whole head
block; the 3-row variant contract does not apply. Same rules otherwise: palette colours only, silhouette
over detail, symmetric unless flip-aware.
