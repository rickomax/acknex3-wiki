# Introduction

Congratulations! You have purchased the conitec **3D GameStudio**. This toolkit
allows you to create 3‑D demos, role playing, action, adventure or racing games
without programming knowledge, and to publish them subsequently without having
to pay royalties.

In order to construct a game you write a script file (**WDL**) which contains the
"source code" for the game world — the specifications for textures, regions,
things, and actors. The level topography is contained in a second file (**WMP**)
with the coordinates (vertices) of all objects. All files contain plain ASCII
text, so you may edit them with any ASCII editor (such as WORDPAD or EDIT). The
WMP file, however, is normally created with the topography editor **WED**.

WED contains the ACKNEX 3‑D engine for running and testing the game. When your
game is finished, WED can create a runtime module for it — a separate `.EXE`
file — which you can then freely distribute or publish. Two different runtime
modules are created for each game: one for DOS, the other for Windows 95 /
DirectX.

There are three versions of ACKNEX — the **lite**, **commercial** and
**professional** version. They differ in the features supported and, of course,
in price. The features and royalty conditions are listed in
[[Publishing Your Game]].

This handbook serves as a reference. To familiarize yourself with the many
features ACKNEX offers, we suggest you also work through the
[[Tutorial Monday|tutorial]].

## The elements of a virtual world

The virtual world you create with 3D GameStudio is composed of the following
elements.

### Levels

A **level** is a part of a game, and can be compared to a two‑dimensional "map".
The graphics and other components of a level reside in RAM (or virtual memory) in
their entirety. After loading a level the world is precalculated and pre‑rendered,
which may take several seconds. Every change of level therefore results in a disk
access.

### Regions

The level is split into **regions**. A region is defined by the walls surrounding
it and by its floor and ceiling. You can think of it as a solid column of infinite
height, with a "gap" between the floor and the ceiling. It may have the shape of a
complex polygon. Each region may contain any number of further regions.

The heights and slopes of the floor and ceiling of a region are freely definable;
this way you can build staircases, gorges, etc. (one step of a staircase equalling
one region!). One source of light may be defined per region. You can stack regions
vertically to insert more "gaps" into the column — to construct bridges, suspended
objects, vehicles or multi‑floor buildings. Floors and ceilings have textures and
ambient light values, or may be given a backdrop (sky) texture. The same region may
appear several times within a level.

### Walls

The regions are bordered by vertical **walls** of any height, length and angle.
There must be a region on each side of a wall. Walls are represented by lines
connecting two vertices on the two‑dimensional map, and can be at any angle to one
another. Their visible heights — the parts of a wall you actually see — are
determined by the difference of floor and ceiling heights of the regions on both
sides. If both regions have the same floor and ceiling heights, the wall is hidden
within the floor and ceiling, and is therefore invisible.

```
            Ceiling
   +-------+        +--------+
   |Region |  Wall  | Region |
   |   1   |   1    |   3    |
   |       +--------+        |
   |        Region 2         |   <- middle region acts as a slanted "wall"
   +-------------------------+
            Floor
        Side view of three neighboring regions
```

Steps, windows or portals result from different heights of floors or ceilings.
Floors and ceilings may be inclined by the Z‑coordinates of their vertices. If the
ceiling of a region is given a sky texture, the open sky will be seen over that
region.

### Things and actors

In the level you can place objects (**things**) or living beings (**actors**) at
any position. The texture of such an object may change with the perspective, so
that the object — although only two‑dimensional — may appear to be spatial.

**Actors** are independently moving, "intelligent" objects. The behaviour and
properties of an actor are defined through the WDL language. With WED you can give
an actor a "**way**" to follow in the level. Each actor is a state machine with
determined actions, triggered by events.

### Textures

**Textures** are bitmaps of any desired size, which can cover walls, floors,
ceilings or objects. Textures may be animated, can have a sound, and can change
during gameplay (e.g. by being hit). Through the *attach* feature you can cover a
wall with as many "layers" of textures as you desire — for example to display
inscriptions or bullet holes. Textures are shaded in real time, depending on the
ambient light, their own reflectivity and their distance to the player. 256 colors
are available, including a transparent color for holes or windows.

---

The following chapters describe the commands and functions of the editor and of
the World Definition Language (WDL) as a reference listing. If you want to start
building a game level right away, first read the [[Tutorial Monday|tutorial]],
where you will find a step‑by‑step explanation of how to build a minimal level.
