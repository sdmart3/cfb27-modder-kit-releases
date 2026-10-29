# CFB 27 Modder Kit

**Two command-line tools for people who make College Football 27 mods.**

- **Test runner (`cfb-runner`).** Replays the same menu steps in the game with a virtual controller and takes screenshots along the way, so testing a mod doesn't mean clicking through the same menus every time. With the optional Background Pad add-on, you keep using your PC while it runs.
- **.fbmod pack kit (`fbmod-pack`).** Packs an editor project (`.fbproject`) or a folder of edited legacy files into an installable `.fbmod` from the command line, without opening the MMC Editor.

Windows only. Free to use and share unmodified, not for sale, no warranty (see `LICENSE`).

## Download

Get the latest zip from **[Releases](https://github.com/sdmart3/cfb27-modder-kit-releases/releases/latest)**. Each release has a `.sha256` file with the zip's checksum.

## What you need

- Windows 10 or 11, College Football 27 on PC, and the MMC Mod Manager.
- Node.js (the version is in the README inside the zip).
- For the test runner: Python with `vgamepad` (the ViGEmBus virtual controller driver). `doctor --fix` offers to install what's missing.

## Before your first run

- **Administrator rights.** The Mod Manager asks for administrator rights every time it starts, and a test that starts on its own can't click a Windows prompt. `doctor --fix` offers a small helper for that: one prompt to install it, none during runs, and `uninstall-broker` removes it.
- **Your saves.** Every save except one throwaway test dynasty is backed up outside your saves folder before each launch and watched while the game runs. Any change stops the run. Save screens pick the row by reading its name on screen, never by counting down the list.
- **Game updates.** After a game update the runner refuses to run until you set it up again (`calibrate`), because menus can move.

## Start

```
node bin\cfb-runner.js doctor --fix
node bin\cfb-runner.js calibrate
node bin\cfb-runner.js run 01-title-to-main-menu --attended
```

The README inside the zip walks through every step, every scenario, writing your own routes, save safety, Background Pad, and every `fbmod-pack` command.

## How I've used it

**1. Press Box: a test that passed, on a design no person could use.** Press Box lets you play, coach or watch any CPU-vs-CPU game in your dynasty. The first version put its game list behind a hidden hub item with a numbered chooser of 25 to 33 entries. The runner took games, played them, watched them and advanced weeks, and every run passed. But its screenshots showed entries 1 to 5 drawn above the top of the screen, and the whole flow made no sense to anyone who hadn't built it. A pass isn't the same as usable, so Press Box was rebuilt around something players already use: pick a game on the Schedule and it becomes a card on your hub. Tests now also ask "would a player find this, and does it use the game's own words?", not just "did it work?".

**2. My own lab overwrote a real save, and the kit got a save guard.** A save route pressed "down 2" to reach the test dynasty. The save list re-orders by date, so one night "down 2" landed on a real dynasty's autosave and the overwrite prompt got a Yes. That's why 0.3 backs up every other save before each launch, watches them while the game runs and stops on any change, and why save screens are read, not counted. On the same day a game update landed: the runner hit the Mod Manager's "SDK out of date" screen, stopped every launch instead of pushing through, and that became the game-update check.

**3. Sharing the PC, and sizing tests to what changed.** The first shared runs failed: a probe showed the game ignores a virtual controller while another window is in front. That finding became the Background Pad add-on; the next probe passed, and now tests run while I use the PC. Around the same time a planned batch of tests came out at about 10 hours, because every test played full games. Cutting each test to only what the change touched (one take, one drive, one advance) brought it to about an hour. And when a friend hit "access denied", it turned out the Mod Manager always asks for administrator rights, so `doctor` now checks for that and explains the helper.

## Status

The runner's launch, virtual controller, screen checks, stop and restore have run for real on one PC and one game build. The save guard, the screen-checked save rows, the game-update check and shared mode by default have been tested offline so far. Report what breaks.

See [CHANGELOG.md](CHANGELOG.md) for what changed in each version.
