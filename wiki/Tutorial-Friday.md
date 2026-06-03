# Tutorial — Friday: Panels, Menus, Overlays

So far we built only the virtual world and its inhabitants. This chapter covers
functions that, while outside the world, are still part of the game — controlling
it and communicating with the user.

## Texts

To put written text on screen, start with raw text (`STRING`):
```
STRING my_string, "This is a text!!";
STRING my_string_3,
    "This is the first line,
this is the second,\nand this, finally, is the third line.";
```
A string may span several WDL lines or use `\n` for line feeds. To display it you
need a **font** — a bitmap of little images for each character, in the order of the
first 128 PC (ASCII) characters, all the same size. (The demo disk includes
`PANFONT.PCX` as an example.)
```
FONT standard_font, <panfont.pcx>, 8, 10;
```
The last two parameters are the width and height (pixels) of each character — so
the bitmap must contain exactly 8 × 10 × 128 = 10240 pixels. Now make a `TEXT`:
```
TEXT my_text {
    POS_X  20;
    POS_Y  40;
    FONT   standard_font;
    STRING my_string;
}
ACTION show_my_text { SET my_text.VISIBLE, 1; }
```
`POS_X`/`POS_Y` are the position from the upper‑left in pixels. Changing them
in‑game lets text scroll:
```
ACTION roll_my_text {
    SET my_text.POS_Y, 400;           // set text below screen
    SET my_text.VISIBLE, 1;
    WHILE (my_text.POS_Y>0) {         // while text still on-screen
        WAIT 1;
        RULE my_text.POS_Y -= 1;      // roll upwards one pixel line
    }
    SET my_text.VISIBLE, 0;
}
IF_R roll_my_text;
```

## Panels

Games use displays (numbers, bars) for player state and variables. A `PANEL` is
defined like a text, but with display elements (`DIGITS`, `HBAR`, `VBAR`, …). A
debug panel:
```
FONT standard_font, <panfont.pcx>, 8, 10;
PANEL debug_panel {
    POS_X 0;
    POS_Y 12;
    FLAGS VISIBLE, REFRESH;
    DIGITS 18,2,3,standard_font,160,TIME_FAC;        // frames per second
    DIGITS 50,2,2,standard_font,1,ACTIVE_NEXUS;
    DIGITS 80,2,2,standard_font,1,ACTIVE_OBJTICKS;
    DIGITS 110,2,2,standard_font,1,ERROR;
    DIGITS 150,2,4,standard_font,10,PLAYER_SIZE;
    DIGITS 200,2,4,standard_font,10,PLAYER_ANGLE;
    DIGITS 240,2,4,standard_font,1,PLAYER_X;
    DIGITS 280,2,4,standard_font,1,PLAYER_Y;
}
STRING debug_labels, "  fps nex obj err   siz  ang   x    y";
TEXT debug_text {
    POS_X 0; POS_Y 2;
    FONT standard_font; STRING debug_labels;
    FLAGS VISIBLE;
}
```
Each `DIGITS` line shows one skill: X/Y position (relative to the panel's
upper‑left), digit count, font, a multiplication factor, and the skill. The
`REFRESH` flag rebuilds the panel constantly — needed to display it over the 3D
window (texts set `REFRESH` automatically).

The frame‑rate display (`TIME_FAC`) changes too quickly; smooth it:
```
SKILL fps {}
ACTION show_panels {
    WHILE (1) {                       // once CALLed, repeat forever
        WAIT 1;
        RULE fps = 0.8*fps + 0.2*TIME_FAC;
    }
}
// ... then display the "fps" skill instead of TIME_FAC ...
```

### A compass (horizontal bar)
```
BMAP kompass_map, <kompass.pcx>, 0,0,160,22;
PANEL kompass_pan {
    POS_X 140;
    POS_Y 376;
    FLAGS VISIBLE, REFRESH;
    HBAR  0,0,40,kompass_map,-19.05,PLAYER_ANGLE;
}
```
`HBAR` parameters: position, display width (pixels), the bitmap to shift, the
multiplication factor, and the skill. For a cyclical bar, the bitmap width must
exceed the maximum shift by the display width. Here the compass scale repeats
every 120 pixels; a full rotation (`PLAYER_ANGLE` = 2π ≈ 6.3) should shift it 120
pixels left, so `factor = -120 / 6.3 = -19.05`.

## Overlays — weapons & tools

