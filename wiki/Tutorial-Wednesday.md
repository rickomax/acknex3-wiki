# Tutorial — Wednesday: Portals, Elevators, and Waterfalls

Up to now we built static worlds. A 3D game is far more impressive with moving
parts — clockworks, secret portals opening, the player suddenly underwater. This
chapter builds a library of reusable WDL actions for moving parts of the world.

## Portals (texture‑animated doors)

The easiest movement is a door opening, done by animating a texture. We define a
wall that plays an animation as the player approaches and reverses it on
departure; near the end, the player can pass through. Example: a four‑winged
pneumatic bulkhead. (You draw the bitmaps and sample the sounds yourself.)
```
SOUND tuer_zisch, <tuer_auf.wav>;
BMAP gtuer_map,  <grauxit2.lbm>, 0,0,256,256;   // closed door
BMAP gtuer_auf1, <grauxit3.lbm>, 0,0,256,256;   // opening door
BMAP gtuer_auf2, <grauxit4.lbm>, 0,0,256,256;
BMAP gtuer_auf3, <grauxit5.lbm>, 0,0,256,256;
BMAP gtuer_auf4, <grauxit6.lbm>, 0,0,256,256;
BMAP gtuer_auf5, <grauxit7.lbm>, 0,0,256,256;
BMAP gtuer_auf6, <grauxit8.lbm>, 0,0,256,256;

TEXTURE grautuer_auftex {
    SCALE_XY 18,18;
    CYCLES   7;
    BMAPS    gtuer_map, gtuer_auf1, gtuer_auf2, gtuer_auf3, gtuer_auf4, gtuer_auf5, gtuer_auf6;
    DELAY    2,2,1,1,1,2,2;
    FLAGS    ONESHOT;
    SOUND    tuer_zisch;
    SCYCLE   2;
    SVOL     0.3;
    SDIST    50;
}
TEXTURE grautuer_zutex {
    SCALE_XY 18,18;
    CYCLES   6;
    BMAPS    gtuer_auf5, gtuer_auf4, gtuer_auf3, gtuer_auf2, gtuer_auf1, gtuer_map;
    DELAY    2,1,1,1,1,2,2;
    FLAGS    ONESHOT;
    SOUND    tuer_zisch;
    SCYCLE   1;
    SVOL     0.2;
    SDIST    50;
}
WALL grautuer {
    TEXTURE grautuer_auftex;
    FLAGS   SAVE;
    DIST    22;
    IF_NEAR grautuer_auf;
    IF_FAR  grautuer_zu;
}
```
Two textures are needed (opening and closing) since the animation direction is
preset; they differ only in phase order. The `ONESHOT` flag plays each animation
once. `DELAY` times make the wings speed up then slow down. The `IF_NEAR`/`IF_FAR`
events trigger actions; `SAVE` is needed so the door state survives save/load
(because the action changes `TEXTURE`).
```
ACTION grautuer_auf {
    SET  MY.TEXTURE, grautuer_auftex;
    CALL texdoor_open;
}
ACTION grautuer_zu {
    SET  MY.TEXTURE, grautuer_zutex;
    CALL texdoor_close;
}
ACTION texdoor_open {
    SET MY.DIST, 22;                       // hysteresis
    SET MY.TRANSPARENT, 1;
    SET MY.PLAY, 1;
    SET MY.EACH_CYCLE, texdoor_checkopen;
}
ACTION texdoor_checkopen {
    SET MY.PASSABLE, 1;                    // now the player may pass
}
ACTION texdoor_close {
    SET MY.DIST, 20;
    SET MY.PASSABLE, 0;                     // close door again
    SET MY.PLAY, 1;
    SET MY.EACH_CYCLE, texdoor_checkclose;
}
ACTION texdoor_checkclose {
    SET MY.TRANSPARENT, 0;
}
```
Splitting into **specific** (`grautuer_auf`) and **global** (`texdoor_open`)
actions is good practice: the global ones can be reused for any texture‑animated
door (place them in a shared WDL via `INCLUDE`). Since the actions are triggered
by wall events, `MY` refers to the triggering wall.

**`TRANSPARENT`** is set early so you can see through the gap as the door opens;
it's reset after closing (leave it set only when necessary — it costs render
time). **Hysteresis**: `IF_NEAR` raises `DIST` a few steps so `IF_FAR` triggers a
little later, preventing the door from oscillating at a critical distance.

