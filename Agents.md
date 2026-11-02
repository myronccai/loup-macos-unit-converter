# Agents.md — loup-macos-unit-converter

A native macOS unit converter written in Loup: a category pop-up (Length,
Mass, Temperature, Volume, Speed, Data size), From / To unit pop-ups that
follow it, a value field converted live as you type, a Swap button and a
decimal-places segmented control. It is one of the sample applications built
on the `appkit` library
([loup-macos-appkit](https://github.com/myronccai/loup-macos-appkit), which
must be checked out next to this repository, at `../loup-macos-appkit/`; the
manifest depends on it by that path).

## Read first

- [README.md](README.md) — what the app does, how to build / run / test it,
  its structure, the appkit modules it uses and the toolchain notes. Its
  "Getting started" section lists the requirements (macOS, the Xcode
  Command Line Tools, a Loup toolchain newer than 1.27.0).
- Sources:
  - `src/main.lphc` (`host.native`) — the AppKit window, menus, handlers and
    the `--self-test` driver.
  - `src/units.loup`, `src/numtext.loup` (`host.dual`) — the unit tables,
    conversions, number parsing and formatting; no AppKit.
  - `src/units_test.lpscr`, `src/numtext_test.lpscr` — `loup test` tests.
- The appkit library
  ([loup-macos-appkit](https://github.com/myronccai/loup-macos-appkit),
  checked out at `../loup-macos-appkit/`): the header comment of each module
  in it (`app.lphc`, `objc.lphc`, `controls.lphc`, `layout.lphc`,
  `dialogs.lphc`, …) and `../loup-macos-appkit/docs/*.md`; its probes in
  `../loup-macos-appkit/probes/` are working examples of every module. Its
  `README.md` and `Agents.md` describe the whole collection of example apps,
  and its `run_all.sh` is the gate.
- The Loup macOS developer skill, bundled with the appkit library:
  `../loup-macos-appkit/skills/loup-macos-developer-skill/SKILL.md` (and its
  `references/`).

## Rules for changing it

- Keep logic in the `host.dual` modules and test it with
  `loup test --strict src`; `main.lphc` only wires controls to it. The
  controls hold the state; handlers read them back (the Category menu's
  check marks too, found with `menuBarMenu`). The handlers are methods of
  `Ui` (which owns the window and its controls); `main` and `buildMenus`
  hook them up with closures that capture the `Ui`, and the self-test
  helpers take it as a parameter. There is no module-level state.
- Use the appkit proxies; the app currently sends no raw Objective-C. If a
  library gap forces a raw message, comment why at the call.
- A unit is an affine map to its category's base unit
  (`base = (value + offset) × factor`); a new unit is one line in
  `unitsOf`, with a test. Pop-up titles and symbols must stay unique (each
  category's default pair is looked up by symbol in `defaultSymbols`).
- Number text rules live in `numtext`: commas only as groups of three after
  a first group that does not start with 0, overflow is "Too large",
  scientific notation only at 10^15 and above or for a non-zero result that
  would round to zero at the chosen places (`formatResult`). Change them
  with their tests.
- `units.convertBetween` converts two plain-factor units by their ratio and
  only affine (temperature) pairs through the base unit; keep it that way so
  tiny values stay precise.
- Numbers are formatted by `numtext` digit by digit (fixed decimals, digit
  groups, scientific notation from 10^15), never by the runtime's float
  `to_string`, which prints the shortest round-trip form (the same text on
  every lane, but not a display format).
- Run the app only with `--self-test` (through
  `../loup-macos-appkit/run_all.sh` or lldb); a self-test must drive the
  controls and read results back from them, and print nothing time- or
  pointer-dependent.
- After a change to what the self-test prints, regenerate `selftest.out` and
  check that
  `bash ../loup-macos-appkit/run_all.sh --screenshots loup-macos-unit-converter`
  prints PASS (both toolchains build it and agree).
- Leave no build litter (`.temp/`, `.loup-cache/`, the binary,
  `*.loupbuild.fp`; see `.gitignore`).
