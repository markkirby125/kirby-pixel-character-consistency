# 03 - Consistency Workflows

## The 5-step pipeline (run it once per squad, reuse forever)

1. **Research.** For each real person: two dated sources for skin tone, natural hair
   colour, current style (dye, cut, length), squad number and kit. Real names are fine as
   plain text; never draw crests, sponsor logos or badges.
2. **Write the sheets.** One `@char_sheet` per character from the research. Resolve the
   HAIRS/variant mapping; note anything that would not read at sprite scale.
3. **Uniqueness check.** Collect `(skin, hair, variant)` for the roster; assert every
   tuple unique; resolve collisions by variant first. Record the result.
4. **Preview sheet.** Render a contact sheet (`@preview_sheet`): every character's sprite
   states at final scale (idle + run, keeper states for keepers), with name tags. **Open
   the image and look at it** - silhouettes distinct at a glance, colours reading true,
   variants aligned on-grid, nothing garbled. This is the same discipline as the video
   review gate, applied before any frame is written.
5. **Lock and reference.** Freeze the squad file; clip files reference sheet ids via the
   `@roster_map` and never hard-code colours.

## The roster map

One per video (or one per squad-season, filtered per video):

```yaml
roster: riverside-matchday-2026-11-07
sheets:
  - { id: riverside-okafor,     si: 4 }   # si = legacy engine index, only where the engine still wants one
                                          # MISMATCH (open): si 4 gives DBROWN skin / ORANGE hair; sheet is LSALMON / BLACK
  - { id: riverside-lindqvist,  si: 2 }   # MISMATCH (open): si 2 gives TAN skin; sheet skin is LSALMON (hair DBROWN matches)
  - { id: riverside-castellano, si: 3 }   # sheet not shown in this module: record any mismatch here when it is written
```

Rules:
* The map is the only place looks are assigned. Clip files never pick skin or hair.
* **Legacy-index caveat:** some engines couple skin and hair into one index
  (`skin = SKINS[si % 5]`, `hair = HAIRS[si % 5]`). That coupling cannot express a sheet
  (e.g. dark skin + blond twists). Where it binds, either extend the engine to take the
  sheet's cmap directly (a one-time change - do it, the sheets are the asset), or use the
  closest `si` and record the known mismatch in the map until the engine catches up.
  Never distort the sheet to fit the engine.
* **Multi-team isolation:** the two teams' look spaces must not collide - a home player
  and an away player must never share `(skin, hair, variant)` in the same match video,
  and keepers must be visually distinct from every outfielder on the pitch (kit handles
  most of it; check the tuple anyway).

## Kit swaps = recolour only

Away kits, third kits and keeper kits change the KIT resolution, never the identity
fields. A character in an away kit keeps `skin`, `hair`, `variant`. This is the pixel
analogue of wardrobe decoupling: identity rides in the head rows, clothing rides in the
body rows, and the two never trade places.

## Cross-video continuity

* Same squad file across the season. Looks change only when the real world does (hair
  cut/dye), and then via a dated `version` bump - not silently, not per video.
* Substitutes and new signings get sheets the day they become relevant; a player who
  appears without a sheet is a build blocker, not a judgment call.
* Retired sheets are kept (never deleted) so old videos stay reproducible.

## Big-sprite reuse

The MOTM / podium / poster moments use the big sprite. Read the same sheet: build from
`build`, hair from the big-head overlay of `variant`, kit from the same `team` entry. The
big sprite must be recognisably the same character as the 9 px sprite - same colours, same
silhouette cues - or the payoff moment breaks the identity you built the whole video on.
