# Regions

Regions are closed areas of the map, bordered by walls, that may be stacked
vertically at will. Their properties are defined in the region definition:

```
REGION Keyword { ... }
```

The region defined this way can be assigned to the right or left side of walls in
WED. A region definition may contain — in the sequence given — the following
keyword assignments.

> *Notation:* `<` marks a value changeable during gameplay; `#` marks a value
> set automatically.

## Floor, ceiling and textures

| Keyword | Meaning |
|---------|---------|
| `<FLOOR_TEX Texture;` | Texture for the floor. The texture sound plays while moving, accompanying the `WALK` or `WAVE` cycles depending on whether the texture's `SLOOP` flag is set. Volume/pitch via predefined skills (e.g. `PSOUND_VOL`). |
| `<CEIL_TEX Texture;` | Texture for the ceiling. The sound plays when entering the region; with `SLOOP` set it continues regardless of player movement. |
| `<FLOOR_HGT Number;` `<CEIL_HGT Number;` | Floor/ceiling height in steps; slanted regions add the vertices' Z heights. The floor–ceiling difference should not exceed 1000 steps. |
| `<FLOOR_OFFS_X` `<FLOOR_OFFS_Y` `<CEIL_OFFS_X` `<CEIL_OFFS_Y Number;` | Shifts the floor/ceiling textures along X/Y in pixels, relative to the playfield coordinates. |
| `<FLOOR_ANGLE` `<CEIL_ANGLE Number;` | Texture angle of the floor/ceiling textures (0…2π, default 0). Corresponds to one revolution of the texture around the `GENIUS` center. Revolved textures cost more computing time and are only possible with non‑slanted regions. |

## Stacking and lighting

| Keyword | Meaning |
|---------|---------|
| `BELOW Region;` | Assigns a predefined lower region — for balconies, bridges, multi‑storied buildings, hovering objects or underwater regions. The lower region may itself have a `BELOW` region, and so on. All regions connected by `BELOW` are shown stacked. |
| `#TOP Region;` | The uppermost region if this is part of a `BELOW` chain; otherwise the region itself. Set automatically at game start. |
| `<GENIUS Object;` | Assigns a thing or actor as a light source for the region's walls (alternative to `LIGHT_ANGLE`, cf. `ALBEDO`) and as the rotational center for `ROTATE` commands. The object may be `INVISIBLE` but must have a texture `AMBIENT > 0` to serve as a light source. It may be outside the region and assigned to several regions at once. |
| `<AMBIENT Number;` | Basic brightness of the region (0…1, 16 degrees, default 0). Combined with texture/region ambient, `PLAYER_LIGHT` and player distance, it forms the texture's brightness. |
| `CLIP_DIST Number;` | Walls farther than this (steps) from the player are not rendered. Without it, the region uses the default `CLIP_DIST`. Regions never visible inside (e.g. floor and ceiling touch) should use `CLIP_DIST 0`. |
| `<SKILL1 … <SKILL8 Number;` | Eight "universal parameters" with no initial effect, changeable/evaluable by actions. |

## Event actions

| Keyword | Triggered when… |
|---------|-----------------|
| `<IF_ENTER Action;` | The player enters the region (must remain at least one frame — important for small regions and fast players). |
| `<IF_LEAVE Action;` | The player leaves the region. |
| `<IF_DIVE Action;` | The player's eye level (`PLAYER_Z`) reaches or falls below floor level. |
| `<IF_ARISE Action;` | The player's eye level reaches or rises above ceiling level. |
| `<EACH_CYCLE Action;` | At the end of every floor/ceiling texture animation cycle, or end of a `ONESHOT` animation. |
| `<EACH_TICK Action;` | After every frame cycle. |

## Flags

```
FLAGS Flag1, Flag2 ...;
```

