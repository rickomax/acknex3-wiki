# Actions

Actions give the objects of the game world the pseudo‑intelligent behaviour of
state machines. There are **global** actions, triggered at certain moments or by
a key press, and **object** actions, triggered by object events such as
`IF_NEAR` or `IF_FAR`. An action is a list of instructions executed one after
another.

ACKNEX is a **multitasking** engine: any number of actions execute
simultaneously, as in real life. The instructions form a simple programming
language for influencing gameplay.

```
ACTION Keyword { ... }
```

The following instructions may be used within an action.

## Assignment and arithmetic

### `SET`
```
SET [Object1.]Keyword1, [Object2.]Keyword2;
```
The most common instruction: assigns any keyword or object parameter a new value
or another keyword (so actions can even change or disable themselves by assigning
`NULL`). `Object1/2` is the keyword/synonym of the region, wall, actor or thing
whose parameters change; it must be placed in the level or at least defined in the
WDL before the action. If several objects share the keyword, only the **first**
placed in WED is changed (cf. `SET_ALL`). For `Keyword1`, a `FLAG` may be set (1)
or cleared (0).
```
SET tank.CEIL_TEX, water_tex;   // changes ceiling texture
SET bumper.FLAG3, 1;            // sets FLAG3 of thing "bumper"
SET IF_S, NULL;                 // de-assigns the S key
```

### `SET_ALL`
```
SET_ALL [Object1.]Keyword1, [Object2.]Keyword2;
```
Like `SET`, but affects **all** objects/regions with the given keyword/synonym,
not just the first. Positions (`X,Y,Z,X1,Y1,Z1,X2,Y2,Z2`) cannot be changed with
`SET_ALL`.

### `ACCEL`
```
ACCEL Skill, Number/Skill2;
```
Adds the number/skill to the first skill in an acceleration‑like way: the first
skill behaves like a speed accelerated/decelerated by the second parameter.
Inertia and friction are regulated by the predefined skills `INERTIA` (default 1)
and `FRICTION` (default 0.5).

### `RULE`
```
RULE Skill / [Object.]Keyword = expression;
```
Assigns the result of an arithmetic expression. Expressions may contain numbers,
skills, object parameters, brackets, operators, and these functions:

| Function | Meaning |
|----------|---------|
| `SIN(x)`, `COS(x)`, `TAN(x)` | Trigonometric functions |
| `ASIN(x)`, `ACOS(x)` | Inverse trigonometric functions |
| `LOG(x)`, `LOG10(x)`, `LOG2(x)` | Logarithm of x |
| `RANDOM(x)` | Random number between 0 and x |
| `SQRT(x)` | Square root of x |
| `SIGN(x)` | −1 if x<0, 1 if x>0, 0 if x==0 |
| `ABS(x)` | Absolute value of x |
| `INT(x)` | Integer value of x |
| `EXP(x)` | e to the power x |

In addition to `+ - * /`, these binary operators are supported (integers only):
`%` (modulo), `|` (bitwise OR), `^` (bitwise XOR), `&` (bitwise AND). Operators
combine with `=`: `+=`, `-=`, `*=`, `/=`.

Because of limited accuracy (fixed‑point, 3 decimals), avoid factors or
intermediate values below 0.01. Invalid operations (e.g. square root of a
negative) cause an engine abort!
```
RULE x = (a + 1) * b / c;
RULE z = x % 10;
RULE actor.ANGLE = ASIN(3*x + 0.5);
RULE x += 1;              // increase skill x by 1
RULE x += 2*TIME_CORR;    // increase x time-corrected by 2
```
`RULE` only assigns numerical values. For flags, textures, actions etc. use `SET`
/ `SET_ALL`.

## Control flow

### `IF` / `ELSE`
```
IF (expression) { instructions... }
IF (expression) { instructions... } ELSE { instructions... }
```
Executes the first block only if the expression is true (non‑zero); otherwise the
`ELSE` block (which may be omitted). Comparison operators:

