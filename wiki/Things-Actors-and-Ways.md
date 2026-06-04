# Things, Actors, and Ways

**Things** are zero/one‑dimensional items with only one vertex (as opposed to
walls). Their properties are defined in the thing definition:

```
THING Keyword { ... }
```

An **actor** differs from a thing only in its more complex action and reaction
behaviour:

```
ACTOR Keyword { ... }
```

Defining an actor this way lets you place him anywhere on the map in WED. You can
also assign him a **way** — a list of waypoints:

```
WAY Keyword;
```

The actor assigned a way "wanders" in a cycle from one waypoint to the next. No
collision detection is carried out unless the actor's `CAREFULLY` flag is set. An
actor walks the way in **reverse** when its `SPEED` is negative; the following
instructions make it turn around on the spot:
```
RULE MY.SPEED *= -1;
IF (MY.SPEED>0) { RULE MY.WAYPOINT += 1; }
ELSE           { RULE MY.WAYPOINT -= 1; }
```

A thing or actor definition contains the following assignments, in sequence.

> *Notation:* `<` marks a value changeable during gameplay; `#` marks a value set
> automatically.

## Appearance and overlay

| Keyword | Meaning |
|---------|---------|
| `<TEXTURE Texture;` | The texture (bitmap or model). On multi‑sided bitmap textures the side changes with the viewing angle. |
| `<ATTACH Texture;` | Attached texture. Its `CYCLES` and `SIDES` must match the object's texture's or equal 1. Things/actors may only have a single `ATTACH` texture; the texture's own `ATTACH` parameter is ignored. |
| `<CYCLE Number;` | Current animation cycle of the texture. |
| `<OVERLAY Overlay;` | Assigns an overlay keyword, accessible by actions (e.g. picking up the object with the mouse, or showing it in an inventory). |

```
SET MY.INVISIBLE, 1;        // disappear
SET MSPRITE, MY.OVERLAY;    // appear (as the mouse pointer)
```
These lines, triggered by a texture `IF_CLICK`, let you "pick up" an object with
the mouse pointer.

## Behaviour: `TARGET`

```
<TARGET Keyword;
```
Assigns a new destination or basic behaviour. Combining or event‑triggered
changes of `TARGET` can produce complex behaviour. Target keywords:

| Target | Behaviour |
|--------|-----------|
| `NULL` | The actor does not move (default). |
| `MOVE` | Moves straight in the direction of his `ANGLE`. `SPEED` sets horizontal speed; `VSPEED` is the tangent of the vertical angle (e.g. `VSPEED=1` climbs at 45°). With `CAREFULLY`, `IF_ARRIVED` triggers at each region border. |
| `BULLET` | Like `MOVE`, but on hitting an obstacle the `IF_HIT` action triggers (`CAREFULLY` must be set to detect collisions). |
| `STICK` | Tries to keep its current distance and angle to the player at max `SPEED`. `REL_ANGLE`/`REL_DIST` change its position relative to the player. With `CAREFULLY`, the skills `IMPACT_VX`, `IMPACT_VY`, `IMPACT_VROT` change on wall collisions. Can give the player a 3D "structure" (e.g. only entering narrow corridors sideways). |
| `FOLLOW` | Moves toward the player at `SPEED`; if `VSPEED > 0`, adjusts height to the player's (max vertical speed = `VSPEED`). If `VSPEED < 0`, moves up at `SPEED` at most and down at `VSPEED`. With `CAREFULLY`, `IF_ARRIVED` triggers at each region border. |
| `REPEL` | Like `FOLLOW`, but moves away from the player; height does not change. |
| `VERTEX` | Moves toward `TARGET_X`/`TARGET_Y`. On arrival, `IF_ARRIVED` triggers. |
| `HOLD` | (MDL actors only) Appears in front of the player regardless of position/tilt/angle — used for 3D weapons or tools. Direction set by `ANGLE`; position by `HEIGHT`, `TARGET_X`, `TARGET_Y`. Keep the parameter order; only one `HOLD` actor may be visible (set unused ones `INVISIBLE`). |
| `NODE1` / `NODE2` | The actor is remote‑controlled by the player plugged in via that node (professional only). Invisible until communication is established. If the number matches the current node (`-NODE`), the player inherits this actor's position and perspective at game start. |
| `Way` | Assigns a way; the actor immediately moves toward the first waypoint at `SPEED`. `WAYPOINT` is set to 1; `VSPEED` is ignored. `IF_ARRIVED` triggers at each waypoint. |
| `Wall`/`Thing`/`Actor` | A keyword/synonym for a wall, thing or other actor assigns its first vertex/position as the target, which the actor tries to reach at `SPEED`/`VSPEED`. With duplicates, the lowest WED index is chosen. On reaching the target, `IF_ARRIVED` triggers. |