| Flag | Effect |
|------|--------|
| `FLOOR_ASCEND` `CEIL_ASCEND` `FLOOR_DESCEND` `CEIL_DESCEND` | Lift the floor/ceiling corner points upward (ASCEND) or downward (DESCEND) by the vertices' Z values, tilting the surface (see notes below). |
| `#VISIBLE` | Set automatically while a floor, ceiling or wall of the region is visible. |
| `<SEEN` | Set automatically once the region is seen by the player. |
| `SAVE` | All region properties are saved on game saving. |
| `SAVE_ALL` | All walls and things within the region are saved by `SAVE` instructions — useful if the region is moved during gameplay (e.g. by `ROTATE`). |
| `#HERE` | Set automatically while the player is within this region. |
| `<BASE` | Ensures the player's floor height (`PLAYER_HGT`) does not change on entering — useful for narrow holes/slits the player should not "fall" into. |
| `STICKY` | The player moves together with the region's floor under `SHIFT`/`ROTATE`. |
| `<PLAY` | Animates floor and ceiling textures for one cycle, stopping on the last phase. The texture must have `ONESHOT` set. `PLAY` resets to 0, triggering `EACH_CYCLE` if present. |
| `<FLAG1 … <FLAG8` | Eight "universal flags" with no initial effect, changeable/evaluable by actions. |

## Restrictions for BELOW regions

- Things and actors **without** the `GROUND` flag are placed on the uppermost
  region at game start. Things and actors **with** the `GROUND` flag are placed on
  the lowest region whose ceiling is above the object's feet.
- Walls of BELOW regions may not have the `FAR`, `PORTCULLIS` or `FENCE` flag set.

## Underwater regions

If the `CEIL_HGT` of a BELOW region exactly equals the `FLOOR_HGT` of the region
above, the BELOW region can serve as an "underwater" region. The predefined skill
`PLAYER_DEPTH` then contains the difference between floor heights (the water
"depth"); it is 0 elsewhere. `PLAYER_DEPTH` may be used to reduce `PLAYER_SIZE`
while wading. If the upper region has an `IF_DIVE` and the lower an `IF_ARISE`
action, floor and ceiling become permeable to the player.

```
REGION under_water {        // to be defined BEFORE the water region
    FLOOR_HGT -5;
    CEIL_HGT  1;
    FLOOR_TEX mud_tex;
    CEIL_TEX  underwater_tex;
    IF_ARISE  water_arise;
}
REGION water {
    FLOOR_HGT 1;            // same as under_water's CEIL_HGT!
    FLOOR_TEX water_tex;
    CEIL_HGT  20;
    CEIL_TEX  sky_tex;
    BELOW     under_water;
    IF_DIVE   water_dive;
}
ACTION water_dive  { FADE_PAL blue_pal, 0.5; }
ACTION water_arise { FADE_PAL blue_pal, 0;   }
```

## Restrictions for slanted regions

(`FLOOR_ASCEND`, `CEIL_ASCEND`, `FLOOR_DESCEND`, `CEIL_DESCEND`)

- Textures are projected vertically onto slanted floors/ceilings so they align
  seamlessly despite the slope. Keep the angle moderate to avoid distortion, and
  use low‑contrast textures. Slanted areas cost computing time — use sparingly.
- With large floor/ceiling bitmaps (256×256), the texture `SCALE_X/Y` factor
  should not exceed 32, or the texture may "swim" in the distance. Smaller bitmaps
  may use a higher factor.
- When assigning Z heights to vertices, the floor and ceiling textures must not be
  "bent".
- Several identical regions with different inclines may not border the same BELOW
  region — define a separate region name for each.
- Walls separating slopes should be set `INVISIBLE` and `PASSABLE` manually.
- Avoid a topography giving a view exactly perpendicular (90°) onto a slope side,
  or distortions may appear.
- A slope's floor must not penetrate the ceiling (keep 0.01 steps clearance).
- Choose the `AMBIENT` of a slanted region so the floor/ceiling luminosity (from
  texture `AMBIENT` and `PLAYER_LIGHT`) never exceeds 1 nor goes negative.
