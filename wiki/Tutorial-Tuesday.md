# Tutorial — Tuesday: Moving the Player

As an example of writing a WDL action, we'll create one of the most difficult
actions of all: the movement of the player. You don't *have* to — you can use the
prefabricated move actions in `MOVE.WDL` — but writing your own teaches you the
problems of WDL arithmetic. It gets a bit mathematical, but the four basic
arithmetic operations are enough.

## The first move action

Using our minimal level, we throw out the prefabricated `set_walking` from
`IF_START` and write our own. Since movement happens continually, we use one of
the 16 global `EACH_TICK` actions (executed after each frame cycle):
```
ACTION my_move {
    RULE PLAYER_VX = FORCE_AHEAD;
    RULE PLAYER_VY = FORCE_STRAFE;
    RULE PLAYER_VROT = 0.1*FORCE_ROT;
}
ACTION my_start {
    SET EACH_TICK.16, my_move;
}
IF_START my_start;
```
The predefined skills `FORCE_AHEAD`, `FORCE_STRAFE` and `FORCE_ROT` hold input
values for forward/backward, sideways and rotational input (mouse, joystick,
cursor keys). E.g. `[8]` sets `FORCE_AHEAD` to 0.7, `[9]` to −0.7. `PLAYER_VX`,
`PLAYER_VY` are the player's speed along X/Y; `PLAYER_VROT` the rotation speed
(multiplied by 0.1 so the player doesn't get dizzy).

## Following the line of sight

Movement should follow the player's line of sight, which combines X and Y motion
relative to `PLAYER_ANGLE`. To move distance V at angle α, shift by `V·cos α` in X
and `V·sin α` in Y. `PLAYER_SIN` and `PLAYER_COS` give the sine and cosine
automatically:
```
ACTION my_move {
    RULE PLAYER_VX = FORCE_AHEAD * PLAYER_COS;
    RULE PLAYER_VY = FORCE_AHEAD * PLAYER_SIN;
    RULE PLAYER_VROT = 0.1*FORCE_ROT;
}
```
Including sideways movement, the first two RULEs become a coordinate
transformation from the player's (rotated) coordinate system into the "resting"
X/Y system:
```
RULE PLAYER_VX = FORCE_AHEAD * PLAYER_COS - FORCE_STRAFE * PLAYER_SIN;
RULE PLAYER_VY = FORCE_AHEAD * PLAYER_SIN + FORCE_STRAFE * PLAYER_COS;
```
Movement still appears jerky, especially with cursor keys. Next: smooth motion.

## Acceleration, inertia, friction

Distance, speed and time relate as `s = v·t` (steps, steps/tick, ticks). To change
speed, a **force** acts on the player, resisted by **inertia** (mass): `a = K/m`.
Three forces matter: the **propelling force** (from input), **drift** (a steady
pull, or gravity, varying per region), and **friction** which counters velocity and
grows with speed: `R = −f·m·v`. All three sum to change velocity:
`a = (K + D)/m − f·v`.

Translate this into an action with new skills:
```
SKILL force_x  {}          // force in direction of X
SKILL force_y  {}          // force in direction of Y
SKILL drift_x  { VAL 0; }  // drift in direction of X
SKILL drift_y  { VAL 0; }  // drift in direction of Y
SKILL strength { VAL 0.1; }// force coefficient
SKILL fric     { VAL 0.2; }// friction coefficient
SKILL mass     { VAL 1; }  // inertial mass of player

ACTION my_move {
    RULE force_x = strength*(FORCE_AHEAD*PLAYER_COS - FORCE_STRAFE*PLAYER_SIN);
    RULE force_y = strength*(FORCE_AHEAD*PLAYER_SIN + FORCE_STRAFE*PLAYER_COS);
    RULE PLAYER_VX = PLAYER_VX + (force_x+drift_x)/mass - fric * PLAYER_VX;
    RULE PLAYER_VY = PLAYER_VY + (force_y+drift_y)/mass - fric * PLAYER_VY;
    RULE PLAYER_VROT = 0.1*FORCE_ROT;
}
```

### The accuracy trap
The player won't stop! Skills and RULEs compute with only **3 decimal places** —
values below 0.001 are treated as 0. When `PLAYER_VX` drops below 0.005,
`fric * PLAYER_VX` (× 0.2) rounds to 0, so the player creeps forever at 0.005
steps/tick. Fix it with a mathematically identical but working form:
```
RULE PLAYER_VX = (1-fric) * PLAYER_VX + (force_x+drift_x)/mass;
RULE PLAYER_VY = (1-fric) * PLAYER_VY + (force_y+drift_y)/mass;
```
Now drift, strength, mass and friction can be tuned per region (e.g. `fric` 0.05
on ice, 0.5 in swamp), and movement is fluent.

### Frame‑rate compensation
Adding acceleration each frame means fast PCs accelerate more often. The
predefined skill `TIME_CORR` compensates: it's 1 at 16 fps, 2 at 8 fps, 0.5 at 32
fps. But naive use (`1 - TIME_CORR * fric` approaching 1) makes a stationary
player fidget. The whole equation — with time compensation, creep suppression,
oscillation suppression and special cases — is built into the **`ACCEL`**
instruction, which uses the predefined `FRICTION` and `INERTIA` skills (set them
before each `ACCEL`):
```
ACTION my_move {
    SET   INERTIA, mass;
    SET   FRICTION, fric;
    RULE  force_x = strength * (FORCE_AHEAD*PLAYER_COS - FORCE_STRAFE*PLAYER_SIN) + drift_x;
    ACCEL PLAYER_VX, force_x;
    RULE  force_y = strength * (FORCE_AHEAD*PLAYER_SIN + FORCE_STRAFE*PLAYER_COS) + drift_y;
    ACCEL PLAYER_VY, force_y;
    SET   FRICTION, 0.5;
    RULE  FORCE_ROT = 0.1 * FORCE_ROT;
    ACCEL PLAYER_VROT, FORCE_ROT;
}
```

## Vertical movement

Horizontal movement stays the same (except friction); vertical movement differs by
character. We implement **driving** (keep a constant distance to the surface) and
**walking** (rhythmic up/down, jumping, ducking).

The skills `PLAYER_Z`, `PLAYER_SIZE`, `FLOOR_HGT` and `PLAYER_HGT` are
responsible. `PLAYER_HGT` is the distance of the player's feet from the floor (can
be negative). The simplest height compensation:
```
RULE PLAYER_Z = FLOOR_HGT + PLAYER_SIZE;
```
This passes stairs, but jerkily. For smooth motion, model the vertical forces:
while airborne (`PLAYER_HGT > 0`) only gravity (a downward drift) and air friction
act; on touching/sinking into the ground a **resilient force** (knee joints /
suspension) pushes up, stronger the deeper the player sinks, and friction
increases:
```
SKILL force_z  {}            // force in direction of Z
SKILL fric_air { VAL 0.02; } // air friction coefficient
SKILL fric_gnd { VAL 0.97; } // ground friction coefficient
SKILL gravity  { VAL -0.2; } // gravitational force
SKILL knee_fac { VAL 0.3; }  // elasticity coefficient

ACTION my_move {
    // ... all the previous instructions ...
    SET FRICTION, fric_air;
    SET force_z, gravity;
    IF (PLAYER_HGT <= 0) {            // touching the ground?
        SET  FRICTION, fric_gnd;
        RULE force_z = gravity - PLAYER_HGT*knee_fac;
    }
    ACCEL PLAYER_VZ, force_z;
}
```
(Remove `SET FRICTION, fric_gnd;` and cross a step to see the rubber‑ball /
broken‑shock‑absorber effect — same as too large a `knee_fac`.)

