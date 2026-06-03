# Textures

Textures set the appearance of the world. There are textures for walls, floors,
ceilings, backgrounds (sky textures), things, actors, panels and overlays. They
are all defined as follows:

```
TEXTURE Keyword { ... }
```

A texture definition may contain — in the given sequence — the following keyword
assignments between curly brackets.

## Geometry and animation

### `SIDES`
```
SIDES Number;
```
Number of sides of the texture. Each side corresponds to one bitmap (or several,
for animated textures). The meaning of sides depends on the object:

- **Things and actors:** bitmaps change with the viewing angle, giving a spatial
  appearance. They may have an unlimited number of sides, counted clockwise; the
  first side is the frontal view.
- **Walls:** usually one side. Two‑sided wall textures change with the perspective
  — from the first vertex to the middle the first side shows, from the middle to
  the end vertex the second. With four sides, the first two are for the right
  side/front, the last two for the left side/back. More than four sides are useful
  with multi‑storied regions: the first four go to the uppermost region, the next
  four to the BELOW region, and so on.
- **Sky textures:** may have any number of sides, distributed evenly across the
  360° panorama (e.g. a 10‑side SKY texture uses 36° per bitmap). The horizontal
  scale derives from bitmap size and side count; there is no vertical scaling.

### `CYCLES`
```
CYCLES Number;
```
Number of animation phases (frames). Default 1.

### `FRAME`
```
FRAME Number;
```
Start frame of an animated MODEL texture, between 1 and the max number of frames
in the MDL file. Default 1.

### `BMAPS`
```
BMAPS Bitmap1, Bitmap2 ...;
```
List of bitmap keywords for the texture. All bitmaps of wall, floor or ceiling
textures must be the same size; actor/thing bitmaps may differ. The number of
bitmaps must equal `SIDES × CYCLES`. List the cycles of the front side first,
then all other sides counter‑clockwise around the object.

The keyword `NULL` may be used instead of a bitmap — but only for things, actors
or transparent walls — making that cycle/side invisible. For walls, at least one
bitmap must be non‑`NULL`.

**Size rules:** maximum width/height is 1024 pixels. LBM bitmaps must have an
even width. Wall textures can have any horizontal size; the vertical pixel count
must be a power of two (e.g. 64, 128, 256). Floor and ceiling textures must be
square — only 64×64, 128×128 or 256×256 are allowed. Backdrop (SKY) bitmaps may
have any horizontal size; the vertical size depends on the maximum visible
background section (see `PLAYER_TILT`). Usually the sky bitmap's lower edge
touches the horizon. Sky textures can be shifted via the object's `OFFSET_X` /
`OFFSET_Y` and the predefined skills `SKY_OFFS_X` / `SKY_OFFS_Y`. The sky is not
zoomed or repeated vertically; if the border exceeds the bitmap, the rest is
filled with the edge pixel line, so the first and last lines should be
monochrome.

### `FLIC` (professional only)
```
FLIC Flic;
```
Texture FLIC animation, an alternative to `BMAPS`. `Flic` is a keyword for a
FLI/FLC file. The texture size results from the animation size (same size rules
as `BMAPS`). The FLIC palette must match the level palette. Unless `ONESHOT` is
set, the animation loops.

### `TITLE`
```
TITLE String;
```
Gives a texture a string, displayed with the texture `FONT`. The string may be a
keyword or given directly (`TITLE "It's a title!";`). The same size restrictions
as a bitmap apply (e.g. font height for a one‑line wall title must be a power of
two). A texture with a `TITLE` must not be animated and must have no `BMAPS`.
```
TEXTURE title_tex {
    FONT  my_font;
    TITLE "That's a title!";
}
```

### `MODEL`
```
MODEL Model;
```
3D animated model, an alternative to `BMAPS`. Only assignable to things or
actors. The texture size results from the model vertex positions (in pixels); the
skin palette must match the level palette. Only one `DELAY` value is allowed.
Unless `ONESHOT` is set, the model animation loops. To use only part of the MDL
frames, use `FRAME` and `CYCLES`.
```
TEXTURE hero3d_tex {
    MODEL hero_mdl;
    DELAY 2;
    SOUND hero_snd;
}
```

### `DELAY`
```
DELAY Number, Number ...;
```
List of phase durations in ticks for each animation cycle. The number of values
must equal `CYCLES`. Without it, animation runs one tick per phase. Higher DELAY
values keep animation speed more constant across different frame rates.

