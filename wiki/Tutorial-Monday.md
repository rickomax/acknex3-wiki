# Tutorial — Monday: Building a Minimal Level

> *Teach Yourself 3D Game Creating in 5 Days.* This 5‑day program is about doing
> WDL: how to build a professional 3D game. The assumption is that you have read
> the [[Introduction|reference manual]] and understood none of it — so we'll give
> you enough to start with. You should have `WED.EXE` (or `WEDS.EXE`/`WEDC.EXE`)
> and its auxiliary files in a directory on your DOS path. Run `SETUP` from the
> first diskette; it creates `c:\program files\GStudio` and the subdirectories
> `SKAPH` and `DEMO`. You work in the `DEMO` directory at first. Reboot after
> installation so the GSTUDIO directory is on your DOS path.

On the first day you'll learn, step by step, how to build a level. For simplicity
it will have only a few rooms. For quick starting and testing, work under DOS.

## Writing the WDL file

Use a DOS text editor to create your first WDL file:
```
EDIT MINI.WDL
```
Lines beginning with double slashes are comments. The file needs a **header**
defining the video resolution and the names of the WMP and WDL files:
```
///////////////////////////////////////////////
// Minimum level
// created 12.8.1997 (myself)
///////////////////////////////////////////////
VIDEO   X320x400;
MAPFILE <miniw.WMP>;
BIND    <mini.WDL>;
```
`BIND` is a hint for the professional compiler to compile this WDL into the final
game later. Letter case is irrelevant, but for clarity write all keywords and
predefined skills in CAPITALS and self‑defined names in lower case.

The header also needs `INCLUDE` statements to integrate more WDL files:
```
INCLUDE <move.WDL>;
```
`MOVE.WDL` (on the DEMO disk) contains the action `set_walking`, which initializes
player movement. Smooth movement actions are a little complicated (covered
[[Tutorial Tuesday|tomorrow]]) — for now, just use it. Define it as an `IF_START`
action so the player can move:
```
IF_START set_walking;
```

## The palette

Specify the color palette with its shading ranges, required for textures:
```
PALETTE pal1 {
    PALFILE <vrpal.pcx>;
    RANGE 16,16;  RANGE 32,16;  RANGE 48,16;  RANGE 64,16;
    RANGE 80,16;  RANGE 96,16;  RANGE 112,16; RANGE 128,8;
    RANGE 136,8;  RANGE 144,8;  RANGE 152,8;  RANGE 160,16;
    RANGE 176,16; RANGE 192,16; RANGE 208,16; RANGE 226,16;
    RANGE 240,16;
}
```
If you define more than one palette, the last one is loaded at game start. The
palette file and texture bitmaps are in the DEMO directory.

> The DEMO files are **not** royalty free — practice with them, but don't use
> them in your own games!

## Textures and a wall

A room needs at least one wall texture and one square floor/ceiling texture:
```
BMAP arc_map,    <wandtex.lbm>, 320, 0, 64, 128;
BMAP square_map, <bodentex.lbm>, 0, 0, 128, 128;

TEXTURE arc_tex {
    SCALE_X 10;
    SCALE_Y 10;
    BMAPS   arc_map;
}
TEXTURE square_ftex {
    SCALE_X 16;
    SCALE_Y 16;
    BMAPS   square_map;
}

WALL arc_wall {
    TEXTURE  arc_tex;
    FLAGS    PORTCULLIS;
    POSITION 1;
}
```
`SCALE_X`/`SCALE_Y` tell the engine the "zoom factor" — how many pixels fit into
one step. The `PORTCULLIS` flag with `POSITION 1` guarantees the texture stops
exactly at the ceiling (alternatively, use ALIGN CEILING in WED).

## Regions

Even for a single room you need two regions: one for the room, one for the
surrounding "massive" border:
```
REGION border {
    FLOOR_HGT 40;
    CEIL_HGT  40;
    FLOOR_TEX square_ftex;
    CEIL_TEX  square_ftex;
    CLIP_DIST 0;
}
REGION dungeon {
    FLOOR_HGT 0;
    CEIL_HGT  12;
    FLOOR_TEX square_ftex;
    CEIL_TEX  square_ftex;
}
```
The `border` region's floor and ceiling heights must be **equal** to give an
impenetrable closed wall on all sides. Since its interior is never seen, set its
`CLIP_DIST` to 0 to leave it out of rendering — a speed‑up in complex levels.

