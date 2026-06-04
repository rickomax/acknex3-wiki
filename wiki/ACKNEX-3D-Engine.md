# The ACKNEX 3D Engine

This is the core of the software — the 3D rendering engine that runs the game.
It is integrated in WED as well as in the runtime modules for DOS and Windows 95.
You can start it by running WED with the `-RUN` option, by selecting
**WALKTHROUGH**, or by pressing `[F11]` from the WED window.

## Engine command line options

Apart from the file name, the following WED command line options are available:

| Option | Effect |
|--------|--------|
| `-WMP name` | Uses the topography from `name.WMP`, even if a different file was defined with `MAPFILE`. |
| `-Dir name` | Uses the given directory for the game score, even if a different one was defined in the WDL. |
| `-D name` | Defines the token `name` for evaluation in the WDL file by `IFDEF`. |
| `-NODE n` | Number of the network node (0…1) for multiplayer games (commercial/professional only). |
| `-COM n` | Port number of the serial link (1…4) for multiplayer communication. The serial interface may not be used by the mouse and must have a free interrupt. COM1/COM3 use INT4; COM2/COM4 use INT3. |
| `-MODEM` | Connect via modem for 2‑player mode (professional Windows version only). Enter the modem and phone number in the box that appears after game start. |
| `-TCP` | Connect via a network or the internet using the TCP/IP protocol (professional Windows version only). Enter the session's server address in the box that appears after game start. |
| `-IPX` | Use a LAN with IPX protocol (e.g. Novell) instead of a serial link for multiplayer (professional Windows version only). |
| `-OS` | Run without sound. |
| `-OM` | Run without music. |
| `-OCD` | Run without activation of audio CDs. |
| `-OT` | Test run without sound, music and timer. |
| `-RO` | Use the MPU401 (if available) for wavetable music instead of FM sound. |
| `-NJ` | Disable the joystick; useful to avoid joystick port problems with the DOS engine under Win95. |
| `-NM` | Disable the mouse. |
| `-C` | Syntax check mode; checks only the syntax of the WDL and WMP files. |
| `-E` | Number of error messages. Usually up to 6 are returned, after which a key press is required. |
| `-W1` | Extended syntax check, with warnings for potential errors in the WDL file. |
| `-W2` | Like `-W1`, but also shows unused objects and superfluous definitions. |
| `-SST` | Single step mode; press `[S]` after each frame. |
| `-GOD` | God mode: the player can pass through massive walls and objects (be careful — may crash!). |
| `-NC` | No `CLIP_DIST` check. |
| `-WND` | Start in window mode instead of fullscreen. |
| `-3D_SIMEYE` | Start in 3D stereoscopic mode with SimulEyes® shutter glasses (professional Windows version only; not supported by all cards/drivers). |

## Debug key combinations

With the game running you can toggle special debugging modes. These are disabled
within the runtime modules.

| Keys | Effect |
|------|--------|
| `[Ctrl]-[Alt]-[G]` | Toggles God mode. |
| `[Ctrl]-[Alt]-[C]` | Toggles `CLIP_DIST` check. |
| `[Ctrl]-[Alt]-[S]` | Toggles single step mode. The game "freezes" and can be continued frame by frame with `[S]`. |
| `[Ctrl]-[Alt]-[End]` | Quits the game. |
| `[Alt]-[Return]` | Toggles between fullscreen and window mode. |
| `[F11]` | Switches back to WED. |

## Default in‑game keys

If not redefined by WDL, the following keys are active within the game:

| Key | Effect |
|-----|--------|
| `[F2]` | Saves the game into `TEST_0.SAV`. |
| `[F3]` | Loads the last saved game. |
| `[F5]` | Toggles motion blur. |
| `[F6]` | Takes a screenshot into `SHOT_n.PCX`. |
| `[F10]` | Quits the game. |
| `[F12]` | Toggles music and sound. |
| `[Tab]` | Toggles the automap. |
| `[0]` | Activates a default player movement allowing flying and vertical movement (disabled within the runtime module). |
