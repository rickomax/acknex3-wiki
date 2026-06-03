# Skills & Role Playing Games

"Role Playing Games" is a traditional term for games in which players and actors
have individual properties (**skills**) — health, combat prowess, magic ability,
and anything else the author imagines. Skills are like variables in a programming
language. A number of skills are predefined (e.g. determining the player's
movement); any number more can be defined freely.

## Defining skills

```
SKILL Keyword { ... }
```

A skill definition may contain — in order:

| Keyword | Meaning |
|---------|---------|
| `TYPE Keyword;` | `LOCAL`, `GLOBAL`, or `PLAYER` (default). They differ only in save/load behaviour: `GLOBAL` skills are saved with `SAVE` and `SAVE_INFO`; `LOCAL` only with `SAVE_INFO`; `PLAYER` only with `SAVE`. |
| `<VAL Number;` | Starting value (default 0). |
| `<MIN Number;` | Lower margin — the value never drops below it. |
| `<MAX Number;` | Upper margin — the value never exceeds it. |

**GLOBAL** skills suit values kept across levels (prowess, score, severity); after
a game start or `LEVEL` change they can be reloaded from the last save with
`LOAD_INFO`. **LOCAL** skills always keep their last value when loading a score —
good for user settings (volume, resolution) that shouldn't change with an old
score.

Once defined, skill values are changed by `SET`, `RULE` and other instructions.

### Example — health that recovers over time
```
SKILL health     { VAL 100; }
SKILL recover    { VAL 0; }
SKILL rec_factor { VAL 0.5; }
ACTION permanent {
    WAITT 16;                                  // wait one second
    RULE health += rec_factor * recover - 0.005;
    RULE recover -= 1;                         // decrease value down to 0
}
ACTION ouch {
    RULE health -= 25;                         // hit – health drops
    RULE recover = 10;                         // 10 seconds to recover
}
EACH_TICK permanent;
```

## Predefined skills

The following skills may be evaluated in actions and — if marked `<` — also
changed. Skills marked `#` are set automatically.

### Screen and 3D window

| Skill | Meaning |
|-------|---------|
| `<SCREEN_WIDTH` `<SCREEN_HGT` | Width/height of the 3D window in pixels (default/max from `VIDEO`). Changeable in‑game: width divisible by 16, height by 2. Redefining at start sets a max window size (never exceeded later) to save memory. |
| `<SCREEN_X` `<SCREEN_Y` | Distance of the 3D window from the upper‑left screen corner (default 0). Horizontal must be divisible by 4. |
| `<ASPECT` | Height‑to‑width ratio of the rendered scene (0.1…10; default 1 = 1:1). |
| `<EYE_DIST` | Eye distance for 3D glasses (default 0.5 steps; needs `-3D_SIMEYE`). 0 disables the stereoscopic effect; stereo mode halves the frame rate. |
| `<SKY_OFFS_X` `<SKY_OFFS_Y` | Shift of all sky textures in pixels. `SKY_OFFS_X` must never be negative; increasing it periodically gives a "wandering cloud" effect. |
| `<MOTION_BLUR` | Motion blur parameter (0…1). Reduces resolution during movement for smoother motion on slow machines (default 0). |
| `<BLUR_MODE` | 1 = motion blur always on; 0.5 = on when player or mouse moves; 0 = default. |
| `<RENDER_MODE` | Rendering activity: `2` full (after screen‑size change or player displacement), `1` partial (after moving objects/walls/region heights), `0.5` default (renders per player movement), `0` suppressed (titles, credits, menus). Returns to 0.5 automatically each frame. |
| `<MOVE_MODE` | Movement of players/actors (default 1). `0.5` stops actor movement and object `EACH_TICK`; `0` also stops player movement and object/region events; `-0.5` stops all actions except keyboard events. |
| `<CLIPPING` | Object suppression outside `CLIP_DIST`: `0` (default) hide walls with one vertex outside; `0.5` hide walls with both vertices outside (unless `FAR`); `1` hide walls with both vertices outside or whose region's wall is invisible (use only with small regions). |
| `<LOAD_MODE` | 1 = `LOAD`/`LEVEL` won't change the screen resolution. |
| `<THING_DIST` `<ACTOR_DIST` | Ratio of the thing's/actor's `CLIP_DIST` to a wall's. `THING_DIST` 0…1 (default 1); `ACTOR_DIST` ≤ `THING_DIST`. |

