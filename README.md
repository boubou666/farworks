# Farworks

An automation game on an unmapped planet. You come down hard with nothing but a broken escape pod:
strip it for a mining pistol, raise a Main Station out of its iron, and start cutting ore out of
the ground. Everything after that is belts, smelters and the question of what should be feeding
what.

The world is real 3D under a camera you can turn a quarter at a time, so a plateau is something you
walk up and stand on rather than something drawn behind you.

![In front of a tree](docs/screenshots/v1.35.0/81-in-front-of-a-tree.png)

## Download it

**[Latest release](../../releases/latest)** — every version is here as a standalone player.
Nothing to install.

**Windows** — take the `.zip`, unpack it anywhere, run `Farworks.exe`.

**Linux** — take the `.tar.gz`, `tar xzf` it, run `./Farworks`.

Saves live in your user data folder rather than in the game's, so a new version unpacked beside the
old one picks up a game in progress where you left it.

Windows may say it does not recognise the publisher, which is what it says about anything unsigned.
More info ▸ Run anyway.

## Playing it

The first run is guided: nine tutorial steps teach movement, recycling the pod, raising the Main
Station, building the crafting bench, making the mining pistol and the survey scanner at it, and
then leave you to it.

Controls are bound to actions and keys are stored by physical position, so the layout follows your
keyboard — the movement cluster is `WASD` on QWERTY and `ZQSD` on AZERTY, and the build menu is the
key just left of it either way. The in-game legend always shows what your own hardware says.

| Input | Action |
| --- | --- |
| `W A S D` (AZERTY: `Z Q S D`) / arrows | Move |
| Left mouse | Mine what is under the cursor, or survey with the scanner equipped |
| `Q` (AZERTY: `A`) | Open the build menu |
| `E` | Interact — recycle the pod, open a machine |
| `R` | Turn the footprint a quarter turn while placing |
| `C` | Cycle to the next building of the same kind |
| `,` and `.` | Turn the camera a quarter round the world |
| `Tab` or `I` | Open the cargo hold |
| `M` | Open the map, once a milestone has paid for one |
| `X` | Disassemble mode: point at what should come apart |
| `G` | Drop a stack on the ground |
| `1`–`4` | Select an equipment slot |
| Mouse wheel | Zoom |
| `-` `=` `` ` `` | Slow down, speed up, back to normal |
| `Ctrl`+click | Send a whole stack where it obviously goes |
| `Shift`+click | Split a stack — half, a typed figure, or a slider |
| `Esc` | Cancel, or the pause screen |
| `F5` / `F9` | Save / continue from the newest save |

Keys are read by position, so the table above is the same set of keys wherever they are engraved.

## Versioning

`MAJOR.MINOR.PATCH` — major when a save made before it will not open, minor for a new system,
patch for fixes and tuning. What changed in each one is in [CHANGELOG.md](CHANGELOG.md), and the
same text is the notes on that version's release.

## Reporting something

Open an [issue](../../issues) with what you were doing and what happened. A screenshot and the
version out of the pause screen settle most of it.

---

This repository is where the game is handed out. The source lives elsewhere.
