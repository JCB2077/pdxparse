# pdxparse work in progress - handoff

Paste this into a new task if the device re-link isn't possible.

## What this is

JCB maintains a fork of pdxparse (a Haskell parser that turns Paradox game
script into wiki tables) at **github.com/JCB2077/pdxparse**, used to generate
content for the Old World Blues, Equestria at War, Red Flood and The New Order
wikis on wiki.gg.

Work happens on the **`custom-for-mod-wiki`** branch, not `master`.

## State as of this handoff

Branch `custom-for-mod-wiki` is at `3e683be`, pushed, **6 ahead / 0 behind**
BiscuitCookies upstream. Tag `v0.8.11.1` points at it and a GitHub Actions
build was running. Repo variable `GAME=hoi4` is set (the workflow needs it).

### Changes made

National focus output (`src/HOI4/NationalFocus.hs`):
- `cost` is now rendered as a completion time. It was parsed and discarded, so
  focus tables never showed how long a focus takes. Default changed from
  `undefined` to 10, which would otherwise have crashed the writer on any
  focus omitting an explicit cost.
- `cancel`, `cancelable`, `available_if_capitulated`, `cancel_if_invalid`,
  `continue_if_invalid`, `will_lead_to_war_with` and `search_filters` all
  parsed to no-ops despite having fields. Now populated and partly rendered.
- `alternate_icon` rendered as a comment reading "ALT icon presentt"; now
  shows the icon.
- Fixed a malformed HTML comment (`->` instead of `-->`) in the
  completion-tooltip block.
- Rows carry a `<span id="focus_id">` anchor; the existing `id=` uses the
  localised name, which is neither unique nor stable.

Cross-wiki compatibility:
- New `focus_box_template` setting in settings.yml, default `iconbox`.
  Needed because `{{Iconbox}}` is an achievements template on OWB and EAW
  (derives its image from the name, ignores `image=`) and doesn't exist at all
  on TNO.
- `wiki-templates/` holds drop-in wikitext for every template the output
  depends on, a `mildtable` stylesheet, and notes on patching TNO's
  `Template:Icon` (which takes a named `iconfile=` parameter, so all ~280
  positional `{{icon|...}}` calls render empty there).

Regressions repaired from an earlier bad merge on this branch:
- `src/HOI4/Settings.hs` had the whole technology and unit-tag subsystem
  deleted (37 lines restored).
- `src/HOI4/CharactersAndTraits.hs` had lost the `scientist` clause.
- `clt_cp_cap` was a field in `Types.hs` with no initialiser, which is
  `-Wmissing-fields` and this branch builds with `-Werror`, so the branch as
  previously pushed would not have built.
- Enabled `writeHOI4NationalFocuses`, which was commented out.

The whole tree type-checks clean: 73 modules, zero errors, zero warnings in
project source.

## What's left

Run it for the first time. Six writers are now enabled (technologies,
designers, ideas, characters, country leader traits, unit leader traits,
national focuses) and none has been run against real game files.

1. Unzip the CI artifact `pdxparse-v0.8.11.1-hoi4-windows` into `F:\pdxparse`.
2. Put the accompanying `settings.yml` **next to pdxparse.exe**. The lookup is
   `replaceFileName exePath "settings.yml"`, falling back silently to the copy
   Cabal installed, so editing the wrong one appears to do nothing.
   Verify with `pdxparse.exe --paths`.
3. From `F:\pdxparse`, run:

       pdxparse.exe --nowait > run.log 2>&1

   Output lands in `.\output\` relative to the **current directory**, not next
   to the exe.
4. Read `run.log`. Parsers emit `trace` lines for script they don't recognise.

## Gotchas already hit

- The repo clone was under OneDrive. Two file writes silently reverted:
  reported success, timestamps updated, content was the old version. Keep the
  clone outside OneDrive.
- Localisation and interface (GFX) lookups read from **both** the mod path and
  the base game path, so the `steam_*` settings matter even in mod mode.
- Setting `mod_name` makes `buildPath` read only the mod folder for scripts.
  Fine for a total conversion like OWB.
- `-Werror` is on. GHC 9.4 flags `HOI4/Events.hs:242` as an overlapping
  pattern; CI uses 8.10.7 where it likely doesn't fire, but that's the first
  thing to check if the Windows build fails.
- The release job is `if: always()`, so a failed build still leaves an empty
  draft release. Check the `build` job, not the release.

## Preference

No em dashes, ever. Hyphens are fine.