### Automap

| Skill | Meaning |
|-------|---------|
| `<MAP_OFFSX` `<MAP_OFFSY` | Deviation of the map center from the 3D window center, in pixels (default 0). |
| `<MAP_CENTERX` `<MAP_CENTERY` | Like `MAP_OFFSX/Y` but in steps — the center stays fixed regardless of `MAP_SCALE`. |
| `#MAP_MAXX` `#MAP_MINX` `#MAP_MAXY` `#MAP_MINY` | Max/min X and Y of all level objects (auto‑calculated at game start). |
| `<MAP_EDGE_X1/X2/Y1/Y2` | Left/right limits of the on‑screen map, in pixels from the upper‑left (defaults = screen size). |
| `<MAP_SCALE` | Automap scale relative to the 3D window (default 0.9; 1 = exact fit). |
| `<MAP_MODE` | Map mode (0…1, default 0). `>0` shows explored regions/objects in the 3D window; `≥1` shows all objects (even unseen). |
| `<MAP_LAYER` | Overlay layer the map appears on (0…16, default 0). Higher layers draw over the map. |
| `<MAP_ROT` | 1 = map rotates with the player angle (radar‑like); only objects within `CLIP_DIST` are shown. |
| `<COLOR_PLAYER` | Color of the player symbol (default 7). |
| `<COLOR_ACTORS` | Default actor symbol color (default 3). |
| `<COLOR_THINGS` | Default thing symbol color (default 13). |
| `<COLOR_WALLS` | Default wall color (default 244); within a shading range, walls are antialiased. |
| `<COLOR_BORDER` | Default border‑wall color (default 244); antialiased within a shading range. |

### Mouse, keyboard and joystick

| Skill | Meaning |
|-------|---------|
| `<MOUSE_MODE` | 0 (default) no pointer; 1 shows the `MSPRITE`; 2 also stops mouse from changing the `FORCE_…` skills (move independently of the player). |
| `<TOUCH_MODE` | 1 (default) shows touched objects' `TOUCH` strings at the pointer. |
| `#MOUSE_MOVING` | 1 = moving, 0 = immobile for ¼ second. |
| `<MOUSE_CALM` | Max distance still counted as immobility (default 3). |
| `<MOUSE_TIME` | Ticks used to measure `MOUSE_CALM` (default 4). |
| `#MICKEY_X` `#MICKEY_Y` | Mouse movement in dots since the last frame; used to set `MOUSE_X/Y`. |
| `<MOUSE_X` `<MOUSE_Y` | Pointer position in pixels (from the upper‑left). Adjust `MIN`/`MAX` so the pointer can't leave the screen. |
| `#MOUSE_ANGLE` | Directional angle of the pointer in the level (for `DROP`); auto‑calculated from `MOUSE_X` and `PLAYER_ANGLE`; changeable to adjust the `DROP` angle. |
| `<TOUCH_DIST` | Max object distance to trigger `TOUCH`/`IF_TOUCH`/`IF_RELEASE`/`IF_KLICK` (default 100 steps). |
| `#TOUCH_STATE` | 0 none; 1 pointer on an object with `TOUCH` text; 2 object has an `IF_CLICK`; 3 both. |
| `#JOYSTICK_X` `#JOYSTICK_Y` | Joystick axis movement (−255…+255). |
| `#MOUSE_LEFT` `#MOUSE_MIDDLE` `#MOUSE_RIGHT` `#JOY_4` | Button state (0/1). The three mouse buttons map to joystick buttons 1, 3, 2. |
| `#KEY_ANY` | 1 if any key is pressed. |
| `#KEY_F1…KEY_F12`, `#KEY_ESC`, `#KEY_TAB`, `#KEY_SHIFT`, `#KEY_CTRL`, `#KEY_ALT`, `#KEY_SPACE`, `#KEY_BKSP`, `#KEY_CUU/CUD/CUR/CUL`, `#KEY_PGUP`, `#KEY_PGDN`, `#KEY_HOME`, `#KEY_END`, `#KEY_INS`, `#KEY_DEL`, `#KEY_PAUSE`, `#KEY_CAR`, `#KEY_CAL`, `#KEY_PLUS`, `#KEY_MINUS`, `#KEY_ENTER`, `#KEY_0…KEY_9`, `#KEY_A…KEY_Z` | State of the key (0/1). |