### `MIRROR`
```
MIRROR Flag, Flag ...;
```
List of mirror flags (0 or 1) per texture side. A flag of 1 shows all bitmaps of
that side side‑inverted. The number of flags must equal `SIDES`.

### `OFFSET_X` / `OFFSET_Y`
```
OFFSET_X Number, Number ...;
OFFSET_Y Number, Number ...;
```
Individual horizontal/vertical shift (in pixels) for each bitmap of an animated
thing/actor texture. The number of values must equal `BMAPS`.
```
TEXTURE animthing_tex {
    SCALE_XY 10, 10;
    BMAPS    thing_map, thing_map, thing_map, thing_map;
    DELAY    6, 6, 6, 6;
    OFFSET_X 5, 10, 20, 40;
    OFFSET_Y 5, 10, 20, 40;
}
```

### `RANDOM`
```
RANDOM Number;
```
Maximum value (0…1, default 0) for a random delay factor added to every phase of
a texture animation.

## Scaling and lighting

### `SCALE_X` / `SCALE_Y` *(changeable)*
```
SCALE_X x;
SCALE_Y y;
```
Scaling in pixels per step. For floor/ceiling textures, `x` scales along X, `y`
along Y. Smaller values display the texture larger; larger values give higher
pixel resolution close up. Default is 16 pixels per step in each direction. Does
not apply to SKY textures.

### `AMBIENT` *(changeable)*
```
AMBIENT Number;
```
Brightness of the texture (−1…1, 32 levels, default 0). At 0 a wall texture is
only lit by the player's light source and the region's ambient light; at 1 the
texture shines brightly. Negative ambients reduce reflectivity (e.g. for shadowed
areas).

### `ALBEDO` *(changeable)*
```
ALBEDO Number;
```
The reflection factor of a texture (0…1, default 0) with respect to light from
`LIGHT_ANGLE` or a separate source (cf. `GENIUS`). Buildings, columns or ledges
appear more spatial with a texture `ALBEDO` of ~0.3.

### `RADIANCE`
```
RADIANCE Number;
```
Maximum brightness of a texture (0…1, default 0). With high ambients, textures
with `RADIANCE > 0` are colored with the palette's brightness color (`#2`).

## Sound

### `SOUND` *(changeable)*
```
SOUND Sound;
```
A keyword for a sound file (`.VOC` or `.WAV`). On floor textures the sound plays
rhythmically while moving; on ceiling textures it plays continuously throughout
the region if `SLOOP` is set.

### `SVOL` *(changeable)*
```
SVOL Number;
```
Maximum volume (0…1). Default 0.5.

### `SDIST` *(changeable)*
```
SDIST Number;
```
Critical horizontal range (steps) within which the sound is audible (default
100). Closer to the object, the louder it gets; with stereo cards you can locate
direction from left/right balance. At the texture, the sound plays at `SVOL`.

### `SVDIST` *(changeable)*
```
SVDIST Number;
```
Critical vertical range (steps) within which the sound is audible (default 0 =
inactive). Only effective with values >0 and with things/actors. If the player is
above or below the object's height plus `SVDIST`, the sound is inaudible.

### `SCYCLES`
```
SCYCLES Flag, Flag ...;
```
List of sound trigger flags (0 or 1) per animation phase. A flag of 1 plays the
texture sound on that cycle. The number of flags must equal `CYCLES`. Without it,
the sound starts on the first cycle.

## Attached textures

### `ATTACH` *(changeable)*
```
ATTACH texture;
```
Attaches an additional, transparent texture to a wall, thing or actor texture —
for paintings, inscriptions, bullet holes or shadows. Another texture may be
attached to the new one in turn.

For **walls**, any number of textures may be overlaid (e.g. a row of bullet
holes) — **but the chain must have an end!** A texture attached (directly or
indirectly) to itself crashes the engine. For **things/actors**, only one
`ATTACH` texture is allowed, and the `ATTACH` parameter of the actor itself (not
of its texture) must be used.

`ATTACH` textures may be `DIAPHANOUS` or semi‑transparent (`GHOST`). They may be
animated if the original is animated; their `CYCLES` must match. The original
texture's `DELAY` and `RANDOM` values apply. To animate only the attach, give the
original a "dummy animation" of identical bitmaps.

### `POS_X` / `POS_Y` *(changeable)*
```
POS_X number;
POS_Y number;
```
Pixel position of the `ATTACH` texture relative to the upper‑left corner of the
original. If both are 0 (default), it appears in the upper left.

