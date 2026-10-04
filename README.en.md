# WorldForge Standard — Arma Reforger map generator

> Standard edition: unlimited **procedural** maps. Support adds real-site tracing from a map (see below).

## ▶ Your first map in 3 minutes

[![Video: first test map](assets/images/preview-procedural.png)](https://streamable.com/jfb5mr)

### **[▶ Watch: how to create the first test map](https://streamable.com/jfb5mr)**

From recipe to addon, step by step.

## What it is

Give it a JSON recipe (or click through the UI) and it writes an addon that Workbench opens: baked terrain, hydrology, properly-profiled roads, forests, towns with streets, missions and `.gproj`.

It doesn't copy anyone's `.terr`: it writes the whole thing from scratch with its own non-colliding GUIDs.

![Procedural example](assets/images/preview-procedural.png)
*8 km temperate island from `recipes/everon_like.json`.*

## Everything included

**Free and complete.** There is no cut-down edition and nothing is locked:
everything here works.

| | |
|---|---|
| Unlimited procedural + 13 templates | ✅ |
| Image mode (your satellite photo → coastline and woodland) | ✅ |
| Preview, Build, Cook, Preflight, Catalog | ✅ |
| **Realistic map: pick the area and it gets traced** | ✅ |
| Copernicus DEM elevation | ✅ |
| ESA WorldCover ground cover | ✅ |
| OSM roads, watercourses and houses | ✅ |
| Import Arma 3 `.wrp` | ✅ |

### Procedural

![Procedural](assets/images/standard-empezar.png)

Start: random, template, custom procedural, from your own image. Biomes:
`everon`, `arland`, `montane`, `arid`, `texas`.

![Editor](assets/images/standard-editor.png)
*Recipe editor: every field with its help text, live validation and a summary
of what will come out.*

### Realistic map

![Realistic map](assets/images/support-mapa-realista.png)

Drag, wheel-zoom, the orange square is your map (2×2 – 16×16 km) → *Create
recipe and preview*. It traces the relief from the Copernicus DEM, the ground
cover from ESA WorldCover and, from OpenStreetMap, the roads, the watercourses
and the building footprints: **around 10,000 houses in 8 km over a city, with
their real plan and rotation**.

## Download

From **Releases**: full `WorldForge` folder (`WorldForge.exe` 82 MB + `WorldForgeTools/` + `recipes/` + `locales/` + `docs/`). Don't download the lone `.exe`.

Requirements: Windows 10/11 64-bit, Arma Reforger Tools.

## First run

### You do not need Python

The `.exe` carries its own Python, numpy, scipy and Tk inside. Unzip and
double-click: there is nothing to install.

What you **do** need:

| | For what | Required? |
|---|---|---|
| **Arma Reforger** | Opening the map you made | Yes, to play it |
| **Arma Reforger Tools** | Cooking (navmesh, player map, generators) | Yes, to finish a map |
| Python | — | **No** |

> **The Tools are a separate entry in Steam**, not part of the game. Search
> for *Arma Reforger Tools*. Without them the whole addon and the terrain are
> still generated, but you cannot cook, and a map that is not cooked opens
> with no AI navigation and an empty player map.

### Unzip the whole folder

The `.exe` on its own does not work: it needs `WorldForgeTools/` (the plugin
that bakes the generators) and `recipes/` beside it. Put it somewhere you can
write to, **not** in `C:\Program Files`.

### Settings: the four paths

This is the step people skip, and then nothing cooks.

- **WorldForge project** — the folder above. Found on its own.
- **Folder for generated addons** — where your maps get written. Put it next
  to the WorldForge folder so the Workbench's `-addonsDir` finds them by
  itself.
- **Workbench executable** — press **Detect Steam**. Otherwise the path is
  usually `C:\Program Files (x86)\Steam\steamapps\common\Arma Reforger Tools\Workbench\ArmaReforgerWorkbenchSteamDiag.exe`.
  **If Steam is on another drive the factory path is wrong**, and this is by
  far the #1 reason cooking fails.
- **Base game addons** — normally
  `C:\Program Files (x86)\Steam\steamapps\common\Arma Reforger\addons`.
  Without it the Workbench dies with `Game addon '58D0FB3206B6F859' not found`.

Press **Save paths**: the dots under *Environment status* turn green. The last
two also work as the `AR_WB_EXE` and `AR_GAME_ADDONS` environment variables.

### Check it all works

**Start → Random map → Generate and build**, then **Cook**. The Workbench
opens and closes on its own, once per step. Tick *Skip the navmesh* if you
only want a look, and *Open the world when it finishes* so the World Editor
opens on it. If that works end to end, everything is set up.

## Video guide

Full `test` map generation (Verdania, 8 km): from recipe to Workbench-ready addon.

[▶ Watch: how to create the first test map](https://streamable.com/jfb5mr)

## Use

```powershell
WorldForge.exe                         # UI
WorldForge.exe --cli preview recipes/everon_like.json --out preview.png
WorldForge.exe --cli build recipes/everon_like.json --out C:/Desarrollo/MiMapa
WorldForge.exe --cli check C:/Desarrollo/MiMapa
WorldForge.exe --cli cook C:/Desarrollo/MiMapa
```

Order: Build → Cook → Preflight → open in Workbench:

```powershell
& "C:/Program Files (x86)/Steam/steamapps/common/Arma Reforger Tools/Workbench/ArmaReforgerWorkbenchSteamDiag.exe" `
  -gproj "C:/Desarrollo/MiMapa/addon.gproj" `
  -addonsDir "C:/Program Files (x86)/Steam/steamapps/common/Arma Reforger/addons,C:/Desarrollo" `
  -wbmodule=WorldEditor -run -load "worlds/MiMundo/MiMundo.ent"
```

Minimal recipe:

```json
{
  "name": "Verdania", "seed": 1979,
  "size_m": 8192, "cell_size": 2.0,
  "landform": "island", "relief": "rolling",
  "biome": "everon", "forest_cover": 0.34,
  "settlement_density": 1.0, "game_modes": ["GM"]
}
```

`--set` overrides without editing: `--set seed=42 --set relief=mountainous`.

## Docs

- `docs/TEMPLATES.md` — 13 templates + field table.
- `docs/PIPELINE_EN.md` — build/cook phases and why in that order.
- `recipes/` — copy/paste, change `name` + `seed`.

## Language

Spanish by default, English included. Auto from Windows, switch in Settings (restarts).

## Support the project

WorldForge is **free and complete**: everything above works and nothing is
held back.

If it saves you time and you want to keep it moving, **new and improved
versions go to supporters first**:

- ☕ **Ko-fi:** <https://ko-fi.com/arribas>
- 💬 **Discord:** `alv4r0` — message me after donating and I'll send you the
  new builds.

No keys, no activation, no nagging.

## License

See [LICENSE](../LICENSE). Free to use to generate maps. No generator source in this repo.