### `HOLD` example
```
MODEL sword_mod, <sword.mdl>;
TEXTURE sword_tex {
    SCALE_X 32;
    SCALE_Y 32;
    MODEL   sword_mod;
}
ACTOR sword_arm {
    TEXTURE  sword_tex;
    TARGET   HOLD;
    ANGLE    3.14;
    TARGET_X 1;
    TARGET_Y 2;
    HEIGHT   -1;
}
```

## Movement and position parameters

| Keyword | Meaning |
|---------|---------|
| `<WAYPOINT Number;` | Assigns a waypoint on the way as destination; the actor moves directly toward it. During gameplay this holds the number of the next waypoint. |
| `<TARGET_X` `<TARGET_Y number;` | For actors with `WAY`, `FOLLOW` or an object target, the current target coordinates. With `TARGET VERTEX`, set the target here. |
| `<REL_ANGLE` `<REL_DIST number;` | Angle and distance to the player for a `STICK` actor; change to move him around the player. |
| `<HEIGHT Number;` | Height of the object's "feet" — absolute (with `GROUND`) or relative to the region floor. A slight sink (e.g. `HEIGHT = -0.25`) can look more realistic. |
| `<ANGLE Number;` | Object angle in radians (0…6.28), added to the WED angle; changeable by actions. |
| `<SPEED Number;` | Horizontal speed in steps per tick (default 0). A negative value makes a `WAY` actor travel its way in reverse. |
| `<VSPEED Number;` | Vertical speed; depending on `TARGET`, either a slope (tangent of vertical angle) or speed in steps per tick (default 0). |
| `ASPEED Number;` | Maximum rotation speed in radians per tick (default 0 = disabled). Multi‑sided/MODEL actors look more natural on ways with ~0.2. |
| `MAP_COLOR n;` | Color (0…255, default 1) representing the object on the map. 0 = not drawn; 1 = default color (skills `COLOR_ACTORS`, `COLOR_THINGS`). |
| `<DIST Number;` | Border distance from the object's center to the player (default 0). Crossing it can trigger `IF_NEAR`/`IF_FAR`. At 0, `IF_NEAR` triggers only by touch. If `DIST > 0` but less than half the object size plus the player width, the player would have to penetrate the object to trigger `IF_NEAR`! |
| `<SKILL1 … <SKILL8 Number;` | Eight "universal properties" with no initial effect, changeable/evaluable by actions. |

## Automatic parameters (evaluable in actions)

| Keyword | Meaning |
|---------|---------|
| `#DISTANCE` | Approximate (±20%) distance of the player to the object's center; only valid within `CLIP_DIST`. |
| `#SIZE_X` / `#SIZE_Y` | Horizontal/vertical size in steps (from pixel size and texture scale). |
| `<X, <Y` | Vertex position. Changing it "beams" the actor to a new position — the new region must be set manually, and if the height above floor changes, a `SHAKE` is needed. |
| `<REGION Region;` | The object's region; may need manual re‑assignment when moving. **The region of *things* may not be changed**, or collision detection breaks. |

## Event actions

Actions run only if the object is within `CLIP_DIST` or has the `LIBER` flag set.

