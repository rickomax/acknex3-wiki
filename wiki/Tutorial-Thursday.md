# Tutorial — Thursday: Artificial Intelligence

How do you define the most intelligent behaviour possible for an opponent? This
chapter infuses creatures with an electronic personality using the theory of
**state machines**.

To populate the world with living creatures we use the `ACTOR` definition. Actors
move independently and are defined like things, with a few extra flags and
keywords: `TARGET`, `WAYPOINT`, `SPEED`, `VSPEED`. With `TARGET` an actor gets a
basic movement (follow a way, advance toward or flee a target) — but to be a
serious opponent, a monster must react intelligently: elude the player while
strong, hunt him when weak. That needs a kind of artificial intelligence.

## The theory of black boxes

We treat the machine as a **black box** defined solely by its observable behaviour.
If simple enough, it's a **state machine**: a limited number of distinct
behavioural patterns (**states**), always in exactly one of them. An external
stimulus (an **event**) causes a transition to another state.

For an actor we might define the states: **Waiting, Attacking, Escaping, Dying,
Dead.** Transitions are triggered by WDL events:

| Event | Realised by |
|-------|-------------|
| Actor views player | `EACH_TICK` action with periodic `SHOOT` |
| Player comes near actor (or vice versa) | `IF_NEAR` with `DIST > 0` |
| Player and actor touch | `IF_NEAR` with `DIST = 0` |
| Player moves beyond a distance | `IF_FAR` action |
| Actor near an object triggering `EXPLODE` | `IF_HIT` with `FRAGILE` set |
| Player hits actor with `SHOOT` | `IF_HIT` action |
| Actor texture animation cycle passed | `EACH_CYCLE` action |
| A skill reaches a certain value | `EACH_TICK` comparing the skill to its limit |
| A certain time has passed | `EACH_TICK` counting down a skill |
| Actor reached a certain waypoint | `IF_ARRIVED` comparing waypoint numbers |
| Actor reached a certain region | `IF_ARRIVED` comparing a region parameter |
| Actor collided with an obstacle | `IF_HIT` with `TARGET BULLET` |
| Random event | `EACH_TICK` comparing `RANDOM` to a value |

A WDL action performs each transition through a number of `SET` instructions.
**Skills** act as the machine's inner variables, influencing behaviour and
transitions.

## The table of states

Dividing behaviour into states and transitions keeps the WDL transparent. Our
example is the **Tron** virus from SKAPHANDER. At start, Tron is in **WAIT**
(lurking). If the player comes near, he **ATTACK**s. Being hit transitions him to
ATTACK, ESCAPE or DIE depending on the hit count (kept in `SKILL1`).

| State | Texture | Target | Player near | Hits ≤ 4 | Hits ≤ 5 | Hits > 5 | Anim. finished |
|-------|---------|--------|-------------|----------|----------|----------|----------------|
| WAIT | tron_tex | NULL | ATTACK | ATTACK | ESCAPE | DIE | – |
| ATTACK | tron_atak | FOLLOW | – | ATTACK | ESCAPE | DIE | – |
| ESCAPE | tron_hovo | REPEL | ATTACK | ATTACK | ATTACK | DIE | – |
| DIE | tron_explo | NULL | – | – | – | – | DEAD |
| DEAD | tron_rest | NULL | – | – | – | – | – |

Empty fields (–) leave the state unchanged. Define the table as completely as
possible — with complex machines, special cases (like the never‑used WAIT→DEAD
transition) aren't always obvious.

## Defining the actor

