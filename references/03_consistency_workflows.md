# 03 - Consistency Workflows

## The 5-step pipeline (run it once per squad, reuse forever)

1. **Research.** For each real person: two dated sources for skin tone, natural hair
   colour, current style (dye, cut, length), squad number and kit. Real names are fine as
   plain text; never draw crests, sponsor logos or badges.
2. **Write the sheets.** If the project has no registers yet, define them first
   (`@palette_lock`: SKINS, HAIRS, KIT entries per team, the letter register and the
   variant library; module 02), plus one big-head overlay per variant in use if the video
   has a big-sprite moment. One `@char_sheet` per character from the research. Resolve the
   HAIRS/variant mapping; note anything that would not read at sprite scale.
3. **Uniqueness check.** Collect the normalised `(skin, hair, variant)` tuple (module 01;
   bald counts as no hair) for the roster; assert every tuple unique and every sheet's
   `hair` entry different from its `skin` entry (module 02); resolve collisions by variant
   first, then escalate (module 01). Record the result.
4. **Preview sheet.** Render a contact sheet (`@preview_sheet`): every character's sprite
   states at final scale (idle + run, every keeper state for keepers, the big sprite
   where the video has one), with name tags. **Open
   the image and look at it** - silhouettes distinct at a glance, colours reading true,
   variants aligned on-grid, nothing garbled. This is the same discipline as the video
   review gate, applied before any frame is written. If you cannot view images, hand the
   sheet to the owner and record no PASS yourself.
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
  - { id: riverside-castellano, si: 3, kit: riverside_away }   # optional kit override (see Kit swaps); sheet not shown here
                                          # MISMATCH (open): si 3 gives BROWN skin and BROWN hair, one entry (module 02 forbids it)
```

Rules:
* `kit:` is optional and selects a KIT entry for this video (away or third kit). It
  changes the KIT resolution only, never `skin`, `hair` or `variant`. Without it, the
  sheet's `team` applies. Keepers keep their keeper KIT entry unless the override names
  another keeper entry.
* The map is the only place looks are assigned. Clip files never pick skin or hair.
* **Legacy-index caveat:** some engines couple skin and hair into one index
  (`skin = SKINS[si % 5]`, `hair = HAIRS[si % 5]`). That coupling cannot express a sheet
  (e.g. dark skin + blond twists). Where it binds, either extend the engine to take the
  sheet's cmap (the letter-to-colour map resolved from the sheet, module 02) directly (a one-time change - do it, the sheets are the asset), or use the
  closest `si` and record the known mismatch in the map until the engine catches up.
  Never distort the sheet to fit the engine.
* **Kit clash:** the two teams' shirts must differ in colour; where both home kits clash,
  one team wears a sourced away kit through the `kit:` override.
* **Multi-team isolation:** the two teams' look spaces must not collide - a home player
  and an away player must never share `(skin, hair, variant)` in the same match video,
  and keepers must be visually distinct from every outfielder on the pitch (kit handles
  most of it; check the tuple anyway).

## Kit swaps = recolour only

Away kits, third kits and keeper kits change the KIT resolution, never the identity
fields; select them per video with the roster map's `kit:` override, never by editing the
sheet. A character in an away kit keeps `skin`, `hair`, `variant`. This is the pixel
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