### Player input forces

| Skill | Meaning |
|-------|---------|
| `#FORCE_AHEAD` | Analog forward/backward value (≈ −1…+1) from mouse/joystick/cursor keys. |
| `#FORCE_STRAFE` | Left/right value (`[,]`/`[.]` keys, or `[Alt]` + sideways mouse/joystick). |
| `#FORCE_ROT` | Horizontal rotation of the line of sight (cursor left/right, sideways mouse/joystick). |
| `#FORCE_TILT` | Lift/lower the line of sight (`[PgUp]`/`[PgDn]`). |
| `#FORCE_UP` | Vertical movement, e.g. jumping/ducking (`[Home]`/`[End]`). |
| `<KEY_SENSE` | Movement sensitivity for cursor/`,`/`.`/`PgUp`/`PgDn`/`Home`/`End` keys (default 0.7). |
| `<SHIFT_SENSE` | Factor raising keyboard movement when `[Shift]` is held (default 2). |
| `<MOUSE_SENSE` | Mouse movement sensitivity (1 = medium, default). |
| `<JOY_SENSE` | Joystick movement sensitivity (1 = medium, default). |

### Sound and music

| Skill | Meaning |
|-------|---------|
| `<SOUND_VOL` | Sound‑effect volume (0…1), multiplied by each sound's volume. |
| `<MUSIC_VOL` | Music volume (0…1); 0 switches music off. |
| `<CDAUDIO_VOL` | CD audio volume (0…1) — DOS only (Windows 95 uses the mixer). |
| `#CHANNEL` | Channel (0…7) the last sound was assigned to; −1 if not played. |
| `#CHANNEL_0…CHANNEL_7` | Channel state: 0 free, 1 normal sound, 2 `SLOOP` sound. |
| `<PSOUND_VOL` | Relative volume of the player's (floor texture) sound (0…2, default 1). Played volume = `PSOUND_VOL × texture.SVOL`. |
| `<PSOUND_TONE` | Pitch of the player's sound (0…4, default 1). |
| `#FLIC_FRAME` | Current flic frame number — trigger sound/music at a precise frame in a global `EACH_TICK`. |
| `#CD_TRACK` | Current CD audio track; updated by `PLAY_CD`. |

### Lighting

| Skill | Meaning |
|-------|---------|
| `<AMBIENT` | Global light intensity (−1…+1, default 0) added to all texture/region ambients. |
| `<PLAYER_LIGHT` | Power of the light the player carries (0…1, 16 grades, default 1). |
| `<LIGHT_DIST` | Distance to the start of a shading area (default 10 steps). Objects closer are unaffected by `PLAYER_LIGHT`. |
| `#DARK_DIST` | Distance of the darkness area from the player; recalculated on every `PLAYER_LIGHT` change. |

### Player physics and movement

