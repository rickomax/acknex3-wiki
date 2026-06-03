# World Editor (WED)

The world editor `WED.EXE` (`WEDS.EXE` in the lite version, `WEDC.EXE` in the
commercial version) is the integrated environment used to create and edit the
world topography. WED scans the WDL and — if one exists — the WMP file. It can
generate a runtime module and (with the professional version) compile all your
game files together into a single compressed resource file.

WED is a DOS application, but runs under Windows 95 as well. Start the world
editor with the following DOS command:

```
WED [WED options] name[.WDL] [name.WMP] [ACKNEX options]
```

If you omit the WMP name, WED loads the WMP file declared in the WDL script.
Should that file not exist, it will be created.

## Command line options

The ACKNEX (engine) options are discussed in [[ACKNEX 3D Engine]].

| Option | Effect |
|--------|--------|
| `-S` | SVGA resolution 800×600 |
| `-I` | SVGA resolution 1024×768 |
| `-VESA` | Use VESA BIOS for the graphic display (slower!). Use this if your video card has an unsupported chipset and the SVGA display seems distorted (e.g. Matrox Mystique). |
| `-VGA` | Start without an SVGA card. Floor and wall textures cannot be displayed in this mode. |
| `-RUN` | Run the game directly without switching to WED. Can also run a compiled game resource (`.WRS`) file. |

### Batch options (professional version only)

| Command | Effect |
|---------|--------|
| `WED -C name[.WDL] [name[.WDL]...]` | Compiles all level files `name.WDL` into the resource `name.WRS`. |
| `WED -CT name[.WDL] ...` | Like `-C`, but compiles all files — even those skipped by `IFDEF`/`IFNDEF`. This lets several versions of a level go into the same resource file. |
| `WED -X name[.WDL] directory` | Copies only the files needed to run `name.WDL` into the given directory. Useful to separate necessary game files from unneeded ones. |
| `WED -XT name[.WDL] directory` | Like `-X`, but copies all files — even those skipped by `IFDEF`/`IFNDEF`. |

## The map display

After start‑up WED shows the WMP file as a "map": walls are white lines, things
are green crosses, actors are red crosses, and a blue cross marks the player
start point. The scale is adjusted so the complete level fits on screen.

You work with WED like a CAD program. Objects are selected with the mouse; to
mark one, click with the left mouse button. The info panel at the bottom of the
screen shows the current mode and the parameters of the most recently
selected/marked object.

If the mouse pointer is positioned over an object it is automatically **selected**
with a yellow frame. Clicking the left button **marks** the selected object with a
green frame. You can drag a frame to mark several objects, or click on them with
the `[Shift]` key held. You can drag and drop the selected/marked object(s) with
the left button held.

In all modes you can zoom the section of the map under the pointer with keys
`[1]`..`[9]` in fixed steps. The `[Tab]` key jumps the selection mark from one
object to the next.

## Menu functions

### FILE `[Alt-F]`

| Function | Keys | Description |
|----------|------|-------------|
| OPEN WMP | `[F3]`, `[Ctrl-O]` | Loads a new WMP file with definitions based on the current WDL file. Created if it does not exist. |
| SAVE WMP | `[F2]`, `[Ctrl-S]` | Saves the current WMP file, creating a backup (`.BAK`). |
| SAVE AS | `Shift-[F2]` | Saves the current WMP file under a new name. |
| PUBLISH | `[Ctrl-P]` | Generates the freely distributable DOS and Windows 95/98 runtime modules (`WRUN.EXE` and `WWRUN.EXE`) for the current level. Use the first level of the game to create the runtime modules. `WWRUN.EXE` needs the definition files `WWRUN.MDF`, `WWRUN.WDF` in its directory to run. |
| COMPILE & PUBLISH | `[F9]` | Like PUBLISH, but also invokes the world compiler to create a world resource file (`.WRS`) from all files belonging to the level (professional only). Displays the 'Magic Key' number needed for royalty publication. Keep the **All IFDEFs** switch enabled to include all files. Runtime modules for a resource are named `VRUN.EXE` (DOS) and `WVRUN.EXE` (Windows 95/98). |
| EXIT | `[Alt-F4]`, `[Ctrl-Q]`, `[F10]` | Exits WED. |

### MODE `[Alt-M]`

| Function | Keys | Description |
|----------|------|-------------|
| VERTICES | `[V]` | Activates vertex mode. |
| WALLS | `[W]` | Activates wall mode. |
| REGIONS | `[R]` | Activates region mode. |
| THINGS | `[T]` | Activates thing/actor mode. |
| WAYS | `[Y]` | Activates way mode. |
| WALK‑THROUGH | `[Shift-^]` / `[Shift-~]` | Starts the game, or switches between game and editor while the game is running, so you can test your topography. Player position changes in WED are taken over instantly; changes to walls, things and actors also appear simultaneously in the game. |

### EDIT `[Alt-E]`

