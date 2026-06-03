# World Definition Language (WDL)

## 1. Introduction

The elements and objects of your virtual world — walls, regions, actors and so
on — are described in a **WDL script** (WDL = *World Definition Language*). If
the game consists of several levels, a separate WDL script can be written for
each level.

There are different types of definitions — e.g. for regions, walls, actors,
things, textures, bitmaps and sounds. Higher‑order objects (like walls) may
contain lower‑order elements (like textures), which themselves may contain
bitmaps or sounds. You describe the properties of an object with predefined
parameters. You may assign **actions** to objects, triggered by **events** that
act upon the object; these actions may in turn change the object's properties.

### Units

The space and time units of the virtual world are the **step** and the **tick**:

- One **step** equals one screen diameter (around 40 cm).
- One **tick** equals the time between two frame cycles on a 486 PC — about
  1/16 second.
- **Angles** are stated in radians (0 to 6.28), counted counter‑clockwise; 0
  corresponds to the positive direction of the X axis.

### Special characters

| Syntax | Meaning |
|--------|---------|
| `... ;` | Semicolon terminates an assignment |
| `... , ...` | Commas separate parameters |
| `{ ... }` | Parameter or instruction lists go between curly brackets |
| `" ... "` | Text goes between quotation marks |
| `< ... >` | File names (without path) go between angle brackets |
| `# ...` | Comment until end of line |
| `// ...` | Comment until end of line |
| `/* ... */` | Comment block |

## 2. Keywords

All elements of the world and their properties are addressed by **keywords**. In
WDL, keywords play a similar role to variables in a programming language. Some
keywords are predefined; keywords for files, objects, textures and so on may be
defined by the user.

Keywords are generally defined and assigned a value by a line such as:

```
TYPE Keyword value;
```
or
```
TYPE Keyword { ... more keywords ... }
```

`Keyword` is any name of up to **30 letters**. Names must not begin with a number
nor contain special characters except the underscore `_`. You may not assign the
same keyword to different objects. `TYPE` is the keyword type and `value` is the
assigned content — a number, more keywords, or a complete list of keywords
between curly brackets `{...}`. The definition may span several lines and is
closed with a semicolon.

### User‑defined keyword types

| Type | Purpose |
|------|---------|
| `ACTION` | A real‑time action — a list of instructions triggered by an event, which may change the properties of walls, things, actors or regions. |
| `ACTOR` | An animated object; a state machine with behaviour primitives and action/reaction capabilities. |
| `BMAP` | A bitmap (graphics file) in `.PCX`, `.WEX`, `.LBM` or `.BBM` format, or a rectangular section of one. |
| `FLIC` | An animation file in `.FLI`, `.WEX` or `.FLC` format (professional only). |
| `FONT` | A bitmap interpreted as a character set to represent numbers or text. |
| `MODEL` | A textured 3D model file in `.MDL` format. |
| `MUSIC` | A song file in `.CMF` or `.MID` format. |
| `MSPRITE` | The mouse pointer overlay. |
| `OVERLAY` | A screen overlay (cockpits, weapons, tools) shown over the rendered scene. |
| `OVLY` | A bitmap assigned to an overlay. |
| `PANEL` | A display panel with bar graph or numeric displays. |
| `REGION` | A region in the map bordered by walls. |
| `SKILL` | A numeric property. |
| `SOUND` | A sound file in `.WAV` or `.WEX` format. |
| `STRING` | Raw text, e.g. for descriptions, menus or dialogues. |
| `SYNONYM` | A blank "template" to fill in. |
| `TEXT` | Formatted text. |
| `TEXTURE` | An animated texture, assignable to floors, ceilings, walls, things or actors. |
| `THING` | A tool, weapon or other inanimate object. |
| `WALL` | A wall separating regions. |
| `WAY` | The way an actor covers within a level. |
| `VIEW` | A 3D window into the virtual world. |

### Assignable values

| Value | Meaning |
|-------|---------|
| `Number` | An integer of up to five digits (e.g. `12345`) or a fixed‑point number with up to three decimals (e.g. `-12345.678`). |
| `Number, ... Number` | A list of numbers separated by commas. |
| `Keyword` | Another previously defined keyword. |
| `Keyword, ... Keyword` | A list of previously defined keywords. |
| `"Text"` | A text string. Line feeds are written in C notation as `\n`. |
| `<Filename>` | The content of a file. The file type comes from the extension: `.LBM`, `.BBM`, `.PCX` (bitmaps/palettes), `.FLI`, `.FLC` (animations), `.MDL` (models), `.MID` (songs), `.IBK` (instruments), `.WAV` (sound effects), `.WEX` (graphics/sound/animation read from outside the resource). All files in angle brackets are compiled into a world resource by the compiler. |
| `{ ... }` | A complex description: a list of parameter assignments. May span any number of lines, closed by `}`. |

