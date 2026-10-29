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

## Status

The runner's launch, virtual controller, screen checks, stop and restore have run for real on one PC and one game build. The save guard, the screen-checked save rows, the game-update check and shared mode by default have been tested offline so far. Report what breaks.

See [CHANGELOG.md](CHANGELOG.md) for what changed in each version.