| Operator | True if… |
|----------|----------|
| `\|\|` | either is true (OR) |
| `&&` | both are true (AND) |
| `!=` | both are not equal |
| `==` | both are equal |
| `<=` / `>=` | first ≤ / ≥ second |
| `<` / `>` | first < / > second |

Only parameters of the same type may be compared; flags use 1/0, non‑numerical
parameters use `NULL` for non‑existence. Comparisons evaluate to 0 (false) or 1
(true) and may be combined with brackets. **Use `==` for equality, not `=`!** `IF`
instructions may be nested.
```
IF (x<0) {
    RULE y=-1;
    RULE z=-1;
} ELSE {
    RULE y=1;
    RULE z=1;
}
```

### `WHILE` / `BREAK` / `CONTINUE`
```
WHILE (expression) { instructions... }
BREAK;
CONTINUE;
```
Repeats the block while the expression is true (evaluated at the start of each
repetition). Often used to modify a value slowly over several frames (e.g. opening
a door, moving an elevator) — insert a `WAIT` inside. `BREAK;` ends the loop;
`CONTINUE;` jumps to the next repetition.
```
REGION lift { /* ... */ IF_ENTER lift_start; }
ACTION lift_start {
    IF (lift.FLOOR_HGT >= 10) {        // already on top?
        WHILE (lift.FLOOR_HGT > 0) {   // then move down
            RULE lift.FLOOR_HGT += -0.2;
            WAIT 1;
        }
    } ELSE {
        WHILE (lift.FLOOR_HGT < 10) {  // move up
            RULE lift.FLOOR_HGT += 0.2;
            WAIT 1;
        }
    }
}
```

### `GOTO`
```
GOTO Label;
```
Jumps to a label (a keyword followed by a colon, placed between two instructions)
and continues from there.
```
IF (energy<50) { GOTO alarm; }
// ... further instructions
alarm:
PLAY_SOUND alert, 0.3;
FADE_PAL   red_pal, 0.5;
```

### `CALL` / `BRANCH` / `END` / `EXCLUSIVE`
```
CALL Action;
BRANCH Action;
END;
EXCLUSIVE;
```
- `CALL` — performs the given action immediately, then continues with the next
  instruction (after the CALLed action ends, or during a `WAIT` in it).
- `BRANCH` — *(documented in the v3.8 manual)* aborts the current action at this
  point and instead executes the given new action (it does **not** return). Note
  that `EACH_TICK` actions may not `BRANCH` to actions containing `WAIT`.
- `END` — terminates this action.
- `EXCLUSIVE` — terminates all other actions previously triggered by the same
  wall/thing/actor/region, preventing them from disturbing each other.

### `WAIT` / `WAITT`
```
WAIT Number / Skill;
WAITT Number / Skill;
```
`WAIT` stops the action for the given number of frame cycles; `WAITT` for the
given number of ticks (a fixed period of time). Other actions keep running during
the wait. **`EACH_TICK` actions may not contain `WAIT`**, nor may actions called
directly (`CALL`) or indirectly (`IF_HIT`) from them.

## Sound and music

| Instruction | Effect |
|-------------|--------|
| `PLAY_SOUND Sound, Volume, [Balance];` | Starts a sound at the given volume (0…1). `Balance` (−1…+1) shifts between stereo channels; alternatively an object keyword makes the sound emanate from that object's direction within its texture `SDIST`. |
| `PLAY_SOUNDFILE <filename>, Volume, [Balance];` | Like `PLAY_SOUND`, but plays directly from CD‑ROM/hard disk without using memory (no object Balance). |
| `STOP_SOUND;` | Aborts all running sounds. |
| `PLAY_SONG Music, Volume;` | Starts a background song, repeated until the next `PLAY_SONG`. Volume 0 switches music off. |
| `PLAY_SONG_ONCE Music, Volume;` | Like `PLAY_SONG`, but plays once. |
| `PLAY_CD Start, End;` | Plays CD audio tracks (DOS 6.0+/MSCDEX 2.2+). `Start`/`End` are track numbers (1…99). `Start`=0 stops; `End`=0 resumes; both 0 = nothing. Afterwards `CD_TRACK` holds the current track (call `PLAY_CD 0,0;` repeatedly to keep it updated). |