> **Notation used in this reference:** predefined keywords are written in
> CAPITALS; user‑defined keywords, parameters and numbers in *italics*. Keywords
> whose values can be changed during gameplay through actions are marked with an
> arrow (`<`). Keywords whose values automatically change after each frame (and
> can be evaluated by actions, but may not be predefined) are marked with a
> rectangle (`#`).

## 3. Files

Keywords may be assigned to files to define bitmaps, animations, sounds or songs.
Files that belong to the game and must be compiled are stated **without a path,
in angle brackets** `<...>`.

### `BMAP`
```
BMAP Keyword, <Filename>;
BMAP Keyword, <Filename>, x, y, dx, dy;
```
Assigns a 256‑color bitmap (`.PCX`, `.LBM` or `.BBM`) to the keyword. A `.PCX`
file can be renamed to `.WEX`, in which case it is first looked for outside the
resource. The second form assigns a rectangular section: `x,y` is the upper‑left
corner, `dx,dy` the width and height in pixels. Omitting the coordinates assigns
the entire bitmap.

### `OVLY`
```
OVLY Keyword, <Filename>;
OVLY Keyword, <Filename>, x, y, dx, dy;
```
Like `BMAP`, but assigns an **overlay** (for cockpits, weapons, mouse pointers,
etc. drawn over the rendered scene). Overlays use two to three times the RAM of a
bitmap but draw much faster. The number of horizontal pixels must be divisible
by 4.

### `FONT`
```
FONT Keyword, <Filename>, width, height;
```
Assigns a character set. `width, height` is the size of each character in pixels
(all characters must be the same size). The bitmap can contain **11 characters**
(0–9 and space) for numeric displays, or the **128/256 characters** of the PC /
ASCII set for displays and texts. The alphanumeric sequence must match the PC
(ASCII) character set; characters may appear over several lines. The bitmap size
must be exactly 11×, 128× or 256× the given character size.

### `MODEL`
```
MODEL Keyword, <Filename>;
```
Assigns a 3D polygonal model in `.MDL` format. You can use a freeware/shareware
MDL tool (e.g. QMe or Meddle) to create the files and import DXF meshes. For
skin palettes, use PCX2PAL to create a raw 768‑byte `PALETTE.LMP` for QMe, or
Paintshop Pro to create a JASC `.PAL` file for Meddle.

The MDL file must obey these restrictions: no more than **1024 faces** (fewer is
better); a single skin (adapted to the level palette); triangles only; polygons
must not penetrate each other; certain "critical" polygons (especially long,
narrow ones) may need to be split if drawn in the wrong order.

### `SOUND`
```
SOUND Keyword, <Filename>;
```
Assigns a sound file in `.WAV` format. A `.WAV` file can be renamed to `.WEX`
(looked for outside the resource first). Sample rate may be 11 or 22 kHz. The DOS
engine (including WED) accepts only 8‑bit `.WAV` files; the Windows engine also
accepts 16‑bit files.

### `MUSIC`
```
MUSIC Keyword, <Filename>;
```
Assigns a song file in MID format. The internal predefined instruments
correspond to the General MIDI standard.

### `FLIC`
```
FLIC Keyword, <Filename>;
```
Assigns an animation file in `.FLI` or `.FLC` format. A `.FLI` file can also be
named `.WEX` (looked for outside the resource first) — useful for very large
`.FLI` files, which play a little faster as separate files outside the resource.

## 4. Predefined Keywords

The following keywords, which must appear at the **beginning** of the WDL script,
define the basic modes for graphics and sound.

### `VIDEO`
```
VIDEO Keyword;
```
Sets the screen resolution. Predefined resolution keywords:

| Keyword | Resolution |
|---------|-----------|
| `320x200` | VGA 320×200 |
| `X320x240` | VGA 320×240 (mode X) |
| `X320x400` | VGA 320×400 (mode X) |
| `S640x480` | SVGA 640×480 (commercial version or above) |
| `S800x600` | SVGA 800×600 (professional version) |

Low resolutions are for fast action games; the high "S" resolutions for
adventures or commercial applications. Note that the "X" resolutions are not
supported by the DirectX drivers of some graphics cards (Windows version).