An `OVERLAY` is like a graphic on a transparent foil laid over the screen — ideal
for cockpits, swords, reticules. Example: the laser gun ("debugger") from
SKAPHANDER PLUS, with recoil and a fire flash:
```
OVLY cannon1_0 <canon1.LBM>, 0,0,68,82;
OVLY cannon1_1 <canon1.LBM>, 68,0,68,80;
OVLY cannon1_2 <canon1.LBM>, 136,0,68,79;
OVLY cannon1_3 <canon1.LBM>, 204,0,68,80;
OVLY cannon1_5 <canon1.LBM>, 68,82,68,82;
OVERLAY cannon0 { POS_X 27; POS_Y 250; FLAGS ABSPOS; OVLYS cannon1_0; }
OVERLAY cannon1 { POS_X 25; POS_Y 252; FLAGS ABSPOS; OVLYS cannon1_1; }
OVERLAY cannon2 { POS_X 24; POS_Y 253; FLAGS ABSPOS; OVLYS cannon1_2; }
OVERLAY cannon3 { POS_X 25; POS_Y 252; FLAGS ABSPOS; OVLYS cannon1_3; }
OVERLAY cannon5 { POS_X 27; POS_Y 250; FLAGS ABSPOS; OVLYS cannon1_5; }
```
`OVLY` has the same syntax as `BMAP` but stores graphics specially: 2–3× the
memory, but drawn much faster. `OVERLAY` relates to `OVLY` as `TEXTURE` does to
`BMAP`. `POS_X`/`POS_Y` are relative to the **3D window** corner — unless `ABSPOS`
is set, when they're relative to the screen.
```
SOUND twaeng, <laser.wav>;
ACTION fire {
    WHILE ((KEY_CTRL>0) || (MOUSE_LEFT>0)) {
        PLAY_SOUND twaeng, 0.3, -0.7;     // mounted on the left
        SET cannon0.VISIBLE,0; SET cannon1.VISIBLE,1;
        RULE PLAYER_LIGHT += 0.5;          // flash of light
        WAIT 1;
        SET cannon1.VISIBLE,0; SET cannon2.VISIBLE,1;
        RULE PLAYER_LIGHT -= 0.5;
        WAIT 1;
        SET cannon2.VISIBLE,0; SET cannon3.VISIBLE,1; WAIT 1;
        SET cannon3.VISIBLE,0; SET cannon5.VISIBLE,1; WAIT 1;
        SET cannon5.VISIBLE,0; SET cannon0.VISIBLE,1; WAIT 1;
    }
}
ACTION init_fire {
    SET cannon0.VISIBLE,1;
    SET IF_CTRL, fire;
    SET IF_LEFT, fire;
}
```
Cyclically changing the overlay animates the gun; slightly different positions give
recoil. `PLAY_SOUND` with a balance of −0.7 places the gun on the left.

### Making the gun hit

Add hit flashes and rising smoke as short‑lived actors with two `ONESHOT`
animations each:
```
TEXTURE flash_tex {
    SCALE_XY 12,12; CYCLES 4;
    BMAPS flash_1,flash_2,flash_3,flash_4;
    DELAY 1,1,1,1; FLAGS ONESHOT;
    AMBIENT 1;                    // flash shines in dark regions too!
}
TEXTURE wolke_tex { SCALE_XY 12,12; CYCLES 5; /* smoke */ FLAGS ONESHOT; AMBIENT 0.5; }
ACTOR flash1 { TEXTURE flash_tex; FLAGS GROUND,INVISIBLE,PASSABLE; }
// ... flash2, flash3, flash4 likewise ...

ACTION init_flash {
    SET MY.INVISIBLE,0;          // activate flash
    SET MY.TEXTURE,flash_tex;
    SET MY.PLAY,1;
    RULE MY.DIST = HIT_DIST-0.5;
    RULE MY.HEIGHT = PLAYER_Z - 0.83*HIT_DIST*PLAYER_TILT + RANDOM(0.4) - 1.2;
    DROP MY;                     // place in front of the hit object
    SET MY.SPEED,0.3;
    SET MY.TARGET,FOLLOW;        // flash approaches player
    SET MY.EACH_TICK,NULL;
    SET MY.EACH_CYCLE,init_wolke;
}
ACTION init_wolke {
    SET MY.TEXTURE,wolke_tex;
    SET MY.PLAY,1;
    SET MY.EACH_TICK,wolke_auf;
    SET MY.EACH_CYCLE,flash_weg;
}
ACTION wolke_auf { RULE MY.HEIGHT += 0.4*TIME_CORR; }   // smoke rises
ACTION flash_weg { SET MY.INVISIBLE,1; }                // free for reuse
```
In the `fire` action, after `SHOOT` (with `SHOOT_RANGE 200`), if `HIT_DIST > 0`,
find an unused flash actor and `CALL init_flash`:
```
SHOOT;
IF (HIT_DIST>0) {
    SET MY,flash1;
    IF (MY.INVISIBLE) { GOTO set_flash; }
    SET MY,flash2;
    IF (MY.INVISIBLE) { GOTO set_flash; }
    SET MY,flash3;
    IF (MY.INVISIBLE) { GOTO set_flash; }
    SET MY,flash4;
    set_flash:
    CALL init_flash;
}
```
Several flash actors are needed since the gun fires continually — while one smoke
cloud rises, new hits light up beside it.

