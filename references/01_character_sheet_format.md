# 01 - The Character Sheet Format

The `@char_sheet` is the single source of truth for a character's identity. Write it once,
before any clip is built, and never re-decide a look inside a clip file. One sheet per
character, one file per squad (or cast), versioned and dated.

## Schema

Every sheet carries these fields. Keep the field names stable across a project so sheets
can be diffed and validated mechanically.

| Field | Type | Rule |
|---|---|---|
| `id` | slug | Stable forever (e.g. `riverside-okafor`). Never rename; retire instead. |
| `name` | text | Real name as shown on screen tags (uppercase, matches the project's font rules). |
| `number` | int | Squad number. Used for tags and the upload kit, not drawn on the sprite (too small). |
| `role` | enum | `GK` / `DF` / `MF` / `FW` (or the project's own cast roles). Selects keeper sprites and default pitch zones. |
| `team` | key | Key into the project's KIT register (e.g. `riverside`, `harbour`, `gk_lime`). |
| `skin` | palette name | From the SKINS register. Factual: research the person. |
| `hair` | palette name | From the HAIRS register. Factual for natural colour; dyed colours allowed but must be sourced and dated. |
| `variant` | id | Key from the `@variant_lib` (e.g. `short`, `afro`, `bun`). This is drawing style, chosen to read true at sprite scale. |
| `build` | enum | `regular` / `tall` / `stocky`. Informs side-view `z` and casting notes; top-down sprites do not scale. |
| `keeper` | bool | True only for goalkeepers. Sets gloves colour (`k`) and keeper sprite states (dive, punch, slump, sit). |
| `extras` | map | Optional: `gloves` colour, `band` accent colour, `captain` flag (armband colour). Every `extras` key here must be referenced by a letter in the register (`gloves` -> `k`; `band` and `captain` -> `b`, where `captain` overrides `band`; see module 02). |
| `sources` | list | Two dated sources for every factual claim (skin tone, hair colour/style, kit). Real names are fine as plain text; never draw crests, sponsor logos or badges. |
| `version` | date | Last factual revision (hair dye, shaved head). Log the reason in `history`. |
| `history` | list | Optional. One entry per superseded revision: `{ version, date, change_reason }` (`version` = the retired value, `date` = when it changed, `change_reason` = why, e.g. "dyed platinum, per source"). |
| `notes` | text | Optional. Free text for sprite-scale compromises (e.g. nearest honest variant used) and anything the review gate should know. |

## The uniqueness tuple

`(skin, hair, variant)` must be **unique within every frame two characters can share** -
practically: unique across the whole roster that appears in one video, and across both
teams in a match video. Colours are usually fixed by fact, so resolve collisions by
changing the **variant** first (drawing style is yours), then kit contrast, and only
recolour as a last resort with a source. Keepers additionally carry gloves colour, which
helps but does not excuse a tuple collision with an outfielder.

Run the check mechanically: collect tuples for the roster, assert uniqueness, and state
the result in the delivery notes.

## Literal example

```yaml
id: riverside-lindqvist
name: LINDQVIST
number: 17
role: DF
team: riverside        # shirt WHITE, shorts NAVY - plain colours, no crest, no sponsor
skin: LSALMON
hair: DBROWN
variant: long          # hair widening down over the shoulders - his recognisable shape
build: regular
keeper: false
extras: {}
sources:
  - club media guide, 2026-11-01
  - matchday photo set, 2026-11-07
version: 2026-11-07
```

A keeper with extras:

```yaml
id: riverside-okafor
name: OKAFOR
number: 13
role: GK
team: gk_orange        # keeper kit drawn from the KIT register, not the outfield kit
skin: LSALMON
hair: BLACK
variant: short
build: tall
keeper: true
extras: { gloves: WHITE }
sources: [club media guide 2026-11-01, matchday photo set 2026-11-07]
version: 2026-11-07
```

## Field rules that get projects into trouble

* **Skin and hair are facts, not vibes.** Research each person (two sources, dated). No
  sheet without two dated sources: if you cannot verify, stop and escalate to the project
  owner instead of guessing. An unverified sheet never enters a build.
* **Variant is drawing, not fact.** `variant` is the minimum set of pixels that makes the
  silhouette read correctly at 9 px wide: `long` for hair past the shoulders, `afro` for
  volume, `bun` for tied-up hair, `twists` for textured styles, `bald` for none. Do not
  draw facial features, beards or logos - the project's no-likeness rule stands: a generic
  sprite whose only identity is colour + silhouette.
* **Numbers, names and kits are checked against the fact sheet**, like every other fact in
  the video. Kits: plain colours; away kit only if sourced, else say so in the delivery note.
* **Version on change.** Real people change hair mid-season. When a sheet changes, bump
  `version`, add the superseded revision to the sheet's `history` list with its
  `change_reason`, and re-run the preview sheet.