## Walking, ducking and jumping

For walking, vary `PLAYER_SIZE` via the predefined `WALK` skill:
```
RULE PLAYER_SIZE = my_size + 0.3 * WALK;
```
```
SKILL my_size     { VAL 4; }   // player is normally 4 steps tall
SKILL WALK_PERIOD { VAL 5; }
SKILL WALK_TIME   { VAL 10; }
```
**Ducking** ([End] sets `FORCE_UP` to −0.7) — fluent version with a fading skill:
```
SKILL duck_val { VAL 0; }
// ...
RULE duck_val = 0.8*duck_val;          // reduce skill to "fade out"
IF (FORCE_UP<0) { RULE duck_val = duck_val + 0.5*FORCE_UP; }
RULE PLAYER_SIZE = PLAYER_SIZE + duck_val;
```
This repeats every tick; multiplying by 0.8 makes ducking/rising fade gently.
(The duck minimum solves to `(1−0.8)·duck = 0.5·0.7 → 1.75`.)

**Jumping** has three phases via `jump_phase`: duck, upward acceleration, free
fall. Insert before the `PLAYER_SIZE` reduction:
```
SKILL jump_phase { VAL 0; }
// ...
IF (jump_phase>0) { GOTO jump_1; }     // jump already going on?
IF (FORCE_UP<0.1) { GOTO no_jump; }    // NO pressing of [Home]?
SET jump_phase,1;                      // jump starts
jump_1:
IF (jump_phase>1) { GOTO jump_2; }
RULE duck_val = duck_val - 0.5;        // ducking
IF (duck_val<-0.7) { SET jump_phase,2; } // ducked deep enough? then jump up
GOTO no_jump;
jump_2:
IF (jump_phase>2) { GOTO jump_3; }
SET  duck_val,0;
RULE PLAYER_Z = FOOT_HGT + my_size;    // rise again
SET  PLAYER_VZ,0.5;                    // leaping off speed
SET  jump_phase,3;                     // now: flying freely
GOTO no_jump;
jump_3:
IF ((PLAYER_VZ<=0) && (PLAYER_HGT<=0)) { SET jump_phase,0; } // landed? jump finished
no_jump:
// ... now the RULE duck_val ... follows
```