To center an `ATTACH` texture on a wall independent of wall height and scaling
(rule of thumb):
```
POS_X = OFFSET_X + SIZE_X/2 - WA/2
POS_Y = HW - OFFSET_Y - HA/2 - (CEIL_HGT + FLOOR_HGT) * SCALE_Y/2
```
where `WA`/`HA` are the attach width/height, `OFFSET_X/Y` the wall offsets, `HW`
the wall texture height, `SCALE_Y` the wall scaling, `SIZE_X` the wall length,
and `CEIL_HGT`/`FLOOR_HGT` the ceiling/floor heights (all in pixels).

`PORTCULLIS` or `FENCE` walls are simpler: at wall `POSITION` 0 the attach
texture's vertical zero refers to the lower edge (shift upward with a negative
`POS_Y` to make it visible); at `POSITION` 1 it refers to the upper edge.

## Mouse response

The following parameters control how objects with the texture respond to mouse
touch/clicking. The skill `TOUCH_RANGE` sets the maximum reactive distance;
objects with the `IMMATERIAL` flag are invisible to the mouse. `ATTACH` textures
may respond too, but only the uppermost texture is relevant.

### `TOUCH` *(changeable)*
```
TOUCH string;
```
The string is displayed when the mouse pointer touches the object and the
predefined skill `TOUCH_MODE` ≥ 1. Maximum object distance is set by the skill
`TOUCH_DIST` (default 100). Appearance is influenced by the synonym
`TOUCH_TEXT`.

### `FONT`
```
FONT font;
```
Character set for the `TOUCH` or `TITLE` string.

### `IF_TOUCH` / `IF_RELEASE` / `IF_KLICK` *(changeable)*
```
IF_TOUCH action;
IF_RELEASE action;
IF_KLICK action;
```
- `IF_TOUCH` — triggered when the mouse pointer touches the texture.
- `IF_RELEASE` — triggered when the pointer leaves the object's texture.
- `IF_KLICK` — triggered by left‑clicking the object (instead of the common
  `IF_KLICK` action).

## Texture flags

```
FLAGS Keyword1, Keyword2 ...;
```

| Flag | Effect |
|------|--------|
| `ONESHOT` | The texture is not permanently animated; it shows the first phase. Setting `PLAY` on the assigned object animates it for one cycle, stopping at the last phase. |
| `GHOST` | Shown semi‑transparent on things, actors or transparent walls. |
| `DIAPHANOUS` | Like `GHOST` but diaphanous and not dithered. Works only with a `BLUR` palette and adds ~30% computing time. |
| `BEHIND` | The `ATTACH` texture appears behind the original (e.g. with things/actors). |
| `SHADOW` | The `ATTACH` texture appears at the actor's floor height. With a dark `DIAPHANOUS` texture this creates actor shadows. |
| `LIGHTMAP` | Maps light and shadows onto a wall texture (see below). |
| `SKY` | Parallax (not perspective) projection — shown as background (mountains, sky, horizon). Not zoomed, so it appears at infinite distance. Assignable to walls, floors and ceilings. |
| `WIRE` | (MODEL only) Displays the model as a non‑textured wireframe with `MAPCOLOR`. |
| `CLUSTER` | (MODEL only) Displays only the vertices of the model polygons. |
| `NO_CLIP` | Thing/actor textures are not cut off at a region's floor or ceiling. |
| `CLIP` | Thing/actor textures are cut off at the floor/ceiling, preventing objects being visible through the ceiling of BELOW regions. |
| `SLOOP` | Texture sounds play continuously in a loop, regardless of animation phase; volume and direction adapt continually. Computationally expensive — use sparingly. |
| `CONDENSED` | Compresses the mouse text (`TOUCH`) horizontally by 1 pixel (italics often look better). |
| `NARROW` | Like `CONDENSED`, compressed further. |
| `SAVE` | Required if the texture's parameters are changed by an action during the game. The texture is then saved with the score (`SAVE` command). |

### `SHADOW` example
```
TEXTURE shadow_tex {
    BMAPS shadow_map;                  // flat black spot (2nd color)
    FLAGS DIAPHANOUS, BEHIND, SHADOW;
}
ACTOR guy {
    TEXTURE guy_tex;
    ATTACH  shadow_tex;
}
```