> The door wall must have the **same region on both sides** (or `PASSABLE` won't
> let the player through). Remember `DIST` refers to the **nearest vertex** — on a
> 50‑step wall with `DIST` 20, the player could touch the center without triggering
> `IF_NEAR`.

## Elevators

Let a section of surface rise or fall at constant speed via an `EACH_TICK` action;
the move action carries the player along. Keep parameters (speed, levels) in the
region's universal skills (`SKILL1`..`SKILL8`):
```
REGION lift0_20 {
    FLOOR_HGT 0;
    CEIL_HGT  40;
    FLOOR_TEX boden_tex;
    CEIL_TEX  decke_tex;
    FLAGS     SAVE;
    SKILL3    0;     // lower level of elevator in steps
    SKILL4    20;    // upper level
    SKILL6    0.3;   // elevator speed in steps per tick
    IF_ENTER  lift_start;
}
```
`SAVE` is needed since the region changes.
```
SOUND rumpel, <rumpel.wav>;   // very short sound (<0.3 sec!)
SKILL lift_speed { }          // intermediate value

ACTION lift_start {
    IF (THERE.FLOOR_HGT < THERE.SKILL4)               // up or down?
    { SET THERE.EACH_TICK, lift_uptick; }
    ELSE
    { SET THERE.EACH_TICK, lift_downtick; }
}
ACTION lift_uptick {
    SET  lift_speed, THERE.SKILL6;
    RULE lift_speed = TIME_CORR*lift_speed;
    RULE THERE.FLOOR_HGT = THERE.FLOOR_HGT + lift_speed;
    IF (THERE==HERE)                                  // is player in elevator?
    { RULE PLAYER_Z = PLAYER_Z + 0.5*lift_speed; }
    IF (THERE.VISIBLE)                                // if elevator is visible,
    { SET RENDER_MODE, 1; }                           // re-render image constantly
    PLAY_SOUND rumpel, 0.3;
    IF (THERE.FLOOR_HGT >= THERE.SKILL4) {            // upper level reached
        SET THERE.FLOOR_HGT, THERE.SKILL4;            // nominal value
        SET THERE.EACH_TICK, NULL;
    }
}
ACTION lift_downtick {
    SET  lift_speed, THERE.SKILL6;
    RULE lift_speed = -TIME_CORR*lift_speed;
    RULE THERE.FLOOR_HGT = THERE.FLOOR_HGT + lift_speed;
    IF (THERE==HERE)
    { RULE PLAYER_Z = PLAYER_Z + 0.8*lift_speed; }    // suppress "shaking"
    IF (THERE.VISIBLE)
    { SET RENDER_MODE, 1; }
    PLAY_SOUND rumpel, 0.3;
    IF (THERE.FLOOR_HGT <= THERE.SKILL3) {
        SET THERE.FLOOR_HGT, THERE.SKILL3;
        SET THERE.EACH_TICK, NULL;
    }
}
```
Triggered by the region's `IF_ENTER`, so `THERE` is the region. `lift_start` picks
direction from the current height. When the player is in the lift
(`THERE == HERE`), partially moving `PLAYER_Z` with the floor suppresses jerky
elastic adjustments. `RENDER_MODE 1` keeps rendering even when the player is
still. `rumpel` plays every tick so it must be short (<⅓ s).

> Note: `END` does **not** terminate an `EACH_TICK` action — it just restarts next
> tick. Only setting the `EACH_TICK` to `NULL` stops it.

## Folding (swinging) doors