## Looking up and down

`PLAYER_TILT` tilts the line of sight, returning to center when released:
```
RULE PLAYER_TILT = 0.8 * PLAYER_TILT + 0.3 * FORCE_TILT;
```
For a dramatic falling effect (tilt forward when falling off a cliff, but not while
jumping):
```
IF (jump_phase<=0) {
    RULE PLAYER_TILT = PLAYER_TILT + 0.03 * (PLAYER_HGT + 0.3);
}
```

## Per‑region parameters

To change `fric`/`gravity` by region, the cleanest way is `IF_ENTER`/`IF_LEAVE`
actions on the regions that deviate from the standard:
```
ACTION set_swamp_fric  { SET fric, 0.5; }
ACTION set_normal_fric { SET fric, 0.2; }
REGION swamp {
    // ... remaining region parameters ...
    IF_ENTER set_swamp_fric;
    IF_LEAVE set_normal_fric;
}
```
(Alternatively, read a region's universal skills via `HERE.SKILL5`, but those then
get "used up".)

## Special effects

**Zoom glasses** ([Ins]/[Del] change the focal arc):
```
IF (KEY_INS!=0) { RULE PLAYER_ARC = PLAYER_ARC - 0.1; }
IF (KEY_DEL!=0) { RULE PLAYER_ARC = PLAYER_ARC + 0.1; }
```
**Earthquake** ([E] triggers it) — random jitter scaled by a `richter` skill:
```
SKILL richter { VAL 0; }   // Richter scale
ACTION my_move {
    // ... all the rest ...
    RULE PLAYER_X = PLAYER_X + richter*(RANDOM(1) - 0.5);
    RULE PLAYER_Y = PLAYER_Y + richter*(RANDOM(1) - 0.5);
    RULE PLAYER_Z = PLAYER_Z + 2*richter*(RANDOM(1) - 0.5);
}
ACTION quake {
    SET richter,0.1; WAITT 16;   // start slowly (16 ticks = 1 sec)
    SET richter,0.3; WAITT 32;
    SET richter,0.5; WAITT 48;   // climax
    SET richter,0.2; WAITT 80;   // recover
    SET richter,0.6; WAITT 32;   // 2nd climax
    SET richter,0.1; WAITT 16;   // fade out
    SET richter,0;               // calm again
}
IF_E quake;
```

## Seeing through an actor's eyes

Instead of the player, we can bestow the view on an actor (the spherical "tour
guide" in `VRDEMO.WDL`), triggered by `IF_NEAR`:
```
ACTION guide_eye {
    SET  PLAYER_X, MY.X;
    SET  PLAYER_Y, MY.Y;
    RULE PLAYER_Z = MY.HEIGHT + THERE.FLOOR_HGT;  // relative → absolute height
    SET  HERE, THERE;                              // set player region
    SET  FRICTION, 0.5;
    RULE FORCE_ROT = 0.1 * FORCE_ROT;
    ACCEL PLAYER_VROT, FORCE_ROT;                  // we'll still like to rotate
}
ACTION change_body {
    SET EACH_TICK.16, NULL;          // remove old move action
    SET MY.EACH_TICK, guide_eye;     // start new one
    SET MY.PASSABLE, 1;              // or the player rebounds on himself
    SET MY.TEXTURE, blackpixel_tex;  // a near-invisible texture
}
ACTOR kugel {
    // ...
    DIST  0;
    FLAGS CAREFULLY;
    IF_NEAR change_body;
}
```
The actor's `BERKELEY` flag must no longer be set (he can't see himself). `DIST` 0
triggers `IF_NEAR` only on actual touch. `blackpixel_tex` is a tiny transparent
texture so the actor's body doesn't block the view.

You can now write further move actions for swimming, diving, climbing, skiing and
remote cameras. Enjoy!