| Keyword | Triggered when… |
|---------|-----------------|
| `<IF_NEAR Action;` | The player approaches the border distance (`DIST`). Without a `DIST`, on every contact. |
| `<IF_FAR Action;` | The player moves beyond the border distance. |
| `<IF_ARRIVED Action;` | An actor with `CAREFULLY` crosses a region border, or arrives at its target/next waypoint. |
| `<IF_HIT Action;` | The object is hit by `SHOOT`/`EXPLODE`, or collides with an obstacle (see `BULLET`, `FRAGILE`). |
| `<EACH_CYCLE Action;` | After every animation cycle, or at the end of a `ONESHOT` animation. |
| `<EACH_TICK Action;` | After every frame cycle (≈ every 1/16 second). |

## Flags

```
FLAGS Flag1, Flag2 ...;
```

| Flag | Effect |
|------|--------|
| `#THING` / `#ACTOR` | `THING` is set automatically for things and actors; `ACTOR` for actors. Used to determine the type of an object synonym. |
| `<INVISIBLE` | The object is invisible and passable, as if absent. Its actions don't run and events don't trigger. Things/actors defined in WDL but not placed are automatically `INVISIBLE`. |
| `<PASSABLE` | Passable to the player. |
| `<IMPASSABLE` | The thing or actor becomes impenetrable, even if invisible. (Since the Spring 1998 release this flag, formerly walls‑only, may also be set on things and actors.) |
| `#VISIBLE` | Set automatically while visible. |
| `<BERKELEY` | Inactive (`INVISIBLE`) while unseen and the player is outside its `DIST`. Stops actor movement out of sight and saves rendering time with many animated objects. |
| `<LIBER` | The object moves and runs actions even outside `CLIP_DIST` (costs computing time). Useful for behaviour independent of the player — patrolling guards, `EACH_TICK` time bombs, projectiles. |
| `FAR` | Invisible actors farther than their `DIST` normally do a `TARGET` movement only every 6th frame (see `SKIP_FRAMES`). Setting `FAR` makes such an actor move continuously regardless of `SKIP_FRAMES`. |
| `<GROUND` | `HEIGHT` refers to the Z coordinate rather than the region height. The object is vertically clipped at its region's floor/ceiling (slopes ignored) — useful for submerging/rising from water. Without `GROUND`, objects are always shown in full (though walls/region borders still clip them); they start in the uppermost region. Changing this flag by action automatically converts `HEIGHT`! |
| `CANDELABER` | (Only if `GROUND` is not set) The object is suited to the ceiling instead of the floor — useful for pendant objects like stalactites or candelabra. |
| `<SEEN` | Set automatically once seen by the player (used by the automap). |
| `<MOVED` | Set automatically if the actor moved in the latest image cycle; evaluable in actions and may need manual reset. |
| `<PLAY` | Animates the texture for one cycle, stopping on the last phase (texture must have `ONESHOT`). `PLAY` resets to 0, triggering `EACH_CYCLE` if present. |
| `<IMMATERIAL` | No longer affected by `SHOOT` instructions or mouse clicks. |
| `FLAT` | *(v3.8 manual)* The object does not always turn its front side to the player; instead it behaves like a transparent wall along its thing‑angle (`ANGLE`) — intended for flat or longish objects (spears, missiles). Like `IMMATERIAL`, it is no longer affected by `SHOOT` instructions or mouse clicks. |
| `<SENSITIVE` | `IF_HIT` triggers when the actor collides with an obstacle, and `IF_NEAR` triggers already when the actor is spotted by the player. |
| `<FRAGILE` | The object's `IF_HIT` action may be triggered by an `EXPLODE` instruction. |
| `<CAREFULLY` | The actor performs collision detection on his way, avoiding walls, things and other actors, and detects region changes (adjusting height if `GROUND` is not set). Costs computing time. |
| `MASTER` | With `MASTER` and `CAREFULLY`, the actor triggers `IF_LEAVE`, `IF_ENTER` and `IF_NEAR` events like the player. `IF_NEAR` with `DIST > 0` won't work — he must touch or pass the wall/object. |
| `SAVE` | All properties of the thing are saved at game saving. Automatically set for actors. |
| `<FLAG1 … <FLAG8` | Eight "universal flags" with no initial effect, changeable/evaluable by actions. |

> **Hint:** Since bitmap‑changing via `SIDES` on non‑moving objects can look
> unnatural, only moving and animated actors should be multi‑sided.