```
SOUND   crrahh, <crow.wav>;
TEXTURE bird_tex { BMAPS bird_map; SDIST 200; }
ACTOR   bird     { TEXTURE bird_tex; WAY bird_way; }
ACTION  bird_cry { PLAY_SOUND crrahh, 0.5, bird; }
```

## Animation (flics)

| Instruction | Effect |
|-------------|--------|
| `PLAY_FLIC Flic;` | Starts a full‑screen 320×200 FLI/FLC animation (professional only). Screen building and movement stop; the animation palette activates. The 16 global `EACH_TICK` actions keep running and may trigger sound at a precise frame via `FLIC_FRAME`. |
| `PLAY_FLICFILE <Filename>;` | Like `PLAY_FLIC` but reads from the file (memory freed after playing) — best for long animations. |
| `STOP_FLIC;` | Stops the current flic. |
| `BEEP;` | Plays a short note sequence on the PC speaker — useful during development to confirm an action triggered. |

## I/O ports (professional only)

```
INPORT Skill, adr;        // Skill = content of port #adr
OUTPORT Number/Skill, adr;// send value (0..255) to port #adr
```
Directly access I/O ports, e.g. to control external devices or implement new
input devices.
```
SKILL  input {}
SKILL  my_port { VAL 372; }
INPORT input, my_port;    // set input to the content of port #372
```

## Palette fading

```
FADE_PAL Palette, Number / Skill;
```
Sets or changes the current palette by fading. The factor (0…1) gives the fade;
gradually increasing it gives soft fading. A factor ≥1 makes the palette current.
If the palettes differ in color range, a factor ≥2 recomputes the shading table
(~0.3 s with `AUTORANGE`); ≥3 also recomputes the `BLUR` table if needed (~2 s).
```
PALETTE black_pal { PALFILE <BLACK.PCX>; }   // all colors black
PALETTE level_pal { PALFILE <MYPAL.PCX>; }   // define it AFTER black_pal!
SKILL   fade {}
ACTION  fade_out {                            // fade softly to black
    SET fade, 0;
    WHILE (fade<1) {
        RULE fade += 0.1;
        FADE_PAL black_pal, fade;
        WAIT 1;
    }
}
```

## MIDI

| Instruction | Effect |
|-------------|--------|
| `SETMIDI Channel, status;` `GETMIDI Channel, status;` | Set/get the device type for a MIDI channel (0…15). Status: 0 = FM (default), 1 = digital (DDK only), 2 = external (via Wave/MIDI port). |
| `MIDI_COM Statusb, data1, data2;` | Send data to a MIDI channel. `statusb` = (MIDI command × 16) + channel (0…15). |

Don't send too fast to an external MIDI channel — the MIDI port (31 kBit/s) is
not buffered.

## Strings and text

