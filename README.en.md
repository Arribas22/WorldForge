# WorldForge — Arma Reforger world generator

> Unlimited **procedural** maps and tracing of real places on Earth. Free and complete.

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

## Porting an Arma 3 map

If you have an Arma 3 world, WorldForge rebuilds it in Reforger. It is three
independent pieces and you can use whichever you want:

| In the recipe | What it brings from the original |
|---|---|
| `heightmap.wrp` | **The relief.** Read straight from the `.wrp`. If the Arma 3 map is the same size as yours it is 1:1, no scale model. |
| `objects_wrp` | **The roads and the buildings**, in their real positions. On Jackson County: 2,203 roadway pieces chained into 88 paths, and 855 buildings. |
| `places_file` | **The settlements**, from the `.hpp` `class Names`: name, position, radius and size. On Jackson County, 40 named places. |

```json
{
  "name": "JacksonCounty", "size_m": 10240, "cell_size": 2.0,
  "heightmap":   { "wrp": "C:/.../Jackson_County.wrp" },
  "objects_wrp":        "C:/.../Jackson_County.wrp",
  "places_file":        "C:/.../Jackson_County.hpp"
}
```

Whatever does **not** come from the original is still generated: forests,
fences, power lines and the rest. And if the original map has unreachable
areas, `road_links` adds the roads it is missing (below).

> The radius in an `.hpp` describes the **label** of the place on the map, not
> its built-up area, and tends to be generous: you get 500 m towns where
> there is a hamlet. `settlement_radius_scale` at 0.6–0.8 puts it back.

## Vertical scale: the trap of real-world maps

**This ruins maps and gives no error at all.** If the real box you mark is
245 km across and your map is 12.8, the horizontal shrinks 19 times. If the
height does not shrink with it, **every slope comes out multiplied by 19**:
the map looks great in the editor and cannot be driven.

`heightmap.height_scale` fixes it. You do not have to work it out: in
*Realistic map* press **Work out the recommended scale** and it fills it in.
Three to four times of exaggeration is what leaves a compressed map that
reads well and drives well; 1.0 is the real height and only makes sense if
the box is the same size as your map.

## More things it does

**Multi-region atlas.** One map can be made of several real boxes placed at
specific spots, each with its own vertical scale. That is how the Spain
recipe is built (mainland, Balearics and Canaries in a single 12.8 km map),
and the Mexico one.

**Exact paths.** `road_paths` lays a road down as given, with no router and no
A\*. That is what a circuit needs: for the Nürburgring the geometry **is** the
product and no algorithm gets to choose it.

**Road links.** `road_links` adds the roads the map does not have and needs to
be driveable. Each entry is a route: place names or `x, z` pairs. On an
archipelago, `road_links_water` with a steep cost (150 works well) makes the
router cross at the narrowest strait instead of leaving each island with its
own loose network.

**Real villages.** Houses face the street, carry a plot fence (61% on Everon,
measured) and clutter in the yard; harbours appear next to coastal
settlements and power lines follow the roads.

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
