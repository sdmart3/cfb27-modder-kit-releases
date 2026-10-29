# CFB 27 Modder Kit

Command-line tooling for College Football 27 (PC) modders. Windows only. Free to use and share unmodified, not for sale, no warranty (see `LICENSE`).

- **`cfb-runner`**: a scripted test runner. It launches the game through your MMC Mod Manager using its own `Runner` pack, drives it with a virtual Xbox pad (ViGEmBus via `vgamepad`), and replays JSON routes of presses, waits, screen checks, screenshots and clips. Each run gets a run folder with `REPORT.md`, `report.json` and every capture.
- **`fbmod-pack`** (in `pack-kit/`): packs a `.fbproject`, or a folder of edited legacy files (`.ast` and friends), into a FrostyMod v9 `.fbmod` without the MMC Editor. It reads your install's catalogue from the Mod Manager's cache, Oodle-compresses through the game's own DLL (`koffi`), stamps the installed game version, and reads the result back to verify every payload.

Download: **[latest release](https://github.com/sdmart3/cfb27-modder-kit-releases/releases/latest)** (zip + `.sha256`).

## Requirements

| Need | Why |
|---|---|
| Node.js (version in the zip's README) | both tools |
| Python 3 + `vgamepad` | the virtual pad (`doctor --fix` offers `pip install vgamepad`) |
| ViGEmBus driver | makes the pad look real to the game. Install it yourself as admin, then reboot |
| ffmpeg 6+ (full build with `ddagrab` + `gfxcapture`) | screen capture and comparison (`doctor --fix` offers `winget install Gyan.FFmpeg`) |
| MMC Mod Manager | launches the game with mods applied |

## How the runner works

- **Routes** are JSON step lists: `press`, `wait`, `shot`, `clip`, `note`, `include`, `expect` (wait for a known screen), `until` (repeat until a screen shows) and `saveRow`. A **scenario** names a title, optional `mods`, a test `save`, and includes routes. `cfb-runner serve` gives you a local web driver to record new routes step by step.
- **Screen checks** compare a capture against your own reference pictures from `calibrate`, scored in dB (for example `screen "main menu" reached (61.3 dB, try 1)`). Capture is ffmpeg `ddagrab` (screen, cropped to the game's centred 16:9 frame) or `gfxcapture` (the game window by HWND, which works while other windows cover it).
- **Stall watchdog:** if nothing on screen changes for 3 minutes outside an explicit wait, the run stops with a picture and the last step.
- **Admin:** the Mod Manager's manifest asks for `requireAdministrator`, and an unattended run can't answer UAC. `install-broker` registers two scheduled tasks (`\CFB Test Runner\Launch` / `Stop`) plus a script in admin-only `C:\ProgramData\CFBTestRunner`. They can only start the Mod Manager with the `Runner` pack, and only close processes they started (checked by PID, name and start time). `doctor` reports whether your Mod Manager needs it: compatibility flag or manifest.

## Save safety (new in 0.3)

- **Guard:** before each launch, every file in the saves folder except the declared test save (and its `-AUTOSAVE`) is fingerprinted (name, size, mtime, MD5) and copied to `%LOCALAPPDATA%\cfb-test-runner\save-backups\_guard\<run>\`, outside the saves folder. It re-checks every 5 s, after every A press and at the end. Any change stops the run (exit code 3) and keeps a copy of the changed file. `guard-restore` puts a backup back.
- **`saveRow`:** save routes no longer count rows. The save list re-orders by modified date, so "down 2" lands on a different dynasty whenever any save is written. `saveRow` presses down and classifies the highlighted row against `save-ref` pictures of your test save, and any A press on a save list or overwrite prompt is screen-checked first (it presses B if the row isn't yours).

## After a game update (new in 0.3)

`calibrate` records the build: a hash of `Data\layout.toc` plus the exe's PE timestamp. If either changes, `doctor` fails and `run` refuses, because menu counts and screens may have moved. Start the Mod Manager once so it rebuilds its cache, then `calibrate --redo`, do one attended run, and `game-version record`. `--ignore-game-update` overrides for one run. For the pack kit, run `fbmod-pack.cmd index` again after the Mod Manager refresh.

## Shared mode and Background Pad (default in 0.3)

The game ignores a virtual pad while another window has focus, so you couldn't use the PC during a run. **Background Pad** is an optional Mod Manager plugin (`BackgroundPadPlugin.dll` loads `backgroundpad.dll` into the game). It swallows the game's focus-loss messages, answers its foreground-window queries with the game window, and keeps its handler in front of the game's own. A keyboard guard also stops the game reading keyboard and mouse while another window is in front. Controllers are untouched.

When it's installed, armed and its DLL hashes match the shipped ones, runs use shared mode with window capture automatically. Otherwise they fall back to exclusive mode (wait until you're idle, stop when you come back) and print why. `--exclusive` opts out.

It's code injected into the game process: **offline only**. It auto-disarms after `armed_hours` (default 24) so a forgotten copy never reaches normal play. `bgpad install` / `bgpad remove`, and installing it is the only time the kit writes into the Mod Manager folder (only when you say yes).

## pack-kit commands

```
fbmod-pack.cmd index                                   # catalogue from your install + Mod Manager cache (read-only; rerun after updates)
fbmod-pack.cmd project MyMod.fbproject --out MyMod.fbmod --title "My Mod" --author You --category Gameplay --version 1.0
fbmod-pack.cmd list "common/ui/**"                     # legacy paths in the game
fbmod-pack.cmd extract common/ui/<folder>/<file>.ast   # stock copy to edit
fbmod-pack.cmd legacy C:\work\edited --out MyUiMod.fbmod --title "My UI Mod"
```

- `project` checks that every referenced bundle exists and that modified plugin tables match the game's, then reads the mod back and compares every Ebx payload.
- `legacy` checks that each path exists and that every container listing it holds identical copies, Oodle round-trips each payload, then reads the mod back.
- Neither writes into the game or Mod Manager folders.

## How I've used it

**1. Weekly Strategy: building new practice screens from a direction, not a spec.** The game has a weekly strategy / training screen that dynasty never turns on. Its dynasty data hooks are stubs that return nothing.
- **Who does what:** I point at what the screen should do: the opponent's season numbers, player health on its own page, focus players who get extra practice XP, weekly goals. I add a rough idea of how, Claude runs with it, and the reverse engineering and the build come back to me.
- **What got built:** a small plugin that fills the screen's data slots from the dynasty (tiles, opponent stats, health, goals), expression hooks for focus XP and goals, and UI mods that rewire the screen.
- **Where the runner fits:** every build gets a short scripted pass: load the test dynasty, open the screen, check each panel, advance a week, save. The screenshots come back for review, so I'm looking at the new screen instead of launching the game myself.
- **What it caught that offline checks couldn't:**
  - START TRAINING opened the drill screen with an empty list, because the drills only fill for a Road to Glory player.
  - The first entry point was the R3 button, where nobody would press it, so it moved to View.

**2. Focus, test length and admin: three snags that shaped the defaults.**
- A probe run showed the pad is ignored while another window has focus (the menu didn't move). That became Background Pad, and the next probe passed with input landing while unfocused.
- A planned test batch came out around 10 hours because every test played full games. Scoping each test to what the change touched (one take, one snap, quit without saving, the same card again for the next case) brought it to about an hour.
- A tester got "Access to a file or directory is denied" while creating ModData. Some Mod Manager builds require admin in their manifest, so `doctor` now reads the manifest and the compatibility flags and points at the broker.

**3. Press Box: correct and learn while building.** Press Box (play, coach or watch any CPU-vs-CPU dynasty game) first exposed its games through a hidden hub item with a numbered chooser of 25 to 33 entries. The runner's routes (take a game, play it, watch it, advance weeks) all passed. The captures showed entries 1 to 5 drawn above the top of the screen, and the flow made no sense to anyone who hadn't built it. A route passing isn't the same as a player finding it, so Press Box moved to the game's own Scores / Schedules screen: pick a game, it becomes a hub card. Test reviews now also ask whether a player would find it, in the game's own wording.

## Status

Proven for real on one PC and one game build: launch through the broker, the virtual pad, screen checks, polite stop and restore, and Background Pad (a menu probe plus one live game). Offline only so far (self-tests, fake folders, real screenshots): the save guard, `saveRow` and the A-press gate, the game-update check, and shared-by-default. Report what breaks.

[CHANGELOG.md](CHANGELOG.md) lists every version's changes.
