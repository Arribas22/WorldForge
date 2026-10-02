# WorldForge Standard — Arma Reforger map generator

> Standard edition: unlimited **procedural** maps. Premium adds real-site tracing from a map (see below).

## What it is

Give it a JSON recipe (or click through the UI) and it writes an addon that Workbench opens: baked terrain, hydrology, properly-profiled roads, forests, towns with streets, missions and `.gproj`.

It doesn't copy anyone's `.terr`: it writes the whole thing from scratch with its own non-colliding GUIDs.

![Procedural example](assets/images/preview-procedural.png)
*8 km temperate island from `recipes/everon_like.json`.*

## Standard vs Premium — with screenshots

### Standard: procedural + your image (included here)

- **5 ways in Start:** Random map · From template (13) · Custom procedural · From your image · (the 2 real-site ones are Premium, shown with a badge).
- **13 templates in `recipes/`:** `everon_like`, `arland_like`, `archipelago`, `valle_montana`, `alta_montana`, `arid_plateau`, `costa_urbana`, `interior_agricola`, `bosque_cerrado`, `peninsula`, `isla_grande`, `montane_valley`, `texas_like`.
- **Image mode:** your PNG/JPG drives coast + forest; add a grayscale heightmap and it drives elevation too.
- **Full pipeline:** Preview, Build, Cook (Workbench via CLI: shore map, rivers, generators, `.topo`, triple navmesh, BSP), Preflight, Catalog, Tools, Docs.
- **Biomes:** `everon`, `arland`, `montane`, `arid`, `texas`. Seed fixes everything (same JSON = same map byte-for-byte).

📷 Drop your Standard screenshots here:
- `assets/images/standard-empezar.png` — Start
- `assets/images/standard-editor.png` — Recipe editor
- `assets/images/standard-preview.png` — Preview
- `assets/images/standard-build.png` — Build / Preflight OK

### Premium: realistic map (pick the area on the map)

![Premium realistic map](assets/images/premium-mapa-realista.png)
*Realistic-map screen: drag to pan, wheel to zoom, orange square is your map. Right: Go-to (search + Everon, Fort Bragg, Alcalá de Henares, Normandy, Kyiv), Size (2×2 to 16×16 km), Edge (Automatic / always water / land to edge), “Create recipe and preview” button.*

*(If broken, save your screenshot as `assets/images/premium-mapa-realista.png`).*

- **Real sources:** Copernicus DEM elevation, ESA WorldCover, OpenStreetMap roads + waterways + landuse + building footprints (~10,000 houses over a city at 8 km, real footprint + rotation). Sentinel-2 color.
- **Sizes:** 2×2, 4×4, 8×8 (recommended), 12×12, 16×16 km. 8 km already fits city + region; cooking scales with side².
- **Edge:** Automatic (inland=land, coast=water), Water ring, Land to edge (cut visible — normal in Reforger).
- **On Standard** this screen shows a “Real-site tracing is Premium” card pointing to templates/procedural.

| | Standard | Premium |
|---|---|---|
| Procedural (13 templates, biomes, relief, towns) | ✅ | ✅ |
| Image mode (your satellite → coast & forest) | ✅ | ✅ |
| Preview, build, cook, preflight, catalog | ✅ | ✅ |
| Copernicus DEM elevation | — | ✅ |
| ESA WorldCover | — | ✅ |
| OSM roads, rivers & houses | — | ✅ |
| Arma 3 `.wrp` import | — | ✅ |

## Download

From **Releases**: full `WorldForge-Standard` folder (`WorldForge.exe` 49 MB + `WorldForgeTools/` + `recipes/` + `locales/` + `docs/`). Don't download the lone `.exe`.

Requirements: Windows 10/11 64-bit, Arma Reforger Tools.

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

- `docs/PLANTILLAS.md` — 13 templates + field table (Spanish, auto-translatable).
- `docs/PIPELINE.md` — build/cook phases and why in that order.
- `recipes/` — copy/paste, change `name` + `seed`.

## Language

Spanish by default, English included. Auto from Windows, switch in Settings (restarts).

## Premium

Standard + real-site + `.wrp`. Needs `worldforge.lic` next to the `.exe`. To request it open a “Premium” Issue.

## License

See [LICENSE](../LICENSE). Free to use to generate maps. No generator source in this repo.