### `NEXUS`
```
NEXUS Number;
```
Sets the size of the **nexus** — the internal data structure for picture
rendering. The size depends on the number of objects, walls and regions visible
consecutively. A bigger nexus allows more complex scenes but needs more memory.
The default value is 14. A nexus too small for a scene produces an engine error.

### `CLIP_DIST`
```
CLIP_DIST Number;
```
Default value for the region `CLIP_DIST` (usually 1000 steps). Only walls and
objects entirely within this distance from the player are rendered. Individual
regions may define their own `CLIP_DIST`, which can speed up rendering of complex
levels by up to 30%. Walls beyond `CLIP_DIST` are rendered monochromatically
using palette color 1. A `CLIP_DIST` too small may cause image errors — sometimes
a desirable effect (e.g. distant areas dissolving into darkness or fog).

### `LIGHT_ANGLE`
```
LIGHT_ANGLE number;
```
Defines the direction of an infinitely distant light source (range 0…2π). All
walls with an `ALBEDO`‑type texture reflect this light according to their
orientation. `0` corresponds to east (sunrise); `3.145` to west (sunset).

### `DITHER`
```
DITHER number;   // 0 or 1
```
Switches the dithering effect off or on (default = 1 = on). Dithering produces
smoother light and shadow ranges.

### `IBANK` / `DRUMBANK`
```
IBANK <Filename>;
DRUMBANK <Filename>;
```
Loads the instrument bank for MIDI songs. `Filename` is an AdLib instrument file
in `.IBK` format. Percussion is output through MIDI channel 10. Without
`DRUMBANK`, only 5 standard drums are available; without `IBANK`, internal
General MIDI compatible instruments are used.

### `MIDI_PITCH`
```
MIDI_PITCH Number;
```
Number of octaves by which the pitchbend parameter changes the pitch of MIDI
songs (1…12; default 2).

### `INCLUDE`
```
INCLUDE <Filename>;
```
Reads additional WDL definitions from a separate WDL file, then continues
scanning the original file. This lets definitions common to all levels be
collected in shared WDL files. Any number of `INCLUDE` keywords are allowed in
the main WDL file.

### `BIND`
```
BIND <Filename>;
```
The given file is included in the game by the compiler, e.g. for changing levels.
You may give any number of `BIND` files.

### `MAPFILE`
```
MAPFILE <Filename>;
```
Defines the corresponding WMP topography file. This keyword must be in the main
WDL and cannot be read via `INCLUDE`.

### `SAVEDIR`
```
SAVEDIR "Dirname";
```
Names the directory for saved games (created on first save if missing).
Backslashes must be written in C notation (doubled), e.g. `"C:\\gstudio\\mygame"`.
This may be overridden at runtime by the `-DIR` option of `VRUN.EXE`.

### `PATH`
```
PATH "Dirname";
```
Additional files (bitmaps, sounds, MIDI) are searched first in the current
directory, then in the path given here (again, double the backslashes). Up to 16
`PATH` keywords may be specified, searched in order.

### `DEFINE` / `UNDEF`
```
DEFINE NewName[, OldName];
UNDEF Name;
```
`DEFINE` (like the `-d` command line option) lets you rename keywords and
parameters and conditionally include/exclude WDL lines. Renaming does not affect
function but makes large actions more readable. `UNDEF` "undefines" a `DEFINE`d
keyword for all following lines.

```
DEFINE MyNumber, -3456;
DEFINE HitEffect, SKILL3;
ACTION kill {
    SET RESULT, MY.HitEffect;
    SET MyActor.HitEffect, MyNumber;
}
```

Characters like `{} ,;()<>` cannot be redefined, and `DEFINE`s may not be nested.

### `IFDEF` / `IFNDEF` / `IFELSE` / `ENDIF`
```
IFDEF Name;
IFNDEF Name;
IFELSE;
ENDIF;
```
These skip WDL lines depending on previous `DEFINE`s or the `-d` command line
option. Lines between `IFDEF` and `ENDIF` are skipped if the parameter was not
`DEFINE`d. Lines between `IFNDEF` and `ENDIF` are skipped if it *was* `DEFINE`d.
`IFELSE` reverses the skipping.

```
DEFINE hires;          // alternatively start WRUN -d hires
IFDEF hires;
    VIDEO S640x480;    // SVGA high resolution
IFELSE;
    VIDEO X320x400;    // VGA mid resolution
ENDIF;
```