## Creating the topography in WED

Open a DOS window, change to the directory holding `MINI.WDL` (the DEMO
directory) and run:
```
WED MINI
```
(Lite owners use `WEDS`, commercial owners `WEDC`.) Since `MINIW.WMP` doesn't
exist, WED creates it. At start‑up you see an empty area with a blue point — the
player starting position.

1. Switch to **vertex mode** `[V]` and click several times **clockwise** around the
   player position, ~40 steps out. Each click places a green‑cross vertex. After a
   quadrangle/pentagon, drag a frame around all vertices to mark them.
2. Press `[Shift]-[Ins]`. WED connects the marked vertices with a closed line and
   switches to **wall mode**. Each line shows a green marking with a bar (the
   "nose"); if you placed the vertices clockwise, the noses point **inwards**.
   - If lines cross, you didn't place vertices in sequence — go back to vertex mode
     `[V]` and shift them. Crossed walls cause picture errors!
   - The nose marks the wall's right side. Use `[F]` to swap a wall's left/right
     sides if a nose points the wrong way.
3. Mark all walls, right‑click, and assign the only wall name `arc_wall`.
4. Press `[O]` (SCAN REGIONS) to check/scan the new region; WED enters **region
   mode**. Move the pointer inside the closed area (the border highlights),
   right‑click, and pick a region. Use the **[Alternate/Default]** button to choose
   default heights from the WDL definition, then **[OK]**. Assign `dungeon` to the
   room and `border` to the outside.
5. Verify in wall mode by touching walls (parameters show on the data panel).
   `border` must have equal floor/ceiling heights or the engine won't work. Run
   **[CHECK WALLS]** — faulty areas are marked.
6. Save with `[F2]`, then start a trial run with `[Shift-^]`. You're "beamed" into
   your level!

Default keys before you define your own: `[F5]` motion blur, `[F2]`/`[F3]`
save/recall position, `[F6]` screenshot, `[Shift-^]` toggle game/WED, `[F10]`
quit.

## Aligning textures

You'll notice the wall textures are cut unsightly at the edges. Back in WED,
activate wall mode `[W]`, mark all outside walls, and choose **WALL → ALIGN HOR**.
The `x y offs` value on the info panel changes — WED has shifted the textures
horizontally so they fit seamlessly. Restart with `[Shift-^]` to see. A "seam"
may remain where the first and last walls meet; you could fix it by changing the
last wall's length or the horizontal scaling.

## Adding a second room and a tunnel

Let's add a room with a lower floor and higher ceiling. Create the new room some
distance from the first. Then connect them by a tunnel: mark two opposite vertices
(one per room) and connect with `[Ins]`; do the same with two neighboring
vertices, forming a rectangular connecting region.

> Tip: In wall mode you can create a vertex by touching a wall and pressing `[S]`,
> which splits the wall and adds a vertex in the middle.

You now have three closed regions. Mark all walls and assign `arc_wall`. Press
`[O]` to register the new regions. Assign `dungeon` to the new rooms, but give the
tunnel an alternate **ceiling height of 8**, and the new room a **floor height of
−5** and **ceiling height of 20**. Re‑assign `border` (default heights) too, since
some of its walls changed. Regions needing re‑assignment show in red.

Restart the level (restarting is necessary each time you add regions). You now
face a tunnel instead of a solid wall — if not, you forgot to alter the new
regions' heights. Walls between regions of the same name and heights are shown
solid by the engine.

## A staircase

The new room's floor threshold is too high to climb back out. Build a stair: in
wall mode, touch the tunnel's outer wall and split it with `[S]`. Split both parts
again, and do the same with the other outer wall — now each wall has 4 pieces.
Connect each pair of opposite vertices, creating four regions across the tunnel.
Give them the `dungeon` region with floor heights differing by one step — a
staircase!

---

For higher functions (menus, actors, weapons, combat, puzzles), study
`VRDEMO.WDL` to learn how to build pillars, portals, elevators, outdoor areas and
underwater regions. Sample WDLs on the DEMO disk serve as suggestions. The next
days cover these topics in detail.
