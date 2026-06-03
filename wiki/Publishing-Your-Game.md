# Publishing Your Game

Assume your game or 3D application is ready and runs as desired with `WED -RUN`.
How does it become a distributable product on CD‑ROM?

## Creating runtime modules

Since V3.9, ACKNEX supports two operating systems: **DOS** and **Windows 95/98**
(via DirectX 5). You can ship your game with the DOS runtime module, the Windows
one, or both.

To create the runtime modules, open the **starting level** of your game with WED
and select:

- **PUBLISH** (lite/commercial) — produces `WRUN.EXE` and `WWRUN.EXE`.
- **COMPILE & PUBLISH** (professional) — produces `VRUN.EXE`, `WVRUN.EXE` and a
  `.WRS` resource file.

Additionally, copy `WWRUN.MDF` and `WWRUN.WDF` from the WED directory into your
game directory. Every time you compile a resource file for further levels, the
runtime modules are recreated — you only need the modules for your starting level
(others may be erased).

Test your game with these DOS commands:

| Command | Starts… |
|---------|---------|
| `WRUN levelname` | the DOS version |
| `WWRUN levelname` | the Windows version |
| `VRUN levelname` | the DOS version with a resource |
| `WVRUN levelname` | the Windows version with a resource |

### If your game won't start

- A level exceeds the restrictions for royalty‑free publication (see below).
- You forgot to add your main WDL file to the resource with the `BIND` keyword.
- `WWRUN.MDF` or `WWRUN.WDF` are missing from the runtime module directory.
- The resource file has a different name than the main WDL file.
- A Win95B version is running (a bug prevents launching Windows apps from a DOS
  box) — create and use a desktop link instead.
- DirectX 5 is not installed.

Then create the links/icon files to launch the game by double‑click, copy
everything into a single directory, and burn it to CD‑ROM. The game starts
directly from CD — no separate setup program is needed. A DirectX 5 installation
package can be obtained from Microsoft (observe Microsoft's license terms for
redistribution).

## Feature comparison (as of V3.9)

The three GameStudio versions are 100% upwards compatible (you can upgrade later).

| Feature | Lite | Commercial | Professional |
|---------|------|-----------|--------------|
| Max. objects per level (walls+things+actors) | 1500 | 1500 | 10000 |
| 2‑player mode | no | serial link | serial link & network |
| Multiple views | no | yes | yes |
| Max. resolution | 320×400 | 640×480 | 800×600 |
| Shutter glasses | no | no | yes |
| FLIC player | no | no | yes |
| Polygon models | no | wireframed | textured |
| CD audio | no | no | yes |
| `OUTPORT`/`INPORT` | no | no | yes |
| `LEVEL` instruction | no | no | yes |
| Resource compiler | no | no | yes |
| Runtime modules | yes | yes | yes |
| Email support | no | 90 days | unlimited |

If you need special features not yet supported, contact Conitec — they develop
ACKNEX engines adapted to specific purposes.

## Royalties

Normally you may distribute games or demos created with 3D GameStudio **royalty
free**. Exceptions are games at public places, arcade machines, events, fairs, TV
shows, etc.; or "big" games with polygonal objects (MDLs) or more than 1500
objects per level. For these, the WED‑created runtime module won't suffice — you
must purchase a special runtime module by giving Conitec the **'Magic Key'**
number WED displays for your starting‑level resource file. This module runs only
with the corresponding resource file; recompiling may change the Magic Key.

A single royalty payment is due to Conitec, depending on price and distribution:

| Type | End‑user price < $30 | $30 and above |
|------|----------------------|---------------|
| Demo | $100 | — |
| Shareware game (self‑distributed) | $1500 | $2000 |
| Advertising game | $2500 | $2500 |
| Commercial game * (national distribution) | $4000 | $7500 |
| Commercial game * (international distribution) | $8000 | $15000 |

\* A game sold by a distributor or in shops.

> Graphics in the **DEMO** directory are **not** royalty free — use them for
> practice only, not distribution. Graphics from the action game in the **SKAPH**
> directory may be used royalty free, but only for your own games made with 3D
> GameStudio.