| Skill | Meaning |
|-------|---------|
| `<PLAYER_WIDTH` | Minimum distance from walls/objects for collision (default/min 1.2 steps). |
| `<PLAYER_SIZE` | Distance between the player's feet and eyes (default 3 steps); used for eye level and collision. |
| `<PLAYER_CLIMB` | Max height the player can overcome when changing region, e.g. a stair step (default 1.5 steps). |
| `<WALK_PERIOD` | Steps per period of rhythmic walk/swim motion (default 4). The floor texture sound repeats as footsteps. |
| `<WALK_TIME` | Time constant for the `WALK` motion (default 4 ticks). |
| `<WAVE_PERIOD` | Ticks per period of rhythmic time‑dependent movement (default 16). |
| `#WALK` | Current steering of the rhythmic walk motion (−1…+1). |
| `#WAVE` | Current steering of the rhythmic wave motion (−1…+1). |
| `<PLAYER_VX` `<PLAYER_VY` | Player speed along X/Y (steps per tick); auto‑adapted on collisions. |
| `<PLAYER_VZ` | Vertical player speed. |
| `<PLAYER_VROT` | Rotation speed of the line of sight (radians per tick). |
| `<PLAYER_TILT` | Tangent of the vertical line of sight. |
| `<PLAYER_ARC` | Eye focal length in radians (0.2…2.0) for zoom/"stoned" effects (default 1.0 ≈ 60°). |
| `<PLAYER_X` `<PLAYER_Y` | Player position on X/Y. Directly changing it does **no** collision detection — assign the new region manually! |
| `<PLAYER_Z` | Absolute eye level (steps) relative to the level's zero height. No collision on direct change! |
| `<PLAYER_ANGLE` | Horizontal line of sight (radians 0…6.28); 0 = positive X axis (east). |
| `#PLAYER_SIN` `#PLAYER_COS` | Sine/cosine of the viewing direction (for computing `PLAYER_VX/VY`). |
| `#PLAYER_SPEED` | Speed component along the line of sight (negative when moving backward, 0 when purely sideways). |
| `#ACCELERATION` | Acceleration along the line of sight (negative when moving backward). |
| `#PLAYER_HGT` | Player feet height relative to the region floor: `PLAYER_Z − PLAYER_SIZE − FLOOR_HGT`. |
| `#FLOOR_HGT` `#CEIL_HGT` | Real floor/ceiling height at the player position (from region heights, vertex Z and slope). |
| `#PLAYER_DEPTH` | Depth of an underwater BELOW region below the player. |
| `<PLAYER_LAST_X` `<PLAYER_LAST_Y` | Player position in the previous frame (for internal `STEP`/`WALK` calc). Change to move without `WALK` changes, e.g. riding a vehicle. |
| `#SLOPE_AHEAD` `#SLOPE_SIDE` | Floor ascent along and perpendicular to the line of sight (rise per step). |
| `#SLOPE_X` `#SLOPE_Y` | Floor ascent along X/Y. |
| `#MOVE_ANGLE` `#DELTA_ANGLE` | Real movement direction; and its angular difference from `PLAYER_ANGLE`. |

### Collision and rebound

| Skill | Meaning |
|-------|---------|
| `<IMPACT_VX` `<IMPACT_VY` | Collision vector — change of speed from colliding with a wall/object (or a `STICK` actor) in X/Y. Auto‑increased per collision; reset to 0 manually after use (pinball effect). |
| `<IMPACT_VZ` | Collision vector for floor/ceiling collisions. |
| `<IMPACT_VROT` | For a `STICK` actor bouncing an obstacle, the resulting change of angle relative to the player (gives the player a spin). |
| `#BOUNCE_VX` `#BOUNCE_VY` | Reflection vector on player–wall collision; add to player speed for realistic pinball bouncing. |
| `<FRICTION` | Friction coefficient for `ACCEL` (0…1, default 0.5). |
| `<INERTIA` | Inertia (mass) coefficient for `ACCEL` (>0, default 1). |

### Shooting and combat

