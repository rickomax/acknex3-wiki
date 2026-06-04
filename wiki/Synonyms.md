# Synonyms

Frequently one action needs to change several objects or regions. To avoid
writing an action for each, you can use a **synonym** instead of the keyword
naming the object/region. Some synonyms are predefined (`MY`, `HERE`, `THERE`);
any number of further synonyms may be defined by the user:

```
SYNONYM Keyword { ... }
```

A synonym definition may contain — in the sequence given:

| Keyword | Meaning |
|---------|---------|
| `TYPE Keyword;` | The type of synonym: `OVERLAY`, `TEXTURE`, `WALL`, `THING`, `ACTOR`, `REGION`, `PANEL`, `TEXT`, `STRING`, or `ACTION`. |
| `DEFAULT Keyword;` | Optional: the object the synonym stands for at game start. |

You can use synonyms like normal keywords. Action synonyms can be started by
`CALL`. Synonyms can be assigned to each other:
```
REGION lift_1 { ... }
SYNONYM lift_syn1  { TYPE REGION; }
SYNONYM lift_syn2  { TYPE REGION; }
SYNONYM lift_floor { TYPE TEXTURE; }
ACTION lift1_to_lift2 {
    SET lift_syn1, lift_1;
    SET lift_syn2, lift_syn1;
    SET lift_floor, lift_syn2.FLOOR_TEX;
}
```

## Predefined synonyms

> *Notation:* `<` marks a value changeable during gameplay; `#` marks a value set
> automatically.

| Synonym | Refers to… |
|---------|------------|
| `<HERE` | The region the player is in — important for collision detection and height adjustment. If the player changes region by directly setting `PLAYER_X`/`PLAYER_Y`, `HERE` may need manual reassignment. |
| `<THERE` | The region that triggered the current action (via `IF_ENTER`, `EACH_TICK`, etc.), or the region of the triggering object (`MY`). |
| `<MY` | The object that triggered the current action (via `IF_NEAR`, `IF_FAR`, `EACH_TICK`, etc.), allowing direct addressing without knowing the index (e.g. `SET MY.TEXTURE, crash_tex;`). Undefined if the action wasn't triggered by an object event. |
| `#HIT` | The object last hit by the player's `SHOOT`, last clicked with the mouse, or hit by the last `EXPLODE` (closest to the explosion center). |
| `#TOUCHED` | The object last touched by the mouse. |
| `#TOUCH_TEX` | The texture last touched by the mouse. |
| `#TOUCH_REG` | The region whose floor or ceiling was last touched by the mouse. |
| `<TOUCH_TEXT` | Assigning a `TEXT` here makes all `TOUCH` texts appear with that text's font and flags. If the text's `VISIBLE` flag is set, `TOUCH` texts appear at its `POS_X`/`POS_Y` instead of at the mouse position. |
| `<COMMAND_LINE` | A `STRING` synonym whose contents are passed as a command‑line `-d`/`-u` `DEFINE` parameter to the next level started with `LEVEL`. See [[Levels Saving and Multiplayer]]. |
