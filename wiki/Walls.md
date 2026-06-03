# Walls

Vertical walls run along the connecting lines between two vertices and separate
the regions of the map. Their properties are defined in the wall definition:

```
WALL Keyword { ... }
```

The wall keyword defined this way can later be assigned to a connecting line in
WED. A wall definition may contain — in the given sequence — the following
keyword assignments.

> *Notation:* `<` marks a value changeable during gameplay; `#` marks a value
> set automatically each frame and evaluable in actions.

## Appearance

| Keyword | Meaning |
|---------|---------|
| `<TEXTURE Texture;` | Texture for the wall, up to 4 sides. On two‑sided textures the bitmap changes depending on which vertex is closer to the viewer. Four‑sided textures always differentiate the right and left side. |
| `<ATTACH Texture;` | Superimposed texture for the wall (one side). Its `CYCLES` must match the wall texture's or equal 1. |
| `<OFFSET_X Number;` `<OFFSET_Y Number;` | Shifts the wall texture `OFFSET_X` pixels left and/or `OFFSET_Y` pixels down. `OFFSET_X` must not be negative. If used, the texture cannot be shifted with WED's ALIGN function. For sky textures, `OFFSET_Y` is the horizon distance from the bitmap's lower border. |
| `<CYCLE n;` | The current animation phase of the texture. |
| `<POSITION Number;` | Meaning depends on the flags (see below). |
| `MAP_COLOR n;` | Mode (0…1, default 1) for drawing the wall on the automap. 0 = no map; 1 = antialiased default color (`COLOR_WALLS`, `COLOR_BORDER`). |
| `<DIST Number;` | Border distance of the nearest wall vertex to the player (default 0). Crossing it can trigger `IF_NEAR`/`IF_FAR`. At 0, `IF_NEAR` triggers simply by touching/passing the wall. If `DIST > 0` but less than half the wall length, `IF_NEAR` is not triggered near the center — only the distance to the nearest vertex matters! |
| `<SKILL1 … <SKILL8 Number;` | Eight "universal parameters" with no initial effect, changeable/evaluable by actions. |

## Automatic parameters (evaluable in actions)

| Keyword | Meaning |
|---------|---------|
| `#DISTANCE` | Approximate (±20%) distance of the player to the nearest wall vertex; only valid within the player's `CLIP_DIST`. |
| `#LENGTH` | Length of the wall in steps. |
| `#SIZE_X` | Length of the wall in pixels, depending on texture scaling. |
| `#LEFT` / `#RIGHT` | The regions on the left and right side of the wall. |
| `<X1, <Y1, <Z1` | Position of vertex 1. Changeable by an action to shift/rotate walls or regions (walls may not cross). If the wall length changes, a `SHAKE` command must be executed. |
| `<X2, <Y2, <Z2` | Position of vertex 2. |

## Event actions

| Keyword | Triggered when… |
|---------|-----------------|
| `<IF_NEAR Action;` | The player crosses the border distance (`DIST`) of the nearest vertex. Without a `DIST`, triggered by any touch/pass. |
| `<IF_FAR Action;` | The player moves beyond the border distance. |
| `<IF_HIT Action;` | The player hits the wall with `SHOOT`, or the wall is hit by an exploding object. |
| `<EACH_CYCLE Action;` | After every animation cycle of the wall texture, or after the end of a `ONESHOT` animation. |
| `<EACH_TICK Action;` | After every frame rendering cycle (≈ every 1/16 second). |

## Flags

```
FLAGS Flag1, Flag2 ...;
```
In actions, flags take the value 1 (set) or 0 (not set).

| Flag | Effect |
|------|--------|
| `<INVISIBLE` | Wall is invisible but impenetrable. The same region must be on both sides. |
| `<PASSABLE` | Wall is permeable to the player and actors. |
| `<IMMATERIAL` | Wall is no longer influenced by `SHOOT` instructions or mouse clicks. |
| `<IMPASSABLE` | Wall becomes impenetrable over its entire length and width, even if invisible. If `PASSABLE` is also set, it stays passable for the player but not for actors. |
| `#VISIBLE` | Set automatically while the wall is seen by the player. Evaluable in actions. |
| `<SEEN` | Set automatically once the player has seen the wall, then stays set. Used by the automap. |
| `<BERKELEY` | The wall exists only while viewed or while the player is within its border distance. Saves rendering time in levels with many animated wall textures. |
| `<TRANSPARENT` | Color 0 of the wall texture is transparent (e.g. fences, lattices). The same region must be on both sides. |
| `<PLAY` | Animates the wall texture for one cycle, then stops on the last phase. The texture must have `ONESHOT` set. `PLAY` resets to 0 at the end, triggering an `EACH_CYCLE` action if present. |
| `CURTAIN` | The wall reaches from floor to ceiling, independent of region heights. |
| `<PORTCULLIS` | "Fastens" the wall texture to the upper or lower edge — useful for vertically moving walls (elevators, roll‑down doors). The texture adjusts to the lower border (`POSITION=0`) or upper border (`POSITION=1`) so it moves with the wall. |
| `FENCE` | "Cuts off" transparent walls at the upper edge of their texture. `POSITION` gives the depth (steps) of the lower wall edge within the floor; a negative `POSITION` lets it float above ground. Changing `POSITION` raises/lowers the wall like a lattice. `PORTCULLIS` is set automatically. |
| `<SENSITIVE` | The wall triggers its `IF_NEAR` action already when seen by the player. |
| `<FRAGILE` | The wall's `IF_HIT` action may be triggered by an `EXPLODE` instruction. |
| `FAR` | The wall is visible outside `CLIP_DIST` (see the `CLIPPING` skill) — useful for distant SKY walls, allowing a lower `CLIP_DIST`. `FAR` walls may not border BELOW regions. |
| `SAVE` | All properties of the wall are saved by the `SAVE` instruction. |
| `<FLAG1 … <FLAG8` | Eight "universal flags" with no initial effect, changeable/evaluable by actions. |

> Because of mathematical inaccuracy in texture representation, walls should have
> a maximum length of **200 steps** and height of **1000 steps**. Longer walls
> must be split into sections.
