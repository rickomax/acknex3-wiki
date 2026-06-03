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

To let two players play as opponents (Deathmatch mode), link two PCs via a
nullmodem cable (serial laplink / Norton Commander / Interlink) or via a LAN with
IPX protocol. The **same level** must run on both computers.

To establish the link, give the serial port (`-COM n`) and the network number
(`-NODE 0` or `-NODE 1`) at start‑up. Omitting `-COM` assumes an IPX connection.
The `NODE` given at start‑up can be queried in actions with the `NODE` skill.

After starting, the engine checks whether an ACKNEX session is already running on
the serial link or IPX network. If so, it joins; otherwise it opens a new session.

In the level the opposing player is represented by an actor with `TARGET NODE0` or
`NODE1`, which performs the movements of the player on the PC with the matching
network number.

PCs linked in the simple serial multiplayer mode do **not** transmit actor
positions or changes to objects/regions. However, simple actions (firing, moving
lifts or doors) can be transmitted to the other PC by evaluating the `REMOTE`
skills.