| Function | Keys | Description |
|----------|------|-------------|
| UNDO | `[BkSp]`, `[Ctrl-Z]` | Cancels the last operations one by one. |
| UNDO ALL | `[Ctrl-U]` | Cancels all changes since the last save. |
| COPY | `[Ctrl-V]` | Creates a copy of the selected object, displaced slightly to the lower right. |
| CREATE | `[Ins]`, Left Click | Creates a new object, depending on the current mode. |
| DELETE | `[Del]` | Deletes the selected object. |
| MODIFY | `[Return]`, Right Click | Changes the attributes (name etc.) of the selected/marked object(s). |
| IMPORT | `[Ctrl-I]` | Adds vertices and objects from an external `.WMP` file to the map. All names of the new objects must be defined in the current `.WDL` file! |
| EXPORT | `[Ctrl-E]` | Saves all marked objects (including belonging vertices) to a new `.WMP` file in the current directory. |
| ROTATE & SCALE | `[Ctrl-T]` | Rotates and scales the selected/marked object(s) by a given angle and percentage. |
| NEXT | `[Tab]` | Selects the next object. |
| PREV | `[Shift-Tab]` | Selects the previous object. |

### WALL `[Alt-W]` (wall mode only)

| Function | Keys | Description |
|----------|------|-------------|
| SPLIT | `[S]` | Splits the selected wall, inserting a new vertex in the middle. |
| JOIN | `[J]` | Connects several adjacent marked walls and erases the joint vertices. Properties of the first wall are assigned to the new wall. |
| FLIP | `[F]` | Reverses wall orientation by swapping start/end vertices and the left/right regions. Mirrors the wall texture; required for functions that treat the left and right sides differently. |
| RESET | `[Z]` | Resets horizontal and vertical alignments (X and Y offset) of all marked walls to zero. |
| ALIGN HOR | `[A]` | Aligns the textures of the right sides of connected walls horizontally so edges are seamless (by changing X offsets). Select a connected chain of walls; use FLIP if needed. |
| ALIGN CEILING | `[C]` | Aligns textures of marked walls vertically to the ceiling by changing Y offsets. Upper borders match the ceiling of the right‑hand region of the first marked wall. |
| ALIGN FLOOR | `[L]` | Aligns textures of marked walls vertically to the floor by changing Y offsets, relative to the right‑hand region of the first marked wall. |
| LENGTH | `[Ctrl-L]` | Assigns a defined length to a selected wall by moving its second vertex. Only one wall may be selected; a numeric window lets you set the length in steps. |
| RECTANGLE | `[Ctrl-R]` | Creates a quadrilateral of four walls at the cursor position; length set in a numeric window. |
| POLYGON | `[Ctrl-Y]` | Creates an n‑sided polygon of n walls at the cursor; number and side length set in a numeric window. |

### DISPLAY `[Alt-D]`

| Function | Keys | Description |
|----------|------|-------------|
| GRID | `[G]` | Switches the blue grid on/off. When on, vertices, things and actors can only be placed on grid points. |
| GRID+ | `[+]` | Doubles the scale of the grid. |
| GRID- | `[-]` | Halves the scale of the grid. |
| ZOOM+ | `[Num*]` | Increases the scale of the map (also keys `[0]`..`[9]`). |
| ZOOM- | `[Num/]` | Reduces the scale of the map. |
| SCROLL | `[Scroll Lock]` | Toggles autoscroll. With autoscroll off, the visible section can only be moved with the cursor keys. |

### CHECK `[Alt-C]`

| Function | Keys | Description |
|----------|------|-------------|
| KEYWORDS | `[Ctrl-K]` | Checks the current WMP names against the definitions in the WDL file. |
| STATISTICS | `[Ctrl-H]` | Shows the number of objects in the current level. |
| SCAN REGIONS | `[O]` | Checks all regions and switches to region mode. Necessary when new regions were created, e.g. by splitting a region with a wall. |
| CHECK WALLS | `[Ctrl-W]` | Checks the consistency of region‑to‑wall assignments. Suspicious/wrongly assigned walls are marked automatically. |
| DUPLICATES | `[Ctrl-D]` | Marks all regions, walls, things or actors with the same name as the selected object. |

### HELP `[Alt-H]`

| Function | Description |
|----------|-------------|
| ABOUT | Shows the version number. |

## Edit modes

### 1. Vertex mode `[V]`

You may place, move or erase vertices and connect them with lines that become
walls. Because of limited mathematical accuracy, the distance between a vertex and
an independent wall must be at least **0.25 steps** (more if the wall must be
visible from farther away). When tilting regions via vertex Z coordinates, do not
"bend" floor and ceiling areas, and keep the inclination below about 75°. Tilted
areas are not rendered very precisely, so diffuse textures are preferable there.

| Action | Effect |
|--------|--------|
| Left click on background | Places a vertex (shown as a green cross). |
| Left click on vertex | Marks the vertex. With `[Shift]`/`[Ctrl]`, previously marked vertices stay marked. A marked vertex has a green frame. |
| Left pull on background | Creates a frame to mark several vertices. |
| Left pull on vertex | Marks and shifts the vertex. With `[Shift]`/`[Ctrl]`, all other marked vertices move too. |
| Right click on vertex | Edit the Z coordinate (height) of a vertex, to create tilted regions. |
| `[Del]` | Erases all marked vertices. |
| `[Ins]` | Connects all marked vertices with wall lines (in the order marked) and switches to wall mode. Existing regions to the left/right are assigned automatically. |
| `[Shift]-[Ins]` | Connects all marked vertices (if more than two) into a closed polygon of wall lines. |

