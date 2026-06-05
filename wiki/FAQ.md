# Frequently Asked Questions

> A collection of practical questions and answers from the **ACKNEX User's
> Magazine (AUM)** forum. They complement the [[Introduction|reference manual]]
> with real‑world solutions. Some answers reference the original `move.wdl` /
> demo files shipped with 3D GameStudio.

### Visible bullets / fireballs from an actor

> *Q: I want the bad guys to shoot a visible bullet (like a fireball) that hurts
> whatever it hits.*

Let the bad guy `SHOOT` the player to get the angle relative to the player
(`SHOOT_ANGLE`); then start a fireball with `TARGET BULLET` and that angle. You can
get the `VSPEED` by dividing the vertical distance by the horizontal distance.

### Keeping multiple bullets on screen at once

> *Q: Whenever I shoot twice, the old flying bullet disappears. How can the old
> bullet stay while the new one appears?*

Define several bullets — say 10. Before emitting one, an action scans the bullets
to find which are still flying (`INVISIBLE` not set) and which are free
(`INVISIBLE` set). See `INTERRUP.WDL` for the idea.

### Making one enemy's shot affect another enemy

> *Q: How can one enemy's shooting trigger another enemy's `IF_HIT`?*

It will, if the shooter emits a bullet actor. Two approaches:
1. Beam the player to the target enemy's position, let the first enemy `SHOOT`; if
   `HIT_DIST > 0`, call the second enemy's `IF_HIT` action, then place the player
   back.
2. Set the `FRAGILE` flag on the target enemy and use `EXPLODE` instead of `SHOOT`.

### An actor that keeps moving in its last direction

> *Q: I need an actor (moving away with `REPEL`) to keep going at its last angle
> until it hits a wall and triggers `IF_HIT`.*

`TARGET BULLET` keeps the actor's old angle and speed, and triggers `IF_HIT` on
collision.

### Persisting level state when the player returns

> *Q: If the player kills everyone in a level and comes back later, can the level
> stay the way they left it?*

Yes. Save the level under a special name on exit, and on returning use `LOAD`
instead of `MAP`.

### Blank (black) screen on a trial run

> *Q: When I run my level all I see is a blank screen.*

A black screen usually means you forgot to define a color **palette** for the
level, or you placed the player's start position **outside the map**.

### Fog that also affects the outdoor sky

> *Q: Fog works great indoors, but outdoors the SKY texture isn't affected.*

The sky isn't affected by shading. For fog, use a **monocolored sky bitmap** of
the same color as `#1` of the palette — in the distance you then see only the fog
color.

### NEXUS errors ("slices too small / too many walls")

> *Q: I get NEXUS errors on a huge 948‑wall outdoor level.*

Increase `NEXUS` from its default of 14 to a higher value (e.g. 50) until the
errors stop.

### Moving a weapon image past the screen edge

> *Q: My weapon overlay can't exceed the screen boundaries.*

Don't use an `OVERLAY` — use a `PANEL` with the `TRANSPARENT` flag for the weapon.
Panels draw slower but can exceed the screen; overlays have no clipping code.

### Crouch / duck on any key

> *Q: I assigned jump, but how do I assign a key for duck/crouch?*

In `move.wdl`, ducking uses the `FORCE_UP` skill (set negative by `[End]`). Assign
an action such as `RULE FORCE_UP = -KEY_SENSE;` to, e.g., `IF_X`. Note that `IF_Z`
and `IF_Y` are swapped on US and some European keyboards.

### Single‑key toggle (e.g. a flashlight)

> *Q: How do I make a single key toggle an action on and off?*

```
ACTION light_toggle {
    RULE PLAYER_LIGHT += 1;
    IF (PLAYER_LIGHT > 1) {
        SET PLAYER_LIGHT, 0;
    }
}
```
This toggles the player's light between 0 and 1.

### Rotating a floor texture

> *Q: How do I rotate a floor texture?*

Change the region's `FLOOR_ANGLE` by an action.

### Player jumps up stairs instead of gliding

> *Q: Going up stairs, the player jumps each step suddenly. How do I make it glide?*

The player does glide — perhaps too fast. Adjust the move action (see the
[[Tutorial Tuesday|tutorial]]): reduce the `knee_fac` skill.

### A swinging door under a door frame glitches

> *Q: My door (a region rotated with `ROTATE`) messes up when it leaves the frame
> region.*

Make the frame large enough that the walls never cross.

### Very large levels

> *Q: Can I make a field so big it takes minutes to cross?*

Yes — there is no size limitation. It's a good idea to split long walls into
smaller ones.

### Which MDL editors exist?

> *Q: Is there a tool other than QMe to create MDL models and animation?*

Known MDL editors: **QMe, Meddle, Quark, Qmetal**. Look for Quake tool pages and
pick the one that suits you best.
