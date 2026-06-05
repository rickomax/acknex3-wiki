# Tips & Tricks

> Practical techniques collected from the **ACKNEX User's Magazine (AUM)**,
> issues 1–3, contributed by readers (Markus Fabian, M. Bhatty, George Pirvu and
> the editors).
>
> **Note on syntax:** these articles predate the Spring 1998 (v3.9) release, so
> the examples use the older single‑operation instructions `ADD`, `IF_ABOVE`,
> `IF_EQUAL`, etc. In v3.9 these are superseded by `RULE`/`IF`/`WHILE` (see the
> *Legacy instructions* table in [[Actions]]). The techniques themselves still
> apply — e.g. `ADD x,1;` is `RULE x += 1;`, and `IF_ABOVE a,b; <one instruction>`
> is `IF (a > b) { ... }`.

## BELOW regions: a bridge

A `BELOW` region sits directly under another, letting you build bridges, tunnels,
balconies and multi‑storey rooms. The lower region must be **defined before** the
upper one, and its ceiling must not be higher than the upper region's floor (or
you get a "Region X belows itself!" error). The WED editor only sees the
uppermost region — it ignores whatever is below.

```
REGION unter_der_bruecke {   // the BELOW region — defined FIRST
    FLOOR_HGT 0;
    CEIL_HGT  11.5;
    FLOOR_TEX fussboden_tex;
    CEIL_TEX  holzdecke_tex;
}
REGION bruecke {
    FLOOR_HGT 12;            // 0.5 steps above the BELOW region's ceiling
    CEIL_HGT  20;            // ceiling of the hall the bridge leads through
    FLOOR_TEX fussboden_tex; // the two regions may use completely different textures
    CEIL_TEX  steindecke_tex;
    BELOW     unter_der_bruecke;
}
```
This makes a bridge 0.5 steps thick. Caveat: the wall on the side of stacked
regions is not aligned (the `PORTCULLIS` and `POSITION` flags aren't evaluated
there), so the side walls of stacked regions need manual alignment tricks.

## Rotating a region

Use a **GENIUS** actor as the center of rotation and the `ROTATE` command in an
`EACH_TICK` action. Place the (visible or invisible) actor inside the region: in
the middle for a centered spin, or on an edge to swing it like a door. It can even
be placed outside the region.

```
ACTOR angle_actor { TEXTURE black_tex; }

REGION unter_cubus {          // the BELOW region (we see the cube from below)
    FLOOR_HGT -2;
    CEIL_HGT  8;
    FLOOR_TEX kassetten_tex;
    CEIL_TEX  kassetten_tex;
    GENIUS    angle_actor;     // the rotation center
    AMBIENT   -0.3;            // darker than its surroundings → a shadow
}
REGION cubus {
    FLOOR_HGT 18;
    CEIL_HGT  22;
    FLOOR_TEX kassetten_tex;
    CEIL_TEX  kassetten_tex;
    BELOW     unter_cubus;
}

ACTION rotate_cube {
    ROTATE unter_cubus, 0.05;             // 0.05 rad ≈ 3°
    ADD    unter_cubus.CEIL_ANGLE, 0.05;  // rotate the ceiling texture too
}
```
Only `CEIL_ANGLE` is rotated (so the shadow stays put); for a rotating *stage*
you would rotate `FLOOR_ANGLE` instead. Bind it to a key as a start/stop toggle:
```
IF_R start_rotation;
ACTION start_rotation {
    SET EACH_TICK.1, rotate_cube;
    SET IF_R, stop_rotation;     // redefine R to stop
}
ACTION stop_rotation {
    SET EACH_TICK.1, NULL;
    SET IF_R, start_rotation;    // redefine R to start again
}
```

## Beam actions (teleporting the player)

A hidden beam action saves you from constantly dragging the player in WED, and can
move the player between stories that aren't worth building as BELOW regions.
Remember to disable such "cheats" before shipping!

**1. Simple jump** — beam to an invisible marker thing's position:
```
THING beamer { TEXTURE transparent_black_tex; FLAGS INVISIBLE; }
IF_B beam_me;
ACTION beam_me {
    SET  PLAYER_X, beamer.X;
    SET  PLAYER_Y, beamer.Y;
    SET  HERE, beamer.REGION;                  // set the target region
    RULE PLAYER_Z = HERE.FLOOR_HGT + PLAYER_SIZE;
    SET  IF_B, beam_me2;                        // chain to the next beam (a toggle)
}
```