### 2. Wall mode `[W]`

Place, erase, shift, rotate or scale groups of lines. **Every wall MUST have a
name and a left and right region.** The right side of each wall is marked with a
small perpendicular "nose". Regions must be enclosed by walls on all sides —
regions touching without a wall between them cause visible errors in the picture!

| Action | Effect |
|--------|--------|
| Left click on wall | Marks the wall. With `[Shift]`/`[Ctrl]`, marked walls stay marked. A green frame appears. |
| Left pull on background | Creates a frame to mark walls. |
| Left pull on wall | Marks and shifts the wall with its vertices. With `[Shift]`/`[Ctrl]`, additionally marked walls shift too. |
| Right click on wall | Opens a scrollbox to assign a wall name from the WDL file. |
| `[Del]` | Erases all marked walls including their vertices. |

### 3. Region mode `[R]`

Assign previously defined regions to areas surrounded by walls, and alter floor or
ceiling heights. If you created new regions by placing walls, switch in with `[O]`
(SCAN REGIONS); otherwise `[R]` is sufficient.

| Action | Effect |
|--------|--------|
| Left click on region | Marks the region. With `[Shift]`/`[Ctrl]`, regions stay marked. A green frame appears. |
| Left pull on background | Creates a frame to mark several regions. |
| Left pull on region | Marks and shifts the region with its walls and vertices. With `[Shift]`/`[Ctrl]`, additionally marked regions shift too. |
| Right click on region | Opens a scrollbox to assign a region keyword from the WDL file. You can change region heights or use the defaults from the region definition. |
| `[Del]` | Erases all marked regions including their walls and vertices. |

### 4. Thing and actor mode `[T]`

Place things and actors, assign starting angles, and mark positions — for example
the player's starting position.

| Action | Effect |
|--------|--------|
| Left click on background | Creates a thing or actor (a circle with a green cross). |
| Left click on object | Marks the object. With `[Shift]`/`[Ctrl]`, objects stay marked. A green frame appears. |
| Left pull on background | Creates a frame to mark several objects. |
| Left pull on object | Marks and shifts the object. |
| Right click on object | Opens a scrollbox to assign the player starting position or a thing/actor keyword. |
| `[Ins]` | Creates an object (same as left click). |
| `[Del]` | Erases all marked objects. |

### 5. Way mode `[Y]`

Define a closed path of any number of waypoints for an actor. The path is shown as
a light‑blue dotted line connecting the waypoints.

| Action | Effect |
|--------|--------|
| Left click on background | Creates a new waypoint in the current way. It is inserted between the two closest connected waypoints, or connected to the last marked waypoint. |
| Left click on waypoint | Marks the waypoint. With `[Shift]`/`[Ctrl]`, waypoints stay marked. A blue frame appears. |
| Left pull on background | Creates a frame to mark several waypoints. |
| Left pull on waypoint | Marks and shifts a waypoint. |
| Right click on waypoint | Assigns a way name from a pop‑up list of defined way keywords. |
| `[Ins]` | Creates a new way and sets the first waypoint at the cursor. |
| `[Del]` | Erases all marked waypoints from the way; the remaining waypoints stay connected. |

## 6. WMP file format

The WMP file generated by WED contains plain ASCII text that can be edited with
any text editor (though it is best not to!). It lists all coordinates and objects
in declared sequence, in the following format:

```
VERTEX x y z;
```
Wall vertex definition. `x`, `y` and `z` are fixed‑point numbers — the X and Y
coordinate and the height of the vertex, in steps.

```
REGION name f c;
```
Region definition. `name` is the region keyword from the WDL file, `f` the floor
and `c` the ceiling height.

```
WALL name v1 v2 rr rl sx sy;
```
Wall position. `name` is the assigned wall keyword from the WDL file. `v1` and
`v2` are the consecutive numbers of the two corner vertices. `rr` and `rl` are the
consecutive numbers of the right and left regions (relative to the view from `v1`
to `v2`). `sx` and `sy` are the horizontal and vertical texture offsets in pixels
(changeable with the ALIGN functions).

```
THING name x y a r;
```
Thing position. `name` is the WDL thing keyword, `x`/`y` the coordinates, `a` the
angle (0…360), and `r` the region the thing inhabits.

```
ACTOR name x y a r;
```
Actor position. `name` is the WDL actor keyword, `x`/`y` the starting coordinates
in steps, `a` the angle (0…360), and `r` the region the actor inhabits.

```
PLAYER_START x y a r;
```
Starting position, angle and region of the player.

```
WAY name x1 y1 x2 y2 ... xn yn;
```
Way definition. `name` is the assigned way keyword, followed by the coordinate
pairs for the waypoints.