Define sounds, bitmaps and textures for each state. The Tron uses an 8‑sided
animated texture; `MIRROR` flags save bitmaps thanks to symmetry. The escape
texture is darkened with a negative `AMBIENT` to show damage. (Bitmap lists
abbreviated here.)
```
SOUND tron_snd,    <tron_vir.wav>;
SOUND tron_baller, <schuss1.wav>;
SOUND tron_aua,    <tron_au.wav>;
// ... many BMAP declarations for the 8 sides × 6 cycles, attack, explosion ...

TEXTURE tron_tex {        // WAIT — 8 sides, 6 cycles
    SCALE_X 16; SCALE_Y 16;
    SIDES   8;  CYCLES  6;
    BMAPS   /* 48 bitmaps, front → counter-clockwise, some reused */ ;
    DELAY   3,3,2,2,3,3;
    MIRROR  0,0,0,1,1,1,1,0;
    AMBIENT 0.1;
    SOUND   tron_snd; SCYCLE 1; SVOL 0.2; SDIST 40;
}
TEXTURE tron_hovo  { /* ESCAPE — like tron_tex, AMBIENT -0.2, RANDOM 0.3 */ }
TEXTURE tron_atak  {      // ATTACK — single-sided, shoots once per cycle
    SCALE_XY 16,16; CYCLES 7;
    BMAPS    tron_0vh1, tron_an2, tron_an3, tron_an4, tron_an5, tron_an6, tron_an3;
    DELAY    1,1,1,1,1,1,1;
    AMBIENT  0.3;
    SOUND    tron_baller; SCYCLE 4; SVOL 0.25; SDIST 200;
}
TEXTURE tron_explo { SCALE_XY 18,18; CYCLES 11; /* ... */ AMBIENT 1; FLAGS ONESHOT; }
TEXTURE tron_rest  { SCALE_XY 16,16; BMAPS tron_corpse; }

ACTOR simple_tron {
    TEXTURE   tron_tex;
    SKILL1    0;           // initial number of hits
    HEIGHT    2;           // hovering above ground
    EACH_TICK tron_warten; // initial state
}
```
Setting the initial state as an `EACH_TICK` activates WAIT immediately at game
start — but the action must reset its own `EACH_TICK` to `NULL` or it would repeat.

## The state actions

```
ACTION tron_warten {           // WAIT state
    SET MY.TEXTURE, tron_tex;
    SET MY.TARGET, NULL;
    SET MY.DIST, 60;           // effective range
    SET MY.IF_NEAR, tron_angriff;
    SET MY.IF_HIT, tron_anfluster;
    SET MY.EACH_TICK, NULL;
    SET MY.EACH_CYCLE, NULL;
}
ACTION tron_anfluster {        // transition to ATTACK, ESCAPE or DIE
    RULE MY.SKILL1 += 1;       // increase number of hits
    PLAY_SOUND tron_aua;       // wail
    IF (MY.SKILL1>5) { CALL tron_sterben; }
    IF (MY.SKILL1==5) { CALL tron_flucht; }
    CALL tron_angriff;
}
ACTION tron_angriff {          // ATTACK state
    SET MY.TEXTURE, tron_atak;
    SET MY.SPEED, 0.4;
    SET MY.VSPEED, 0.2;
    SET MY.CAREFULLY, 1;       // avoid obstacles
    SET MY.TARGET, FOLLOW;
    SET MY.IF_NEAR, NULL;
    SET MY.IF_HIT, tron_anfluster;
    SET MY.EACH_TICK, NULL;
    SET MY.EACH_CYCLE, tron_schuss;
}
ACTION tron_schuss {           // fire
    SET SHOOT_X, 0;
    SET SHOOT_Y, -0.5;         // Tron shoots with his legs
    SET SHOOT_RANGE, 200;
    SHOOT MY;
    IF (HIT_DIST>0) { RULE health -= 1; }   // hit? player takes damage
}
```
Since actions are triggered by the actor itself, `MY` is the actor. `VSPEED` with
`FOLLOW` lets Tron hover up toward a player on a ledge. The attack texture fires
once per cycle, so `EACH_CYCLE` (`tron_schuss`) checks for a hit without changing
state. `SHOOT` checks for an unobstructed horizontal line to the player within
`SHOOT_RANGE`; `SHOOT_Y` shifts the shot down (Tron shoots with his "legs") so he
can't hit through a wall.
```
ACTION tron_flucht {           // ESCAPE state
    SET MY.TEXTURE, tron_hovo;
    SET MY.SPEED, 0.4;
    SET MY.VSPEED, 0;
    SET MY.CAREFULLY, 1;
    SET MY.TARGET, REPEL;
    SET MY.IF_NEAR, NULL;
    SET MY.IF_HIT, tron_angster;
    SET MY.EACH_TICK, NULL;
    SET MY.EACH_CYCLE, NULL;
}
ACTION tron_angster {          // transition to ATTACK or DIE
    RULE MY.SKILL1 += 1;
    PLAY_SOUND tron_aua;
    IF (MY.SKILL1>5) { CALL tron_sterben; }
    ELSE { CALL tron_angriff; }
}
ACTION tron_sterben {          // DIE state
    SET MY.TEXTURE, tron_explo;
    SET MY.SPEED, 0.2;
    SET MY.VSPEED, 0;
    SET MY.CAREFULLY, 0;
    SET MY.PASSABLE, 1;
    SET MY.TARGET, FOLLOW;     // makes the explosion exhaust expand toward player
    SET MY.IF_NEAR, NULL;
    SET MY.IF_HIT, NULL;
    SET MY.EACH_TICK, NULL;
    SET MY.EACH_CYCLE, tron_tod;
}
ACTION tron_tod {              // DEAD state
    SET MY.TEXTURE, tron_rest;
    SET MY.PASSABLE, 1;
    SET MY.TARGET, NULL;
    SET MY.IF_NEAR, NULL; SET MY.IF_HIT, NULL;
    SET MY.EACH_TICK, NULL; SET MY.EACH_CYCLE, NULL;
}
```
In `tron_sterben`, the dead Tron still `FOLLOW`s the player — a trick to make the
explosion exhaust approach and expand realistically (with `CAREFULLY 0` so it no
longer avoids obstacles). `PASSABLE` is set so the exploding/dead Tron isn't an
obstacle.

