# Problems & Hints

## Updating from a previous version

Version 3.9 has two differences from previous versions that may, in rare cases
(depending on your WDL files), cause undesired behaviour.

1. **Division by zero in `RULE`.** Older versions tolerated it; V3.9+ does not. If
   you hit a "division by zero" abort, check the indicated action. By limiting
   dividend/divisor values, ensure a division result never exceeds ±2,000,000.

2. **A bug fix for `BULLET` actors.** Suppose an actor fires rockets at the
   player: you define rocket actors, place them at the shooter's position, and run
   them with `TARGET BULLET`. To stop a rocket immediately colliding with the
   shooter, place it outside his collision radius, or set him `PASSABLE` for one
   frame before firing. In all engines before V3.9, a bug caused 6 frames to be
   skipped right after clearing a `BULLET` actor's `INVISIBLE` flag (see
   `SKIP_FRAMES`), so the bullet appeared "in the air" rather than where placed.
   That bug was removed — but the fix means the bullet won't run at all if you
   forget the `PASSABLE`!

## Windows version problems

The Windows 95/98 runtime module needs **DirectX 5** installed, and some older
graphics cards' DirectX drivers won't support the 320×240 or 320×400 resolution.
If you develop a game in 320×400, include at least one other resolution (e.g.
640×480). Some older cards also won't support the stereoscopic (shutter glasses)
mode. If you see **white spots** in the picture that don't appear in the DOS
version, you probably forgot to set color `#255` to white.

## DOS version problems

The engine integrated in WED is the DOS one. Due to multitasking, it (and the DOS
runtime module) runs noticeably slower (up to 25%) under Windows 95 than under
plain DOS. There can be joystick port problems, seen as "spontaneous shaking" of
the player — start with the joystick disabled (`-NJ`).

Large levels with many bitmaps may exceed the virtual memory automatically
reserved for the DOS box, causing an **"Out of Memory"** abort at start‑up. If you
have disk space, increase the virtual memory via the DOS box setting (right‑click
the DOS icon → Properties → Memory → memory for DPMI).

Some sound cards don't play MIDI songs in a DOS screen if a song was played
earlier under Windows — this is an error of Windows 95A, not ACKNEX.

Many notebook video chipsets won't run in SVGA mode. There's also a known problem
with S3Trio cards produced mid‑1996 to mid‑1997 (a VESA BIOS bug). In these cases
use a lower resolution, or produce a Windows runtime module to test.

You can switch between WED and other Windows 95 applications with `[Alt]-[Return]`
if WED was started with `-VGA`.

## Restrictions

Software restrictions to observe during level design:

| Limit | Value |
|-------|-------|
| Total objects (walls, things, actors) per level | 10000 |
| Actors per level | 500 |
| Regions per level | 1000 |
| Walls and things per region | 200 |
| Steps per wall (length) | 200 |
| Height difference between 2 regions (steps) | 200 |

> On the **lite** and **commercial** versions, the total number of walls plus
> things/actors per level is limited to **1500**.

## Frame rate

The frame rate (responsible for "smoothness" of motion) can fluctuate by up to
200% depending on level design and player position. Besides processor performance
and the size/resolution of the 3D window, these factors influence it:

| Factor | Influence |
|--------|-----------|
| Number of objects (walls, things, actors) in the level | ~25% |
| Number of objects within `CLIP_DIST` | up to ~30% (keep `CLIP_DIST`s small!) |
| Simultaneously running actions and ways | up to ~50% |
| Number of visible walls and objects | up to ~200% |
| Ratio of floors/ceilings/things to walls in the picture | ~40% |
| Ratio of inclined to straight floor areas | up to ~100% |

The player's movement action also strongly affects the subjective perception of
speed. For a good frame rate on old 486 PCs, use no more than ~2000 walls and
objects per level, of which no more than ~50 are simultaneously visible. No more
than ~20 actors with a way and the `CAREFULLY` flag should move simultaneously.
Avoid letting actors run over tilted floors. A large level with widely scattered
rooms performs better than a small, narrow level with rooms close together.
