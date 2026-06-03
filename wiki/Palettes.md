# Palettes

Because the pixels of 256‑color bitmaps contain no real colors — only color
*numbers* — your game needs a **palette** to define which visible colors the
numbers represent. The palette is taken from a reference graphics file. Your
paint program stores the palette into the graphics file, so in theory each
texture could have its own palette; but textures with different palettes look
"psychedelic". **All textures in a level should share the same palette.** Most
paint programs offer a "load palette" / "adapt palette" function for this.

Palettes are responsible not only for color representation but also for
**shading** — the darkening or "fogging" of textures depending on surrounding
light and distance from the player. To perform shading, the palette colors must
be sorted into groups — **shading ranges** — of declining brightness (or
increasing "fogginess"). Textures at great distance or in utter darkness are
given the second color of the palette.

## Special colors

The first three colors have a special meaning:

| Color | Meaning |
|-------|---------|
| `#0` | Transparency |
| `#1` | Darkness / fogginess — the color for wide distances (INFINITY color) |
| `#2` | The "brightness" color of the light sources in the level |

Only one palette is active at a moment, but you can switch or fade between
palettes at any time. The **last** defined palette in the WDL file is used at game
start.

## `PALETTE` definition
```
PALETTE Keyword { ... }
```
A palette of 256 colors may be created from any `.PCX`, `.BBM` or `.LBM` file.
The definition may contain, in the order given, the following assignments:

### `PALFILE`
```
PALFILE <Filename>;
```
The 256‑color graphics file whose colors form the basic palette. **Under Windows
the first color (`#0`) must be black and the last (`#255`) must be white.**

### `RANGE`
```
RANGE s, l;
```
A shading range — a group of identical colors with continually declining
brightness. `s` is the number of the initial color (1…254), `l` the number of
colors in the range (up to 255). The initial color must be the brightest, the
last the darkest (nearly equal to the INFINITY color `#1`).

Up to **24** shading ranges may be defined per palette. If no ranges are defined,
shading is switched off (slightly faster rendering). Colors outside all ranges are
not shaded, so they can represent light sources.

### `FLAGS`
```
FLAGS Keyword1, Keyword2 ...;
```
A list of flags determining palette properties:

| Flag | Effect |
|------|--------|
| `HARD` | "Hard" shading: the transition between bright and dark is smaller; colors are shifted rather than converted, giving darkened textures greater contrast so detail is better perceived. |
| `AUTORANGE` | Automatically shades all colors that fall within any `RANGE`, even if unsorted, according to their similarity to the INFINITY color (`#1`). Allows fogging effects even with unsorted palettes (manual sorting still gives more controllable results). |
| `BLUR` | Enables a "softer" motion blur effect: during movement the image is shown slightly out of focus, moderating the pixeling of high‑contrast textures. Also enables semi‑transparent (DIAPHANOUS) textures. Downside: game start‑up and palette changes take ~2 seconds longer per BLUR palette. If all level palettes are similar, setting `BLUR` on the main palette alone suffices. |

### Example
```
PALETTE my_pal {
    PALFILE <raw.pcx>;       // unsorted palette, 2nd color = black
    RANGE   16, 224;         // shade all colors from #16 up to #240
    FLAGS   BLUR, AUTORANGE; // don't bother about ranges
}
```