## Advanced state machines

Cosmetic enlargements give an actor a more "personal" repertoire (a cry of anger
on spotting the player; rebounding or spinning when hit; dodging sideways instead
of a straight charge) — realised in `ENEMY.WDL`. Strategic improvements need
actors to **communicate** (via `EXPLODE` and skills) and consider the player's
position relative to the topography:

- A "guardian" that spots the player alarms a fellow actor within range (sensors,
  cameras, trip‑wires). The player should be aware of the alarm to build suspense.
- Several attackers try to surround and encircle the player.
- Actors lurk around corners, advancing just to where the player can't yet see them.
- "Trap" rooms where actors attack from all sides.
- Actors compare the player's strength to their own and call reinforcement if he's
  still strong, else attack directly.

Don't overdo it — the player should always sense danger ahead; actors that hide
forever and ambush aren't much fun.

## Special machines

Objects can react in special ways when hit or used. Two examples from
SKAPHANDER PLUS: the **barrel** and the **mine**.

### The jumping barrel

The barrel jumps when hit; on landing it plays an animated texture.
```
TEXTURE fass_tex {
    SCALE_XY 35,35; CYCLES 5;
    BMAPS fass1, fass2, fass3, fass4, fass5;
    FLAGS ONESHOT; DELAY 3,3,3,3,3;
}
THING fass {
    TEXTURE fass_tex;
    HEIGHT  0;
    FLAGS   FRAGILE;   // barrel reacts to EXPLODE as well
    IF_HIT  fass_hit;
}
SKILL hupf_abweich { VAL 0; }
ACTION fass_hit {
    IF (MY.RESULT<0.1) { END; }              // no real shot?
    PLAY_SOUND banngg, 0.2;
    RULE MY.SKILL1 = 1 + hupf_abweich;       // initial speed
    RULE hupf_abweich += 0.05;               // individual speed difference
    IF (hupf_abweich>0.25) { RULE hupf_abweich = -0.15; }
    SET MY.EACH_TICK, fass_hupf;
}
SKILL decken_distanz { VAL 0; }
ACTION fass_hupf {
    SET RENDER_MODE, 1;                      // make barrel movement visible
    RULE decken_distanz = THERE.CEIL_HGT - THERE.FLOOR_HGT - MY.SIZE_Y;
    RULE MY.HEIGHT += MY.SKILL1;             // move barrel vertically
    IF (MY.HEIGHT>=decken_distanz) { RULE MY.HEIGHT = decken_distanz; }  // touch ceiling
    RULE MY.SKILL1 += -0.15*TIME_CORR;       // gravitation
    IF (MY.HEIGHT>0) { END; }                // still airborne?
    RULE MY.HEIGHT = 0;
    PLAY_SOUND rrumms, 0.3;                  // impact on ground
    SET MY.EACH_TICK, NULL;
    SET MY.PLAY, 1;                          // start texture animation
}
```
A slightly different `hupf_abweich` per barrel makes rows of barrels look better
when set off together. Gravity reduces the vertical speed by a `TIME_CORR`‑scaled
amount each tick until the barrel falls back.

### The plant‑and‑run mine

