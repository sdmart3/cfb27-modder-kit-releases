# Changelog

## 0.3.0 (2026-10-29): the modder kit
The test runner and the .fbmod pack kit now ship together as one kit (`cfb27-modder-kit`). The runner stays in the top folder
(`cfb-runner.cmd`, `bin/cfb-runner.js`); the pack kit lives in `pack-kit/` with its own `bin`, `lib` and README (`fbmod-pack.cmd` at the top
runs it).

**Save safety (the main change; read docs/SAFETY.md)**
- New SAVE GUARD. Before every launch the kit fingerprints every save in your saves folder except the test save you declared, backs each one up
  outside the saves folder, re-checks them every 5 seconds, after every A press and at the end, and stops the run on any change
  (`cfb-runner guard`, `cfb-runner guard-restore`). Exit code 3 = the guard stopped the run.
- The save routes no longer count rows ("down 2"). `hub-save-over-current` and `hub-save-after-advance` use a new step,
  `{ "saveRow": "declared" }`, that presses DOWN and reads the screen until the highlighted row is your test save. Every A press on a save
  list or overwrite prompt is checked against the screen first and replaced by B if it is not your test save.
- New command `cfb-runner save-ref DYNASTY-<name>`: takes the pictures that check needs. Scenario 07 (saves) refuses to start without them.
- The "load list first entry" calibration picture is stamped with the test save it was taken for; a run for another test save is refused.
  Pictures taken with 0.2 are retaken by the next plain `calibrate`.

**Game updates**
- `calibrate` records the game build (fingerprint of `Data\layout.toc` + the program's build time). `doctor` and `run` notice a game update,
  say "the game updated: re-run calibrate and check the menu counts" and refuse (`--ignore-game-update` overrides; `cfb-runner game-version`).

**Shared mode is the default**
- `doctor --fix` explains the optional Background Pad add-on in plain words and installs it only on a yes (`cfb-runner bgpad install | remove`;
  `--hours N` sets how long it stays armed). This is the one place the kit writes inside the Mod Manager's folder.
- A run uses shared mode and window capture automatically when the add-on is installed, armed and its DLLs are the shipped ones; otherwise it
  is exclusive and prints one line saying why and how to turn shared on. `--exclusive` opts out; `--shared` still works (and refuses to start when
  the add-on is not ready). `--idle-min`, `--now` and `--attended` still mean "wait / watch".
- After a shared run the add-on's own logs are read and the report says whether it installed itself into this game build.

**Other**
- `doctor`: new checks (game build, save guard) and an offer to install the administrator helper; clearer shared-mode line.
- Licence changed from private use to "free to use and share unmodified within the community, not for sale, no warranty".
- README rewritten for people who have never used a terminal; SAFETY and TROUBLESHOOTING updated.
- Pack kit 0.1.0 included unchanged, plus a self-test (`node pack-kit/test/selftest.js`).
- Self-test: `node test/selftest.js` (runner, 57 checks) and `node pack-kit/test/selftest.js` (pack kit, 8 checks); both scan every shipped file for
  personal names and paths.

## 0.2.1 (2026-10-28)
- `doctor` now detects a Mod Manager (or game) that asks for administrator rights through its own built-in manifest. The MMC Mod Manager does, so almost everyone needs the admin helper; 0.2.0 wrongly said "not needed" and the launch then failed with EACCES.
- `doctor` shows why the helper is needed (compatibility setting or built in).
- The runner now launches through the admin helper whenever it is installed (set `alwaysUseBroker false` to opt out). The Mod Manager writes ModData into the game folder, which needs administrator even when the Mod Manager itself opens without asking ("Access to a file or directory is denied" while Creating ModData).
- README: new "Administrator rights: read this first" section before Quick start.

## 0.2.0 (2026-10-23)
- `--shared`: keep using the PC while a run is in progress (keyboard/mouse never stop it, a real controller still does, window capture by default,
  the kit never touches your windows). Needs the optional Background Pad add-on (refuses to start without it; `--no-bgpad-check` overrides).
- `--capture window`: Windows Graphics Capture of the game window (falls back to the screen, counted in the report).
- Stall watchdog (`--stall-min`, default 3, explicit waits excluded).
- The runner's own virtual-pad presses no longer cause false "you are back" stops (idle resets within 2 s of its own input are ignored).
- Polite close first (WM_CLOSE, 90 s game / 20 s manager), then force; the admin helper's stop is polite first too. `--force-close` skips it.
- 8-minute wait for the game when the first process exits and the real one comes later (adopted after 20 s stable); launch retry (3 tries, 30 s apart).
- Optional, off-by-default, read-only reference-file check (`cfb-runner ref-check`).
- `cfb-runner state snapshot|verify|diff|show`: before/after check of plugins, packs, the test save and leftovers.
- `cfb-runner bgpad`: read-only status of the add-on; add-on files and README in `addons/background-pad/` (built files only).
- `docs/WORKING-WITH-CLAUDE.md`; LICENSE and NOTICE (private use, no redistribution or sale).
- Report: mode, launch attempts, how the game closed, stall info.

## 0.1.0 (2026-10-22)
- First pass: generic runner, setup checker, calibration, starter routes, safety defaults. First real run passed on the author's PC.
