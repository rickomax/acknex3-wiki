# acknex3-wiki

English (EN-US) documentation for **conitec 3D GameStudio** / the **ACKNEX** 3‑D
engine, transcribed from the original PDF manuals included in this repository:

- `ACKMAN.PDF` — Reference manual (ACKNEX v3.9, May 1998)
- `ACKTUT.PDF` — "Teach Yourself 3D Game Creating in 5 Days" tutorial
- `3dmanual.pdf` — earlier WED/WDL reference manual (v3.8, 1997)

(The `*_D.PDF` files are the German editions.)

## Wiki

The documentation lives in the [`wiki/`](wiki/) directory as GitHub‑Wiki‑ready
Markdown pages (page names use hyphens; `_Sidebar.md` provides navigation). Start
at [`wiki/Home.md`](wiki/Home.md).

To publish these as the repository's GitHub Wiki, clone the wiki repo and copy the
files in:

```sh
git clone https://github.com/rickomax/acknex3-wiki.wiki.git
cp wiki/*.md acknex3-wiki.wiki/
cd acknex3-wiki.wiki && git add -A && git commit -m "Import EN-US documentation" && git push
```

### Contents

**Reference manual:** Introduction · World Editor · ACKNEX 3D Engine · World
Definition Language · Palettes · Textures · Walls · Regions · Things, Actors and
Ways · Actions · Synonyms · Skills & Role Playing Games · Levels, Saving and
Multiplayer · User Interface · Publishing Your Game · Problems and Hints.

**Tutorial (5 days):** Monday (minimal level) · Tuesday (moving the player) ·
Wednesday (portals, elevators, waterfalls) · Thursday (artificial intelligence) ·
Friday (panels, menus, overlays).