## Automapping

The automap is a special overlay displayed over the 3D window when `MAP_MODE` is
set. Extend `show_panels`:
```
IF_TAB NULL;        // disable the predefined map action
ACTION show_panels {
    RULE fps = 0.8*fps + 0.2*TIME_FAC;
    SET MAP_MODE, 0;
    IF (KEY_TAB) {                         // [TAB] pressed?
        SET MAP_MODE, 0.5;                 // automapping
        IF (KEY_MINUS || KEY_SZ)  { RULE MAP_SCALE = MAP_SCALE*0.95; }
        IF (KEY_PLUS  || KEY_APO) { RULE MAP_SCALE = MAP_SCALE*1.05; }
    }
}
```
`[+]`/`[-]` change the scale. `MAP_MODE` 1 shows the complete level; 0.5 only the
explored area.

## Changing levels

A game usually has several levels; the player must carry features/weapons forward
and resume play later. There are **three** ways to change levels.

### 1. Several levels in one WMP (beaming)

Place sub‑levels far apart in one WMP (outside each other's `CLIP_DIST`) with
border regions between. Beam the player using an invisible target thing:
```
THING transporter_1 {
    TEXTURE pseudo_tex;
    FLAGS   INVISIBLE;
}
ACTION beam_1 {
    SET  PLAYER_X, transporter_1.X;
    SET  PLAYER_Y, transporter_1.Y;
    SET  PLAYER_ANGLE, transporter_1.ANGLE;
    SET  HERE, zielregion_1;                  // region the transporter is in
    SET  PLAYER_Z, zielregion_1.FLOOR_HGT;
    RULE PLAYER_Z = PLAYER_Z + PLAYER_SIZE;
    SET  PLAYER_HGT, 0;
}
```
Remember to re‑assign `HERE` or collision detection breaks. **Pros:** instant
changes, no disk access. **Cons:** the WMP size limits levels (~5000 objects); all
textures load at start‑up.

### 2. A separate WMP per level (`MAP`)
```
ACTION beam_2 { MAP <level_2.wmp>; }
```
Loads walls, things and actors from the new WMP; skills/panels stay. The disk is
accessed for several seconds (the game "freezes"). The WDL is unchanged, so all
objects of both levels must be defined there.

### 3. A separate WDL per level (`LEVEL`, professional only)
```
ACTION beam_3 { LEVEL "level_3.wdl", "level_3.wrs"; }
```
Loads a new WDL/resource and restarts the game. File names are in quotes (so the
compiler doesn't include them). All objects, positions, skills and actions are
forgotten; it continues with the new `IF_START`. The exception: **GLOBAL** skills.

## Global skills

```
SKILL MOTION_BLUR { TYPE GLOBAL; VAL 0.5; }
SKILL SOUND_VOL   { TYPE GLOBAL; VAL 1; }
SKILL MUSIC_VOL   { TYPE GLOBAL; VAL 0.5; }
SAVEDIR "C:\\MYGAMES";
ACTION global_save { SAVE_INFO "test", 0; }
ACTION global_load { LOAD_INFO "test", 0; }
```
GLOBAL skills hold universal parameters (sound/music settings, player strength and
equipment) valid across all levels — they keep their values when changing levels,
even if redefined. **The old and new WDL must define the same GLOBAL skills, with
the same names, in the same order!**

`SAVE_INFO`/`LOAD_INFO` write/read only the GLOBAL skills (independent of game
scores) to `SAVEDIR` (file = name + up to 3‑digit number + `.SAV`). The directory
is created if missing; `LOAD_INFO` leaves skills unchanged if the file is absent.

To load only *some* global skills, copy the others to "shadow" skills and restore
them afterwards:
```
SKILL motion_blur_s { }                    // shadow skill
ACTION global_load_vols {
    SET motion_blur_s, MOTION_BLUR;        // scrap old value
    LOAD_INFO "test", 0;
    SET MOTION_BLUR, motion_blur_s;        // get back old value
}
```
> Tip: delete old skill files after editing the WDL — changing the **number** of
> global skills since the last save can cause crash‑like situations!

## Saving and loading game scores

`SAVE` writes the complete game state to disk: the `.WDL`/`.WMP` names; all normal
and GLOBAL skills; all synonyms; the `EACH_TICK`/`EACH_SEC`/`LAYERS`/`PANELS`/
`MESSAGES` lists; all key actions; the parameters of every `PANEL`/`TEXT`/`OVERLAY`;
all actor coordinates/parameters; the flags of all regions/walls/things; and the
parameters of every region/wall/thing whose `SAVE` flag is set. Scores are keyed
and compressed. `LOAD` reloads them; running `EACH_TICK` actions restart from the
beginning, and a level change happens automatically if needed (loading occurs at
the **end** of the action). As with global skills, loading an old score after
changing the object count aborts the game.

This example saves with `[F2]` and cycles through the last three saved scores with
`[F3]`, showing on‑screen messages. Because loading changes every skill, the
**number** of the last saved score is kept in a GLOBAL skill `slot`.
```
SKILL slot   { TYPE GLOBAL; VAL 1; }   // number of score (1..3)
SKILL number { VAL 0; }
SAVEDIR "C:\\MYGAMES";
STRING save_yesno,  "Save game (Y/N)?";
STRING load_yesno1, "Load last score (Y/N/F3)?";
STRING ok,          "OK";
STRING save_nix,    "ERROR - nothing saved!\nHard disk already full?";
STRING load_nix,    "Sorry - no score found! Remember:\nFirst save - then load!";
// ... screen_txt TEXT and label strings ...

ACTION show_message {                   // show a message for 5 seconds
    SET screen_txt.VISIBLE,1;
    WAITT 80;
    SET screen_txt.VISIBLE,0;
}
ACTION wait_yesno {                     // freeze and wait for Y/N
    SET screen_txt.VISIBLE,1;
    SET MOVE_MODE,0;                    // freeze player
    SET IF_N, clear_yesno;
    SET IF_ESC, clear_yesno;
}
ACTION clear_yesno {                    // end the query
    SET screen_txt.VISIBLE,0;
    SET MOVE_MODE,1;                    // de-frost player
    SET IF_J,NULL; SET IF_Y,NULL; SET IF_Z,NULL; SET IF_N,NULL; SET IF_ESC,NULL;
    SET IF_F3, qload_game1;
}
ACTION qsave_game {                     // save, with query
    SET screen_txt.STRING, save_yesno;
    SET IF_J, save_game; SET IF_Y, save_game; SET IF_Z, save_game;
    CALL wait_yesno;
}
ACTION load_slot { LOAD_INFO "INFO", 0; }
ACTION save_game {
    CALL clear_yesno;
    CALL load_slot;                     // load the slot skill
    RULE slot += 1;                     // and increment it
    IF (slot>3) { RULE slot -= 3; }
    SAVE_INFO "INFO", 0;                // save new slot skill
    SET RESULT, 0;                      // to detect errors
    SAVE "GAME", slot;                  // save the game
    SET screen_txt.STRING, ok;
    IF (RESULT<0) { SET screen_txt.STRING, save_nix; }
    CALL show_message;
}
ACTION qload_game1 {
    SET number,0;                       // last score
    SET screen_txt.STRING, load_yesno1;
    SET IF_Y,load_game; SET IF_Z,load_game; SET IF_J,load_game;
    SET IF_F3, qload_game2;
    CALL wait_yesno;
}
// qload_game2 / qload_game3 are analogous (number = 1 / 2), cycling IF_F3
ACTION load_game {
    CALL clear_yesno;
    SET  screen_txt.STRING, wait_txt;
    SET  screen_txt.VISIBLE,1;
    WAIT 1;                             // show the message before loading
    CALL load_slot;
    RULE slot -= number;
    IF (slot<1) { RULE slot += 3; }
    LOAD "GAME", slot;
    SET  screen_txt.STRING, load_nix;   // only reached if loading failed
    CALL show_message;
}
IF_F2 qsave_game;
IF_F3 qload_game1;
```
The "Yes" action is bound to `[Y]`, `[J]` and `[Z]` for European keyboards. The
`WAIT 1` in `load_game` lets the "Please wait…" message appear before `LOAD`
freezes the screen.

> A closing riddle: this time we don't need to check `RESULT` after loading to
> know whether something went wrong — why? (Because if `LOAD` succeeds, the
> game state is replaced and execution never returns to the line after it!)
