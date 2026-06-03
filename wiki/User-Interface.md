# User Interface: Views, Panels, Texts and Overlays

At game start the 3D window fills the entire screen and no menu is visible. The
user interface can be freely defined with **overlays**, **panels**, additional
3D windows (**views**), and **texts**.

> *Notation:* `<` marks a value changeable during gameplay; `#` marks a value set
> automatically.

## Overlays

Graphic elements within the 3D window (cockpits, tools, weapons, inventory items)
are best defined as overlays. Overlays draw faster than panels, but have no
buttons or displays and need three times the memory. Up to **16** overlays may be
shown over one another.

```
OVERLAY Keyword { ... }
```

| Keyword | Meaning |
|---------|---------|
| `LAYER number;` | Draw order when overlapping other elements (higher = on top). Cannot be changed during the game. |
| `<POS_X x;` `<POS_Y y;` | Distance of the overlay's upper‑left corner from the upper‑left of the first 3D window — or, with `ABSPOS`, from the upper‑left of the screen. Changeable in‑game to shift the overlay. |
| `OVLYS Ovly;` | The keyword of the actual overlay (a bitmap assigned with `OVLY`). |
| `#SIZE_X` `#SIZE_Y` | Size of the overlay bitmap in pixels; evaluable in actions. |

**Flags:**

| Flag | Effect |
|------|--------|
| `ABSPOS` | `POS_X`/`POS_Y` refer to the screen corner, not the 3D window. |
| `<VISIBLE` | The overlay appears only when this flag is set. |

### The mouse pointer
```
<MSPRITE Overlay;
```
This overlay appears as the mouse pointer, moved with `MOUSE_X`/`MOUSE_Y`. The
overlay's `POS_X`/`POS_Y` define the "hot spot" pixel where clicking and touching
occur.

## Panels

Panels represent operating features, displays and images. Panels should be
**outside** the 3D window, or their background must be redrawn every frame (costly
— see the `REFRESH` flag).

```
PANEL Keyword { ... }
```

| Keyword | Meaning |
|---------|---------|
| `PAN_MAP Bmap;` | Background bitmap; its size determines the panel size. Redrawn on every position change. |
| `TARGET_MAP Bmap;` | *(v3.8 manual)* An off‑screen target bitmap the panel is drawn into (double buffering). It must be large enough to hold all panel elements, or the program crashes. Without it, the panel is drawn directly to the screen. |
| `LAYER number;` | Draw order when overlapping (higher = on top). Not changeable in‑game. |
| `<POS_X x;` `<POS_Y y;` | Distance of the panel's upper‑left from the screen corner. Changing it in‑game redraws the whole panel. |

**Flags:**

| Flag | Effect |
|------|--------|
| `TRANSPARENT` | The panel background and the `PICTURE`/`WINDOW` bitmaps are rendered transparently (color `#0` suppressed). |
| `REFRESH` | Panels and displays are redrawn every frame — needed for a panel over the 3D window or scrolling text. Without it, a display is redrawn only when its skill changes. Costly on large panels; sometimes an `OVERLAY` background is better. |
| `RELPOS` | `POS_X`/`POS_Y` are relative to the 3D window corner (not the screen); such panels are clipped at the 3D window edges. |

### Display elements

Within these definitions, `x,y` is the distance of the element's upper‑left corner
from the panel's upper‑left.