| Skill | Meaning |
|-------|---------|
| `<SHOOT_RANGE` | Max object distance for `SHOOT` (default 500 steps). |
| `<SHOOT_SECTOR` | Valid angle range (radians) for `SHOOT` from object to player (default 2π). Set before each `SHOOT`; outside it, `HIT_DIST`/`RESULT` return 0. |
| `<SHOOT_FAC` | Hit‑power factor for `RESULT` in `SHOOT`/`EXPLODE` (0…1, default 1). |
| `<SHOOT_X` `<SHOOT_Y` | Horizontal/vertical deviation of the `SHOOT` direction (0…1). |
| `#HIT_DIST` | Distance to the last object hit by `SHOOT`/`EXPLODE` (0 if none). |
| `#HIT_MINDIST` | Distance of the nearest object hit by an explosion from the `EXPLODE` center. |
| `#RESULT` | The "power" of an event/hit/action, generally 0…1. |
| `#SHOOT_ANGLE` | Angle of the shooting actor relative to the player (for a bullet's initial angle). |
| `#HIT_X` `#HIT_Y` | Texture pixel hit by `SHOOT` (also valid in `IF_CLICK`/`IF_TOUCH`) — for placing bullet holes (`HIT_X` includes the wall `OFFSET_X`). |
| `<SKIP_FRAMES` | Frames skipped by invisible actors outside their `DIST` (default 5). |
| `<ACTOR_CLIMB` | Max step height actors can climb during `TARGET` movement (default 1). |
| `<ACTOR_WIDTH` `<THING_WIDTH` | Factor for actor/thing texture width in collision detection (default 1). |
| `#ACTOR_IMPACT_VX/VY/VZ` | Actor rebound from obstacles or floor/ceiling. Valid only with `CAREFULLY`, within the actor's `EACH_TICK`, right after a move (`MOVED` set). |
| `#ACTOR_FLOOR_HGT` `#ACTOR_CEIL_HGT` | Floor/ceiling height at the actor's position (same validity conditions as above). |

### Bullet‑hole example
```
BMAP    hole_map, <hole.pcx>;
TEXTURE hole_tex { BMAP hole_map; }
SYNONYM last_hitobj { TYPE WALL; }
ACTION  set_hole {
    SHOOT;
    IF (HIT == NULL) { END; }              // nothing hit?
    IF (last_hitobj != NULL) {             // something hit before?
        SET last_hitobj.ATTACH, NULL;      // remove hole from last wall
    }
    SET last_hitobj, HIT;                  // and place it onto the new one
    SET HIT.ATTACH, hole_tex;
    SET hole_tex.POS_X, HIT_X;
    SET hole_tex.POS_Y, HIT_Y;
}
```

### Strings and text

| Skill | Meaning |
|-------|---------|
| `#STR_LEN` | Characters of a `STRING` entered via `INKEY`. |
| `#LINES` | Character lines of a `TEXT` (set by `FIND`). |
| `#SIZE_Y` | Vertical pixel size of a `TEXT` (set by `FIND`). |

### Time and counters

| Skill | Meaning |
|-------|---------|
| `#TIME_CORR` | Time correction factor: 1 at 16 fps; drops at higher rates, rises at lower. Adapts speeds to frame rate. |
| `#TIME_FAC` | Inverse of `TIME_CORR` (higher at higher frame rates). |
| `<TICKS` | Increases by 1 every 1/16 second. |
| `<SECS` | Increases by 1 every second. |
| `<STEPS` | Increases by the number of steps the player covers. |

### Multiplayer

| Skill | Meaning |
|-------|---------|
| `#NODE` | Current node number in multiplayer (from the command line; commercial/professional only). |
| `<REMOTE_0` `<REMOTE_1` | Transmit information/events to a linked PC (commercial/professional only). |

### Testing skills (WED only — not in `VRUN.EXE`)

| Skill | Meaning |
|-------|---------|
| `#ERROR` | Error code for the current image: 0 none, 1 `CLIP_DIST` too small, 3 `NEXUS` too small, 5 missing bitmap, 6 missing region texture, 7 faulty sound, 8 incorrect synonym use, 9 missing region. Held until a new error; an error also clicks the PC speaker. |
| `<DEBUG_MODE` | 1 = stop after every frame until `[S]` (single‑step; also `[Ctrl][Alt][S]` or `-SST`). |
| `#MAX_DIST` | Distance of the farthest visible vertex within `CLIP_DIST` (to optimize region `CLIP_DIST`s). |
| `#ACTIONS` | Number of currently active actions. |
| `#ACTIVE_NEXUS` | Utilization of the nexus. |
| `#ACTIVE_TARGETS` | Number of actors with a `TARGET`. |
| `#ACTIVE_OBJTICKS` | Number of objects with running `EACH_TICK` actions. |
