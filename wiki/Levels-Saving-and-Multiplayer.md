# Changing Levels, Saving the Game, and Multiplayer Mode

## Changing levels

Changing levels with the `LEVEL` instruction corresponds to **restarting** the
game. All skills and parameters are reloaded from the WDL file, **except** the
`LOCAL` and `GLOBAL` skills. For this to work, the new and old levels must define
exactly the same `LOCAL` and `GLOBAL` skills in the same order (the "normal"
skills may differ). `GLOBAL` skills let you buffer parameters and "transport" them
to the new level — e.g. the player's combat prowess and equipment.

Changing only the topography with the `MAP` instruction keeps all skills except
the player position. All walls, things, regions and actors are loaded from the new
WMP file.

### Passing `DEFINE`s to a new level (`COMMAND_LINE`)

The predefined string synonym `COMMAND_LINE` passes a single `DEFINE` parameter to
a level started with `LEVEL`, via a `-d` option (or `-u` to *undefine* a parameter
that was previously `DEFINE`d). Command‑line `-d` definitions are otherwise global
— they persist across all further levels started with `LEVEL` or `MAP`.
```
STRING svga_str,    "-d SVGA";
STRING no_svga_str, "-u SVGA";

ACTION start_level_with_svga {
    SET   COMMAND_LINE, svga_str;
    LEVEL <newlevel.wdl>;
}
ACTION start_level_without_svga {
    SET   COMMAND_LINE, no_svga_str;
    LEVEL <newlevel.wdl>;
}
```

## Saving and loading

The `SAVE` and `LOAD` instructions save and reload game scores. The following are
saved:

- the names of the current `.WDL` and `.WMP` files;
- all `PLAYER` and `GLOBAL` skills (`LOCAL` skills are **not** saved);
- all synonyms;
- the state of the `EACH_TICK` list;
- all key actions (`IF_F1` etc.);
- the current state of all running actions;
- the positions and parameters of every `VIEW`, `PANEL`, `TEXT` and `OVERLAY`;
- the positions, skills and parameters of all actors;
- the flags of all regions, walls and things;
- the positions, skills and parameters of any region, wall, thing and texture for
  which the `SAVE` flag is set.

To save the `ATTACH` texture of an object, set that texture's `SAVE` flag.

All `EACH_TICK` actions running at save time are **started anew** after reloading.
If another level is running at load time, levels are changed automatically. Like
`LEVEL` and `MAP`, the actual loading happens at the **end** of the action, not
directly at the `LOAD` instruction.

The `SAVE_INFO` / `LOAD_INFO` instructions work exclusively on `GLOBAL` skills,
saving general parameters (sound volume, screen resolution) independently of a
game score. Bitmaps changed with `FREEZE` are also saved this way. (Level changing
and game saving are discussed in more detail in the [[Tutorial Wednesday|tutorial]].)

## Multiplayer mode

To let two players play as opponents (Deathmatch mode), link two PCs. The **same
level** must run on both computers. Since the Spring 1998 (professional Windows)
release, four connection methods are available — start each PC with `-NODE 0` or
`-NODE 1` plus one of:

| Option | Connection |
|--------|------------|
| `-COM n` | Serial nullmodem cable (laplink / Norton Commander / Interlink) on port *n* |
| `-MODEM` | Modem (enter the modem and phone number in the box after game start) |
| `-TCP` | Network or internet via TCP/IP (enter the session's server address) |
| `-IPX` | LAN via IPX protocol (e.g. Novell) |

The `NODE` given at start‑up can be queried in actions with the `NODE` skill. To
try the network mode: define two actors (one `TARGET NODE0`, one `TARGET NODE1`),
build a Windows runtime module with PUBLISH, then start `WWRUN levelname -NODE 0
-IPX` on the first PC, and once it is running, `WWRUN levelname -NODE 1 -IPX` on
the second.

After starting, the engine checks whether an ACKNEX session is already running on
the serial link or IPX network. If so, it joins; otherwise it opens a new session.

In the level the opposing player is represented by an actor with `TARGET NODE0` or
`NODE1`, which performs the movements of the player on the PC with the matching
network number.

PCs linked in the simple serial multiplayer mode do **not** transmit actor
positions or changes to objects/regions. However, simple actions (firing, moving
lifts or doors) can be transmitted to the other PC by evaluating the `REMOTE`
skills.

### `REMOTE_KEYS`

> Documented in the v3.8 manual; omitted from the v3.9 reference listing.

```
REMOTE_KEYS Keyword11.Keyword12, Keyword21.Keyword22, ...;
```
A list, analogous to `EACH_TICK`, for transmitting parameter changes in
multiplayer games. After a parameter changes, this list is checked after every
picture build‑up and the changed parameters are transmitted by network, modem or
serial port to the connected nodes. Keep the list short to save computing time.