| Element | Syntax & meaning |
|---------|------------------|
| **Button** | `BUTTON x,y,BmapOn,BmapOff,ActionOn,ActionOff;` — Size = the `BmapOn` bitmap. Normally `BmapOff` shows; left‑click changes it to `BmapOn` and triggers `ActionOn`; release restores `BmapOff` and triggers `ActionOff`. Use `NULL` for `BmapOff` to make the button invisible when inactive. |
| **Slider** | `VSLIDER x,y,len,Bmap,Skill;` / `HSLIDER x,y,len,Bmap,Skill;` — Vertical/horizontal slider for entering a skill. `len` is the range in pixels; `Bmap` the draggable button. The skill's `MIN`/`MAX` set the value range (start = `MIN`, end = `MAX`). |
| **Bar graph** | `VBAR x,y,len,Bmap,fac,Skill;` / `HBAR x,y,len,Bmap,fac,Skill;` — Graphical skill display; the bitmap shifts by `fac × skill` pixels. The bitmap must be at least `len + fac×(skill max)` pixels in that dimension. |
| **Numeric** | `DIGITS x,y,len,Font,fac,Skill;` — Shows the integer part of `skill × fac` with `len` digits. `Font` has 11 chars (0–9 and space) or 128 ASCII chars. Leading zeros suppressed; negatives need a "minus" char in the font. Mind the limited skill accuracy. A one‑digit display can show pictures/symbols. |
| **Picture** | `PICTURE x,y,Texture,Skill;` — Animated picture. The texture may have up to 32 `SIDES`, animated by `CYCLES`, switched by the integer skill value (1 = first side, …). Bitmaps shown unscaled. |
| **Window** | `WINDOW x,y,dx,dy,Bmap,Skillx,Skilly;` — Shows a `dx×dy` cutout from `Bmap` (which must be ≥ `dx,dy`). `Skillx`/`Skilly` position the cutout (pixels from the bitmap's upper‑left); must stay inside the bitmap. |
| **Mask** | `MASK Overlay;` — *(v3.8 manual)* An overlay drawn over the panel after every change of a display. |
| **Mouse pointer** | `MSPRITE Overlay;` — Alternative mouse pointer within the panel. |
| **Click action** | `<IF_KLICK Action;` — Performed by left‑clicking the panel background, instead of the global `IF_KLICK`. |

## Texts and strings

Texts are used for menus, messages and dialogue with actors. The "raw" text is a
`STRING`; the formatted text is a `TEXT`.

```
STRING Keyword, "Text";
```
A character sequence. Line feeds within the text are `\n`; quotation marks `\"`.

```
TEXT Keyword { ... }
```

| Keyword | Meaning |
|---------|---------|
| `LAYER number;` | Draw order when overlapping (higher = on top). Not changeable in‑game. |
| `<POS_X x;` `<POS_Y y;` | Distance of the text's upper‑left from the screen corner. May be off‑screen (only the visible part shows). Changeable in‑game to scroll text. |
| `<SIZE_Y n;` | Display height in pixels (default/max = screen height). Only the range `POS_Y`…`POS_Y+SIZE_Y` is shown. |
| `<OFFSET_Y n;` | First pixel line of the text shown at `POS_Y` — scroll vertically pixel by pixel. |
| `STRINGS n;` | Maximum number of strings the text may contain (shown in succession). |
| `FONT Font;` | The character set (128 or 256 chars in ASCII order). |
| `#SCALE_X` `#SCALE_Y` | Size of a single character in pixels; evaluable in actions. |
| `<STRING Keyword1, Keyword2 ...;` | The actual text (one or more strings; set the max in `STRINGS` first). Changeable in actions; assigning `NULL` shows nothing. |
| `<INDEX n;` | Number of the string changed by the `STRING` keyword in an action (1…`STRINGS`, default 1). |

**Flags:**

| Flag | Effect |
|------|--------|
| `CENTER_X` / `CENTER_Y` | Center the text horizontally on `POS_X` / vertically on `POS_Y`. Otherwise left/top justified. |
| `CONDENSED` | Compress horizontally by 1 pixel per character (good for italics). |
| `NARROW` | Like `CONDENSED`, compressed further. |
| `<VISIBLE` | The text appears only when this flag is set. |

## Views (additional 3D windows)

For rear mirrors, missile cameras, or multiplayer, multiple 3D windows can be
shown (commercial/professional only). Every 3D window except the original is a
`VIEW`.

```
VIEW Keyword { ... }
```

| Keyword | Meaning |
|---------|---------|
| `LAYER number;` | Draw order between views (higher = on top). Note: panels, overlays and texts always draw over all 3D windows regardless of `LAYER`. |
| `<POS_X x;` `<POS_Y y;` | Distance of the view's upper‑left from the screen corner. |
| `<SIZE_X dx;` `<SIZE_Y dy;` | View size in pixels (same restrictions as `SCREEN_X/Y/WIDTH/HGT`). |
| `<GENIUS Keyword;` | A thing/actor — the view looks through its eyes. A moving actor should have the `FAR` flag set. |

**Flags:** `<VISIBLE` — the view appears only when set.

### Example — SVGA screen split into two 3D windows
```
ASPECT 0.5;                       // to get the right proportions
SKILL SCREEN_X      { VAL 0; }    // 1st view: upper half of screen
SKILL SCREEN_Y      { VAL 0; }
SKILL SCREEN_WIDTH  { VAL 640; }  // SVGA mode only!
SKILL SCREEN_HGT    { VAL 240; }
VIEW my_view {
    POS_X  0;                     // 2nd window: lower half
    POS_Y  240;
    SIZE_X 640;
    SIZE_Y 240;
    GENIUS my_actor;              // you'll look through his eyes
    FLAGS  VISIBLE;
}
```