### `LIGHTMAP` details and example
The `LIGHTMAP` bitmap must contain only colors `#0..#15` and `#241..#255`
(otherwise the engine aborts!). The transparent color `#0` does nothing; colors
`#1..#15` darken the wall texture; colors `#255..#241` brighten it (`#241` =
brightest). The darkest color uses palette color `#1`, the brightest palette
color `#2` (should be bright white or yellow).
```
BMAP torch_map, <torchlight.pcx>;
TEXTURE torch_tex {
    BMAPS torch_map;
    FLAGS LIGHTMAP;
    POS_X 40;
    POS_Y 10;
}
WALL stone_wall {
    TEXTURE stone_tex;
    ATTACH  torch_tex;
}
```

## Examples

### Multi‑sided sky texture (landscape with two towers in the east)
```
BMAP hills <huegel.lbbm>;
BMAP tower <turm.lbm>;

TEXTURE tower_landscape {
    SIDES 10;   // bitmap spreads over a 36° viewing sector
    BMAPS tower, hills, tower, hills, hills, hills, hills, hills, hills, hills;
    FLAGS SKY;
};
```

### Multi‑sided animated texture (a swimming fish)
A fish animated in 4 phases and drawn in 5 views (front, right‑front 45°, right
90°, right‑back 135°, back 180°); the remaining three sides are produced by
mirroring.
```
TEXTURE PIRAN_TEX {
    SCALE_X 20;        // 20 pixels per step
    SCALE_Y 20;
    SIDES   8;         // object has 8 sides
    CYCLES  4;         // and 4 phases
    BMAPS   PIRAN_0_1,   PIRAN_0_2,   PIRAN_0_3,   PIRAN_0_4,
            PIRAN_45_1,  PIRAN_45_2,  PIRAN_45_3,  PIRAN_45_4,
            PIRAN_90_1,  PIRAN_90_2,  PIRAN_90_3,  PIRAN_90_4,
            PIRAN_135_1, PIRAN_135_2, PIRAN_135_3, PIRAN_135_4,
            PIRAN_180_1, PIRAN_180_2, PIRAN_180_3, PIRAN_180_4,
            PIRAN_135_1, PIRAN_135_2, PIRAN_135_3, PIRAN_135_4,  // mirrored
            PIRAN_90_1,  PIRAN_90_2,  PIRAN_90_3,  PIRAN_90_4,
            PIRAN_45_1,  PIRAN_45_2,  PIRAN_45_3,  PIRAN_45_4;
    MIRROR  0,0,0,0,0,1,1,1;   // show last 3 sides mirrored
    DELAY   3,3,2,2;           // cycle lengths in ticks
    SCYCLES 1,0,0,0;           // sound starts at first cycle
    RANDOM  0.2;               // additional random delay
    SOUND   PIRAN_SND;
    SDIST   30;                // sound only audible within 30 steps
};
```

## Hints for designing textures

- You can produce bitmaps with paint software, 3D rendering, or scanning models;
  you will usually have to retouch them and adapt them to the level palette.
- All bitmaps for things, actors and walls (except sky, panel and overlay
  bitmaps) are zoomed — single rows of pixels are suppressed by distance, so
  adjacent pixels should not have great contrasts. Dithering, lattice structures
  and thin lines cause flickering and moiré and should be avoided. Sharp,
  high‑contrast edges should blend smoothly into the background on at least one
  side.
- Wall and floor textures should normally tile on all sides (except textures that
  cover a surface completely without repetition).
- A sky texture's vertical size should keep its top/bottom borders out of the
  field of vision; otherwise the upper border should be a single color (it
  repeats when leaving the field of vision).
- Thing/actor bitmaps should be cut out with at least one pixel of "air" on all
  sides.
- Bitmap size does not determine in‑game size (textures are scaled). A larger
  bitmap should add detail, not a larger image. Use large bitmaps for nearby
  things/actors and smaller ones for floors/ceilings. As a rule, far textures use
  `SCALE_X/Y` < 10; textures seen close up use 20 or higher.
- If a wall length, texture scale or light value is changed by an action, the
  texture must be re‑set with a `SET` instruction (see [[Actions]]).
- For LBM/BBM bitmaps in Deluxe Paint, width must be even. When saving LBM in
  Deluxe Paint II, deactivate the **old** button in SAVE AS. Most converters
  (GWS, Paintshop Pro) save LBM in the "old" format that ACKNEX cannot read —
  use PCX instead.

## Hints concerning sounds

WAV sounds must be 8‑bit (DOS) or 8/16‑bit (Windows 95) mono — ACKNEX handles the
stereo effect itself. WAV files should not contain special events (loop points,
regions, play lists). 11 kHz usually suffices; high‑frequency sounds (hissing,
rushing, splashing) may be sampled at 22 kHz.