**2. Relative beam** — add/subtract from the player's position so the move feels
seamless (useful at an `IF_ENTER`/`IF_DIVE`, where you don't know the exact entry
point):
```
ACTION beam_me_up_scottie {
    ADD  PLAYER_Y, 1000;          // shift 1000 steps along Y
    SET  HERE, zielregion;        // set the target region
    RULE PLAYER_Z = HERE.FLOOR_HGT + PLAYER_SIZE;
}
```

**3. Remembering where you came from** — use a `REGION` synonym plus skills to
return (e.g. an ESC service level):
```
SYNONYM LastRegion { TYPE REGION; }
SKILL Last_x {}  SKILL Last_y {}  SKILL Last_z {}

ACTION beam_me3 {
    SET Last_X, PLAYER_X;   SET Last_Y, PLAYER_Y;   SET Last_Z, PLAYER_Z;
    SET LastRegion, HERE;                  // remember the old region
    SET PLAYER_X, beamer3.X;
    SET PLAYER_Y, beamer3.Y;
    SET HERE, beamer3.REGION;
    RULE PLAYER_Z = HERE.FLOOR_HGT + PLAYER_SIZE;
    SET IF_ESC, beam_me_back;
}
ACTION beam_me_back {
    SET PLAYER_X, Last_X;   SET PLAYER_Y, Last_Y;   SET PLAYER_Z, Last_Z;
    SET HERE, LastRegion;                  // restore everything
    SET IF_ESC, beam_me3;
}
```
Alternatively, the `LOCATE` instruction finds the player's region automatically —
but it is expensive on large WMP files, so reserve it for loading a whole new
level.

## A clickable mouse pointer (TOUCH / IF_KLICK)

Draw a pointer bitmap, assign it as an overlay, and toggle it on/off:
```
OVLY    mouse_ovr, <arrow.pcx>;
OVERLAY mouse_sprite { OVLYS mouse_ovr; }
MSPRITE mouse_sprite;

ACTION mouse_toggle {
    RULE MOUSE_MODE += 2;
    IF (MOUSE_MODE > 2) { SET MOUSE_MODE, 0; END; }
    WHILE (MOUSE_MODE > 0) {
        WAIT 1;
        RULE MOUSE_X += 2*MICKEY_X;        // move it with the mouse
        RULE MOUSE_Y += 2*MICKEY_Y;
    }
}
```
Give an object a `TOUCH` string (shown when the pointer hovers it) and an
`IF_KLICK` action (run when clicked):
```
FONT    show_font, <show.pcx>, 12, 12;     // 11, 128 or 256 chars only!
STRING  screwdriver_str, "This is a screwdriver!";
TEXTURE show_me {
    SCALE_XY 16,16;
    BMAPS    yep;
    FONT     show_font;             // a TOUCH string needs a font
    TOUCH    screwdriver_str;
}
TEXTURE door_open_tex {
    SCALE_XY 16,16;
    BMAPS    door_tex;
    IF_KLICK open_door;            // run when the object is clicked
}
```

## A drag‑and‑drop inventory panel