Instead of opening upward, doors can swing horizontally. Define the door as a
stand‑alone rectangular region whose four walls link to no other wall (or it would
move into them). Add an invisible `THING` as the **hinge**, placed inside the
region on a narrow side. Leave the opening area free — the door may not cross
region borders!
```
THING tuer_angel {              // must be within the region!
    TEXTURE dummy_tex;
    DIST    15;
    IF_NEAR swing_open;
    IF_FAR  swing_close;
}
REGION holz_tuer {
    FLOOR_TEX holz_tex;
    CEIL_TEX  holz_tex;
    FLOOR_HGT 10;
    CEIL_HGT  10;
    FLAGS     SAVE;
    SKILL3    0;      // initial angle, door closed (0..3.14)
    SKILL4    1.57;   // final angle, door open (1.57 = 90°)
    SKILL5    0;      // current angle of door
    SKILL6    0.1;    // speed of door in radians per tick
    GENIUS    tuer_angel;
}
```
Opening works like the elevator, but changes the **angle** via `ROTATE` instead of
`FLOOR_HGT`:
```
SOUND knarz, <knarz.wav>;   // very short sound (<0.3 sec!)
SKILL door_speed { }
SKILL angle      { }

ACTION swing_open {                           // open door
    SET THERE.SKILL7, THERE.SKILL4;           // set final angle
    IF (THERE.SKILL4 < THERE.SKILL3)          // decrease or increase angle?
    { SET THERE.EACH_TICK, swing_dectick; }
    ELSE
    { SET THERE.EACH_TICK, swing_inctick; }
}
ACTION swing_close {                          // close door
    SET THERE.SKILL7, THERE.SKILL3;
    IF (THERE.SKILL4 < THERE.SKILL3)
    { SET THERE.EACH_TICK, swing_inctick; }
    ELSE
    { SET THERE.EACH_TICK, swing_dectick; }
}
ACTION swing_inctick {
    RULE   door_speed = TIME_CORR*THERE.SKILL6;
    RULE   THERE.SKILL5 += door_speed;        // increase current angle
    ROTATE THERE, door_speed;
    PLAY_SOUND knarz, 0.3, THERE.GENIUS;
    SET    RENDER_MODE, 1;
    IF (THERE.SKILL5 >= THERE.SKILL7)         // final angle reached?
    { SET THERE.EACH_TICK, NULL; }
}
ACTION swing_dectick {
    RULE   door_speed = -TIME_CORR*THERE.SKILL6;
    RULE   THERE.SKILL5 += door_speed;
    ROTATE THERE, door_speed;
    PLAY_SOUND knarz, 0.3, THERE.GENIUS;
    SET    RENDER_MODE, 1;
    IF (THERE.SKILL5 <= THERE.SKILL7)
    { SET THERE.EACH_TICK, NULL; }
}
```
`SKILL7` temporarily stores the final angle. The door arrests at the final angle if
it overshoots, just like the elevator.

## Underwater regions

A water region is two regions stacked, the upper above water, the lower below:
```
REGION under_water {            // must be defined BEFORE the water region!
    FLOOR_HGT -5;
    CEIL_HGT  1;
    FLOOR_TEX mud_tex;
    CEIL_TEX  underwater_tex;
    IF_ARISE  regio_arise;
}
REGION water {
    FLOOR_HGT 1;                // same as under_water's CEIL_HGT!
    FLOOR_TEX water_tex;
    CEIL_HGT  20;
    CEIL_TEX  sky_tex;
    BELOW     under_water;
    IF_DIVE   regio_dive;
}
```
When the player enters the water region he sinks to the lower region's bottom. An
`IF_DIVE` action on the floor makes it permeable, putting the player underwater
immediately. Add effects for atmosphere:
```
PALETTE blue_pal { PALFILE <blue.bbm> }
SKILL underwater { VAL 0; }

ACTION regio_dive {
    SET RENDER_MODE, 1;
    IF (underwater) { END; }        // already underwater? then end
    SET underwater, 1;
    FADE_PAL blue_pal, 0.7;         // set underwater palette
    RULE PLAYER_ARC += 0.3;         // change point of view
    RULE my_size -= 0.3;            // hysteresis
    RULE PLAYER_SIZE -= 0.3;
    RULE PLAYER_Z -= 0.3;
    CALL set_diving;                // diving mode
}
ACTION regio_arise {
    SET RENDER_MODE, 1;
    IF (underwater==0) { END; }     // already above water?
    SET underwater, 0;
    FADE_PAL blue_pal, 0;           // switch off underwater palette
    RULE PLAYER_ARC -= 0.3;         // normalize point of view
    RULE my_size += 0.3;            // hysteresis
    RULE PLAYER_SIZE += 0.3;
    RULE PLAYER_Z += 0.3;
    CALL set_swimming;              // swimming mode
}
```
The `blue_pal` palette gives a milky‑blue, foggy look; `PLAYER_ARC` adds a slight
"frog‑eye" effect. The **hysteresis** lines lower the player's eye height on diving
so he doesn't constantly dive/arise at the surface. `regio_arise` reverses
everything when the player touches the surface from below. (`MOVE.WDL` in VRDEMO
contains move actions for diving and swimming.)