| Instruction | Effect |
|-------------|--------|
| `INKEY String;` | Copies keyboard input into the named string, waiting for `[Return]` (like `WAIT`; no SAVE/LOAD runs meanwhile). `[Esc]`/arrows abort (restoring the previous content). Edit with `[BackSpace]`, `[Del]`, cursor keys. If the string is shown in a `TEXT`, the input and a flashing cursor (char 127) are visible. `RESULT` is set on termination (−1 = Esc, 72 = Up, 73 = PgUp, 80 = Down, 81 = PgDn, else 0). Length via `STR_LEN`. |
| `SET_STRING String1, String2;` | Copies string 2 into string 1 (length not exceeding string 1's original). |
| `ADD_STRING String1, String2;` | Concatenates string 2 onto string 1 (don't exceed string 1's original length). |
| `TO_STRING String, Skill;` | Copies the digit representation of a skill into the string. |
| `SET_SKILL Skill, String;` | If the string holds a number, sets the skill to its numeric value. |
| `FIND Text, String;` | Sets the `OFFSET_Y` and `INDEX` of the `TEXT` to the string whose first characters match. `RESULT` = the `INDEX` (−1 if none found). Afterwards `LINES` and `SIZE_Y` hold the TEXT's character/pixel line counts. |
| `SET_INFO String, Object;` | Built‑in debugger: writes all interesting parameters of the object into the string (≥1000 chars) for on‑screen display. Available only in WED, not the runtime module. |
| `PRINTFILE "Name", Number/Skill;` | Names a text file for skill/string values (name = up to 5 chars + 3‑digit number + `.TXT`; default `PRINT0.TXT`). |
| `PRINT_VALUE Skill;` | Appends the skill (3 decimals) to the `PRINTFILE`. |
| `PRINT_STRING String;` | Appends the string (or a direct `"..."`) to the file; `\n` creates a line feed. |

## Screen capture

| Instruction | Effect |
|-------------|--------|
| `FREEZE Bmap, Number/Skill;` | Copies the current 3D image into an existing bitmap, reduced (factor 1…16) and without panels/overlays. Set `MOTION_BLUR` to 0 before freezing. Display it as a `PICTURE` in a panel (e.g. a saved‑game thumbnail). Save with `SAVE_INFO` no earlier than one frame later (insert a `WAIT`!). |
| `SCREENSHOT "Name", Number/Skill;` | Grabs the screen as a PCX file (name = up to 5 chars + 3‑digit number + `.PCX`). Set `MOTION_BLUR` to 0 one frame before. |

## Iterating objects (synonyms)

```
NEXT_MY;
NEXT_THERE;
NEXT_MY_THERE;
```
Change synonyms to modify several objects in a loop. `NEXT_MY` places the `MY`
synonym on the next object of the same name (wrapping at the end); `NEXT_THERE`
does the same for regions via `THERE`; `NEXT_MY_THERE` places `MY` on the next
object in the same region.
```
SET THERE, lift1;                  // first region "lift1"
loop1:
RULE THERE.FLOOR_HGT += 1;
NEXT_THERE;                        // next region
IF (THERE != lift1) { GOTO loop1; }
```

## Manipulating the game world

| Instruction | Effect |
|-------------|--------|
| `LOCATE [Object];` | Finds and re‑assigns the lost region of the player/thing/actor (e.g. after beaming a non‑`CAREFULLY` actor). Without a parameter, restores the player region (`HERE`). |
| `DROP Object;` | Places the thing/actor in the player's line of sight (`PLAYER_ANGLE`) at its `DIST` (or `MOUSE_ANGLE` if the pointer is active). For collision detection, the new position must be in the player's region — drop at `DIST` 0. |
| `SHOOT [Object];` | Triggers `IF_HIT` on the nearest object in the player's line of sight. `SHOOT_FAC` = gun power, `SHOOT_RANGE` = max range, `SHOOT_X`/`SHOOT_Y` = deviation (−1…+1). Afterwards `HIT_DIST` = distance, `RESULT = SHOOT_FAC * (1 - HIT_DIST/SHOOT_RANGE)`, and `HIT_X`/`HIT_Y` give the texture pixel hit (for bullet‑hole `ATTACH` textures). `IMMATERIAL` and transparent texture parts are not hit. With a thing/actor as `Object`, tests whether the player is visible from it and sets `SHOOT_ANGLE` (angle from object to player). `SHOOT_SECTOR` (default 2π) limits the actor's firing arc. |
| `EXPLODE Object;` | Triggers `IF_HIT` on all `FRAGILE` walls/things/actors within `SHOOT_RANGE` of the actor `Object`. `RESULT` ∝ hit power (0…1); `HIT_MINDIST` = nearest hit distance. If the hit object's `IF_HIT` itself contains `EXPLODE`, insert a `WAIT 1;` first to avoid mutual blocking. |
| `PUSH Number/Skill;` | Triggers `IF_HIT` with the closest visible object within the distance — to open doors or start elevators. `HIT_DIST`/`RESULT` set as with `SHOOT`. |
| `SHAKE Object;` | Tells the object its position changed — required after directly setting coordinates if the floor height or wall length changed as a result. |
| `LIFT Region, Number/Skill;` | Raises the Z values of all vertices, things and actors in the region (e.g. moving lift contents). Set `SAVE` on the walls/objects to save height changes. |
| `TILT Region, Number/Skill;` | Multiplies the Z values in the region — change a region's slope during gameplay. |
| `SHIFT Region, dx, dy;` | Shifts all vertices/things/actors of the region along X/Y (e.g. moving vehicles or rafts). No wall of the region may touch/cross another while moving! |
| `ROTATE Region, Number/Skill;` | Rotates all walls and objects of the region by the angle, around the region's `GENIUS`. The `GENIUS` may be outside the region and shared by several. No wall may touch/cross another while rotating! |

```
RULE  MY.X1 += 0.5;
SHAKE MY;
```
```
ACTOR wheel_center { TEXTURE any_tex; FLAGS INVISIBLE; }
REGION wheel {
    FLOOR_HGT 2;
    CEIL_HGT  20;
    FLOOR_TEX wood_tex;
    CEIL_TEX  stone_tex;
    GENIUS    wheel_center;
    EACH_TICK wheel_rotate;
    FLAGS     SAVE;          // to keep any changes
}
ACTION wheel_rotate {
    ROTATE wheel, 0.05;              // 0.05 rad ≈ 3°
    RULE   wheel.FLOOR_ANGLE += 0.05; // rotate the texture too
}
```

## Save, load, demos and level change

| Instruction | Effect |
|-------------|--------|
| `SAVE "Name", Number/Skill;` | Saves the game (file name = 5 chars + 3‑digit number + `.SAV`) in the `SAVEDIR`. |
| `LOAD "Name", Number/Skill;` | Loads, decompresses and decodes the score. With `LOAD_MODE` = 1 the resolution is not changed. |
| `SAVE_INFO "Name", Number/Skill;` | Like `SAVE`, but saves only LOCAL/GLOBAL skills, `FREEZE` bitmaps and `INKEY` strings. |
| `LOAD_INFO "Name", Number/Skill;` | Loads only `SAVE_INFO` values. |
| `SAVE_DEMO "Name", Number/Skill;` | Records player/actor movement and events to a demo file. |
| `PLAY_DEMO "Name", Number/Skill;` | Replays a demo (any key stops it). |
| `STOP_DEMO;` | Stops recording/replaying. |
| `MAP <Filename>;` | Level change: loads a new topography (`.WMP`) and resets player/actor starting positions. |
| `LEVEL <Filename>[, "Resourcename"];` | Complete level change (professional only): loads a new WDL script, its `MAPFILE` topography, and runs the new `IF_START`. Don't put the resource name in brackets (it would be compiled in!). With `LOAD_MODE` = 1, resolution is unchanged. All regions/objects/skills except GLOBAL skills are replaced. |
| `EXIT ["Text"/String];` | Ends the game, returning to DOS; optional text is shown on the DOS screen. |

## Standard (global) actions

Besides object‑event actions, these keywords specify standard actions:

| Keyword | Triggered when… |
|---------|-----------------|
| `<EACH_TICK Action, Action ...;` | A list of up to **16** actions performed after every frame cycle, in order. Change one with `SET EACH_TICK.n, Action` (n = 1…16); remove with `SET EACH_TICK.n, NULL`. May not contain `WAIT`, nor `CALL`/`BRANCH` to actions with `WAIT`. E.g. `EACH_TICK.16` for player movement. |
| `<EACH_SEC Action, Action ...;` | *(v3.8 manual)* A list of up to **16** actions performed every **second**, in order. Change one with `SET EACH_SEC.n, Action` (n = 1…16); remove with `SET EACH_SEC.n, NULL`. Suited to periodic tasks that needn't run every frame. |
| `IF_START Action;` | Performed at game start (palette change, title animation, songs, etc.). |
| `<IF_LOAD Action;` | Performed after loading a saved score — reset panels/skills (e.g. `MOVE_MODE`) used for saving. |
| `<IF_LEFT` / `<IF_MIDDLE` / `<IF_RIGHT Action;` | Left / middle / right mouse button (or joystick button 1 / 3 / 2). |
| `<IF_KLICK Action;` | Left‑clicking in the 3D window without hitting any object or panel. |
| `<IF_MSTOP Action;` | The mouse pointer is active and held stationary for ½ second. `MOUSE_CALM` sets the immobility tolerance (default 3); `MOUSE_MOVING` reports motion (0/1). |
| `<IF_ANYKEY Action;` | Any key is pressed. |
| `<IF_F1 … Action;` | The `[F1]` key. Likewise `IF_F2…IF_F12`, `IF_ESC`, `IF_TAB`, `IF_CTRL`, `IF_ALT`, `IF_SPACE`, `IF_BKSP`, `IF_CUU/CUD/CUR/CUL` (cursor keys), `IF_PGUP`, `IF_PGDN`, `IF_HOME`, `IF_END`, `IF_INS`, `IF_DEL`, `IF_PAUSE`, `IF_CAR` (full stop), `IF_CAL` (comma), `IF_ENTER`, `IF_0…IF_9`, `IF_A…IF_Z`. Redefine at runtime with `SET IF_F1, action;`. |

## Legacy instructions (v3.8)

The v3.8 manual documented a set of single‑operation arithmetic and conditional
instructions that were **superseded** in v3.9 by the more powerful `RULE`
(expression assignment with functions) and structured `IF (...) { } ELSE { }` /
`WHILE`. They are listed here for reference when reading older WDL scripts:

| Legacy instruction | Form | Replaced by |
|--------------------|------|-------------|
| `ADD` | `ADD Keyword1, Number/Keyword2;` | `RULE Keyword1 += ...;` |
| `ADDT` | `ADDT Keyword1, Number/Keyword2;` | `RULE Keyword1 += TIME_CORR * ...;` |
| `SIN`, `ASIN`, `SQRT` | `SIN Keyword1, Number/Keyword2;` (etc.) | `RULE Keyword1 = SIN(...);` |
| `AND` | `AND Keyword1, Number/Keyword2;` | `RULE Keyword1 = Keyword1 & ...;` |
| `RANDOMIZE` | `RANDOMIZE Keyword1, Number/Keyword2;` | `RULE Keyword1 = RANDOM(...);` |
| `IF_ABOVE` / `IF_BELOW` / `IF_EQUAL` | `IF_ABOVE Keyword, Number/Keyword2;` (runs only the next instruction) | `IF (a > b) { ... }` etc. |
| `IF_MIN` / `IF_MAX` | `IF_MIN Skill;` (skill at its `MIN`/`MAX` border) | `IF (skill <= skill_min) { ... }` |
| `ELSE` | `ELSE;` (runs the next instruction only if the previous `IF_…` was skipped) | the `ELSE { ... }` block of `IF` |
| `SKIP` | `SKIP Number;` (relative jump; negative skips backward) | `GOTO Label;` |
| `PLACE` | `PLACE Object;` | `SHAKE Object;` (identical purpose) |

> With `IF_EQUAL`, only flags or integer skills that have **not** been altered by
> rules may be compared — fixed‑point/time‑corrected values are rarely exactly
> equal, so use `IF_ABOVE`/`IF_BELOW` (or `>`/`<`) for those.
