# 04 - Failure Modes and the Sprite Review Gate

Deterministic renderers ship a different kind of slop: not model melting, but quiet
human drift - the third video quietly gives the striker the wrong hair because someone
re-decided it from memory. Eight ways it happens, and the checks that catch each.

## The eight drift modes

| # | Drift mode | What it looks like | Fix |
|---|---|---|---|
| 1 | **Re-decided look** | Same character different hair/skin in video 4 | `@roster_map` is the only place looks are assigned; clip files reference ids |
| 2 | **Palette wander** | Skin or hair is a near-miss hex, or a lighting tint | Registers resolve by name; the only sanctioned post-pass is the project's CRT pass |
| 3 | **Variant misalignment** | Hair rows shifted, orphan pixels, head floats off the shirt | Variant library keeps the row-count/width contract; preview sheet catches it |
| 4 | **Legacy-index squeeze** | Dark-skinned player drawn with the hair the player's coupled index implies | Extend the engine to take sheet cmaps; never distort the sheet to fit the engine. Until then, record every open `si` mismatch in the roster map and the delivery note |
| 5 | **Collision** | Two players in one frame are the same person at a glance | Uniqueness tuple per roster; away team checked against home |
| 6 | **Flip betrayal** | Asymmetric variant mirrors so the tail leads the face | Flip-aware variants flagged in the library; review gate checks direction of travel |
| 7 | **Big-sprite split** | Podium sprite reads as a different person than the pitch sprite | Same sheet drives both; hair overlay + kit + build reused, not re-invented |
| 8 | **Unsourced or stale fact** | A sheet built on a guess, or a hair dye/cut that never reached the sheet | No sheet without two dated sources; real-world change = dated `version` bump with a logged `change_reason`, then re-preview |

## Sprite checks to add to the visual review gate

The project's frame review gate already looks at every sheet of frames. Add these twelve
checks to its checklist for any video with sheet-driven characters. Checks that read
hair, skin or variant apply to frames that draw them (top-down sprites, the big sprite);
in in-play side-view frames, which draw kit and build only (module 02), check kit colours
and build against the sheet instead:

- [ ] Every named character in frame matches their sheet: skin, hair colour, variant, kit
      (read the tuple off the frame, not from memory)
- [ ] No two characters in any frame share a normalised `(skin, hair, variant)` tuple
      (module 01: bald counts as no hair), and the two teams' shirts differ in colour
- [ ] Every identity colour is a named palette entry (no off-palette tints anywhere)
- [ ] Hair variants sit on-grid: correct row count, no orphan pixels, head seated on the
      shirt rows
- [ ] Flip-aware variants point the right way relative to movement
- [ ] Keepers: gloves colour on the keeper only; keeper kit from the keeper KIT entry
- [ ] Striped kits: stripes continuous through body rows, no stripe bleed into head rows
- [ ] Numbers/names in tags match the sheets the roster map points to (font rules:
      uppercase, no brackets)
- [ ] Big sprite (MOTM / podium / poster): wears the big-head overlay of the sheet's
      `variant`, the sheet's colours, kit entry and build
- [ ] The `@preview_sheet` for this roster was opened and looked at before the build
      (record it in the review log)
- [ ] Every sheet in the roster has two dated `sources` (no unverified sheet in a build)
- [ ] After any real-world change (hair dye, cut), `version` was bumped and the reason
      logged in `history`

## The preview gate (before any build)

Run before the first render of any new roster or after any sheet change:

1. Render `@preview_sheet`: one cell per character - idle and run states (plus every
   keeper state for keepers, and the big sprite where the video has one) at final integer
   scale, name tag under each.
2. Open the image. Check at full size (alignment, stray pixels) **and** at the size the
   sprite occupies in the delivered frame (does each character still read as the same
   character?).
3. Fix at the sheet or library, never by patching the preview.
4. Record "preview sheet reviewed PASS" in the review log with the date.

## Delivery note

State in the delivery message: the roster map name and the `version` of each sheet used,
any `kit:` overrides, the uniqueness check result,
preview review result, and any `si` mismatches still open from the legacy-index caveat.