The mine can be picked up, carried, thrown and exploded — a more complex machine.
The ticking texture `mine_tick_tex` animates between two identical bitmaps purely
for the rhythmic ticking sound.
```
ACTOR mine {
    TEXTURE mine_tex;
    FLAGS   FRAGILE;
    IF_NEAR nimm_mine;
}
ACTION nimm_mine {              // take mine from ground
    PLAY_SOUND beamer, 0.3;
    SET MY.INVISIBLE, 1;
    SET mine1_ovr.VISIBLE, 1;   // reappears as an overlay
    SET IF_CTRL, mine_werf;
    SET IF_LEFT, mine_werf;
}
ACTION mine_werf {              // throw mine
    SET IF_CTRL, NULL; SET IF_LEFT, NULL;
    SET mine1_ovr.VISIBLE, 0;
    PLAY_SOUND klick, 0.5;
    SET mine.HEIGHT, 1.5;
    SET mine.DIST, 0;
    DROP mine;                  // put at player's position
    SET mine.TEXTURE, mine_tex;
    SET mine.INVISIBLE, 0;
    SET mine.ANGLE, PLAYER_ANGLE;
    SET mine.SPEED, 2;
    SET mine.VSPEED, 0.4;
    SET mine.CAREFULLY, 1;
    SET mine.TARGET, MOVE;      // set in motion
    SET mine.EACH_TICK, wurf_flug;
    SET mine.EACH_CYCLE, NULL;
    SET mine.IF_NEAR, NULL;
}
ACTION wurf_flug {              // flying
    RULE MY.VSPEED += -0.15*TIME_CORR;   // gravitation
    IF (MY.HEIGHT>0) { END; }
    PLAY_SOUND klonk, 0.5;               // hits the ground
    SET MY.HEIGHT, 0;
    SET MY.VSPEED, 0;
    SET MY.EACH_TICK, wurf_rutsch;
}
ACTION wurf_rutsch {            // slide on ground
    RULE MY.SPEED += -0.3*TIME_CORR;
    IF (MY.SPEED>0) { END; }
    SET MY.TEXTURE, mine_tick_tex;
    SET MY.EACH_TICK, NULL;
    SET MY.EACH_CYCLE, mine_scharf;
    SET MY.TARGET, NULL;
    SET MY.IF_NEAR, nimm_mine;
}
```
A classic transition: when touched, the mine disappears from the world and
reappears as a screen **overlay**; pressing `[Ctrl]` or the left button throws it.
Throwing uses `DROP` (the invisible mine was still where it was picked up), then a
flight phase (parabola, with `CAREFULLY` so it rebounds) and a slide phase. Once
still, the mine is armed — ticking, waiting for a `FRAGILE` actor.
```
ACTION mine_scharf {            // armed
    SET SHOOT_FAC, 0;
    SET SHOOT_RANGE, 15;
    EXPLODE MY;                 // test explosion: enemy within 15 steps?
    IF (HIT_MINDIST>0) { CALL mine_explode; }
}
ACTION mine_explode {           // explode (really!)
    SET MY.PASSABLE, 1;
    SET MY.CAREFULLY, 0;        // important for THERE!!
    SET MY.SPEED, 0.3;
    SET MY.TARGET, FOLLOW;
    SET SHOOT_RANGE, 40;
    SET SHOOT_FAC, 1;
    EXPLODE MY;                 // now a "real" explosion
    SET MY.TEXTURE, min_bums_tex;
    RULE MY.HEIGHT -= 2;        // new texture is larger
    RULE THERE.AMBIENT += 0.2;  // illuminate region
    SET MY.EACH_CYCLE, mine_weg;
    SET MY.IF_HIT, NULL; SET MY.IF_NEAR, NULL; SET MY.EACH_TICK, NULL;
    IF (RESULT>0) { RULE health -= 200*RESULT; }   // was the player hit too?
}
ACTION mine_weg {               // disappear
    RULE THERE.AMBIENT -= 0.2;  // region dark again
    SET MY.INVISIBLE, 1;
}
```
Each cycle the armed mine runs a **test `EXPLODE`** with `SHOOT_FAC 0` (harming
no one) to check for a nearby victim — an example of abusing `EXPLODE` for
testing. Since `EXPLODE` is costly, do such cyclical tests at larger intervals
(`EACH_CYCLE`). A real explosion uses a larger `SHOOT_RANGE` (40), so a triggering
actor may doom nearby fellows; hit actors damage themselves via their `IF_HIT`.

> **Trap with `THERE`:** when lighting the region via `THERE.AMBIENT` and darkening
> it afterwards, make sure it's the *same* region. Set the actor's `CAREFULLY` flag
> to 0 first, so the `THERE` synonym doesn't change if the explosion exhaust
> crosses a region border.