Store the inventory icons in a `FONT` whose "digits" are pictures, and show them
with `DIGITS`. Each `*placed` skill picks which icon (0 = empty) shows in each
slot:
```
FONT objfont, <objfont.pcx>, 28, 22;   // 11 "chars": 0=empty,1=greenkey,2=cake,3=fish,4=bluekey
PANEL big_panel {
    PAN_MAP bigpanel_map;
    POS_X 0;
    POS_Y 355;                         // 320×45 panel fits an X320x400 screen
    DIGITS 70,20,3,font01,1,health;
    DIGITS 100,20,3,font01,1,iq;
    DIGITS 145,11,1,objfont,1,bluekeyplaced;
    DIGITS 187,11,1,objfont,1,greenkeyplaced;
    DIGITS 229,11,1,objfont,1,cakeplaced;
    DIGITS 271,11,1,objfont,1,fishplaced;
    FLAGS  REFRESH;
    IF_KLICK place_objects;            // click the panel to drop the carried item
}
```
Define each item as a (non‑moving) **actor** — actors give you 8 skills and an
`OVERLAY` (used as the carried mouse cursor). On click, the item disappears from
the world and `MSPRITE` becomes its overlay:
```
ACTOR greenkey_act {
    TEXTURE greenkey_tex;
    OVERLAY greenkey_ovl;
    SKILL1  50;        // IQ bonus
    SKILL2  0;         // health bonus
    SKILL8  1;         // which cursor this item maps to
}
ACTION takeobject {
    IF (cursornormal == 0) { END; }    // already carrying something
    // ... set the matching cursor skill from MY.SKILL8 ...
    SET MY.INVISIBLE, 1;
    SET MY.PASSABLE, 1;
    SET MSPRITE, MY.OVERLAY;           // cursor becomes the item
    RULE iq += MY.SKILL1;
    RULE health += MY.SKILL2;
    SET cursornormal, 0;
}
ACTION place_objects {
    SET greenkeyplaced, cursorgreenkey;   // update the panel slots
    SET cakeplaced, cursorcake;
    SET fishplaced, cursorfish;
    SET bluekeyplaced, cursorbluekey;
    SET MSPRITE, mouse_sprite;            // cursor back to the pointer
    SET cursornormal, 1;
}
```

## Underwater region with an air bar

Stack two BELOW regions (under‑water and out‑of‑water) so the first's ceiling
height equals the second's floor height; trigger actions on `IF_DIVE`/`IF_ARISE`.
A `VBAR` panel shows remaining air, counting up with `SECS`:
```
REGION belowroom01 {        // under the water
    FLOOR_HGT 0;  CEIL_HGT 45;
    FLOOR_TEX floor_tex;  CEIL_TEX uwater_tex;
    AMBIENT 0.4;
    IF_ARISE region_arise;
}
REGION room01 {             // out of the water
    FLOOR_HGT 45; CEIL_HGT 275;   // floor == below region's ceiling!
    FLOOR_TEX water_tex;  CEIL_TEX ceil_tex;
    AMBIENT 0.7;
    BELOW belowroom01;
    IF_DIVE region_dive;
}
BMAP airbar_map, <airbar.pcx>;
PANEL airpanel {
    VBAR 280,30,20,airbar_map,2,secspassed;   // shrinks as time passes
    FLAGS REFRESH;
}
ACTION counteron {                 // on diving
    SET SECS, 0;
    SET EACH_TICK.05, update_panel;
    SET PANELS.10, airpanel;
}
ACTION update_panel {
    IF (secspassed > 9) { BRANCH drown; }   // out of air
    RULE secspassed = SECS;
}
ACTION counteroff {                // on surfacing
    SET secspassed, 0;
    SET EACH_TICK.05, NULL;
    SET PANELS.10, NULL;
}
```

## Zoom in / out without image errors

Zooming changes `PLAYER_ARC`. Two corrections give a clean result: keep `MAX` at
**1** (the start value), and nudge the player's rotation slightly while zooming so
the texture projection re‑resolves instead of distorting:
```
SKILL PLAYER_ARC { MIN 0.1; MAX 1; }
// in the EACH_TICK move action:
IF (KEY_INS) {
    RULE PLAYER_ARC -= 0.03;
    RULE PLAYER_VROT += 0.001;     // tiny rotation hides projection errors
}
IF (KEY_DEL) {
    RULE PLAYER_ARC += 0.03;
    RULE PLAYER_VROT -= 0.001;
}
```

## Creating sprites from a video camera ("3D scanner")

Two‑dimensional bitmap objects (things/actors) are tedious to draw and a common
source of errors. Instead, use a video camera as a "3D scanner": record the real
object against a neutral (green/blue) background from the angle that matches your
game's viewpoint, digitize the clip, export the chosen frame to a graphics file,
clean it up in a paint program (size, stray pixels), and paste it into your game's
object bitmap. Walking *around* the object while recording yields the multiple
`SIDES` of a spatial sprite. (AUM recommended a capable capture board and video
software such as Ulead Studio or Adobe Premiere, plus Paint Shop Pro for cleanup.)

## Sourcing tileable wall textures

A reader's tip: model‑railway scenery cardboards (e.g. Faller's stone, brick and
clinker sheets, ~25×12.5 cm) are already **tileable** — scan one and adapt it to
your level palette for instant seamless wall textures.
