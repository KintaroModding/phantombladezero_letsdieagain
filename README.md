# Let's Die Again!

The main menu's CONTINUE entry reads LET'S DIE AGAIN! instead (English text).

A mod for Phantom Blade Zero (Unreal Engine 5.7), made with game version `CL-301524_JKS-17219`. This README is
for developers. What the mod does, how to install it and its compatibility are in [openmods.md](openmods.md), the mod
page on [openmods](https://openmods.nexus).

## How it works

The game keeps its texts in one file per language in its `ProjectPBZ-Windows.pak`. The mod replaces the English one,
`ProjectPBZ/Content/Localization/Game/en-US/Game.locres` (locres version 3, namespace `Default`, 11135 entries), with
a copy in which one text changes:

| Namespace | Key | Game | Mod |
|---|---|---|---|
| `Default` | `UI_StartMenu_02` | `CONTINUE` | `LET'S DIE AGAIN!` |

The game's start menu script sets the Continue button's text from this key (`StartMenu.js`, `InitText`). Only this
key changes; its source hash and every other entry stay as they are. The text is in capitals like the other menu
entries (NEW GAME, LOAD GAME, SETTINGS, QUIT GAME). The first commit holds the game's original file; a text dump of
both files (for example with a `.locres` tool) differs in this one line.

The mod replaces whole files. A mod that changes the same files conflicts with this one, and a game update that
changes them needs a new build of the mod from the new files.

## Build

Requirements:

- Windows PowerShell 5.1 (part of Windows).
- [repak](https://github.com/trumank/repak/releases) v0.2.3 and [retoc](https://github.com/trumank/retoc/releases)
  v0.1.5. `build.ps1` looks for `_tools\repak\repak.exe` and `_tools\retoc\retoc.exe` in the parent folders of the
  project; set `UE5_TOOLS` to a folder that holds both to use another place.
- Optional, for mods with JavaScript: [Node.js](https://nodejs.org/) for the syntax check and the offline test
  (`tests\test_mod.js`).

```powershell
powershell -ExecutionPolicy Bypass -File .\build.ps1             # dist\phantombladezero_letsdieagain_P.pak/.utoc/.ucas
powershell -ExecutionPolicy Bypass -File .\build.ps1 -Deploy     # build, then copy the files to Paks\~mods
powershell -ExecutionPolicy Bypass -File .\deploy.ps1 -Wait      # copy the files when the game is closed
```

The build checks every `.js` in `src` with `node --check` and runs `tests\test_mod.js` when there is one. Then it
packs `src` into a `.pak` (repak, pak version V11, mount point `../../../`), writes an empty IoStore container of the
same name (retoc `to-zen`; an IoStore game can ignore a `.pak` without one), and checks that the pak holds exactly
the files of `src`, byte for byte.

`deploy.ps1` copies the three files to `<game folder>\ProjectPBZ\Content\Paks\~mods`. The game folder comes from `gameDir` in
`_tools\gamedata\phantombladezero\game.json`, else `$env:UE5_GAMES_DIR\PBZ`; `-GameDir` or `-ModsDir` override
it. The running game locks the files, so the script stops (exit code 2) or, with `-Wait`, waits until the game is
closed.

Versioning: every change raises the second number in `mod.json` (1.0 -> 1.1): `python scripts\bump_version.py`.

## Project layout

```
src/                               Mod source: the changed game files at their path in the game's .pak
tests/test_mod.js                  Offline test of the scripts (Node.js; only for mods with JavaScript)
dist/                              Build output: the .pak/.utoc/.ucas for Paks\~mods (git-ignored)
docs/                              Cover and screenshots for openmods.md
scripts/bump_version.py            Raises the version in mod.json
build.ps1                          Checks and builds dist\ from src\
deploy.ps1                         Copies the build to the game's Paks\~mods folder
mod.json                           Name, version, game, engine and file names
```

The first commit with a game file holds the original file, so `git diff` against it shows exactly what the mod
changes.

## License

MIT, see [LICENSE](LICENSE). The files in `src` are the game's own files with the changes above; Phantom Blade Zero and
its files belong to their makers.
