# Map templates

Copy the block closest to what you want, paste it into
`recipes/<name>.json`, change `name` and `seed`, and go:

```powershell
py -3 -m worldforge preview recipes/<name>.json --out preview.png   # look first
py -3 -m worldforge build   recipes/<name>.json --out C:/Desarrollo/MyMap
py -3 -m worldforge cook    C:/Desarrollo/MyMap                      # shore map, navmesh, BSP
```

**Look at the `preview` before generating.** It costs half, writes nothing, and
changing `seed` until the coast looks right is faster than fighting parameters.

> Versión en español: [PLANTILLAS.md](PLANTILLAS.md).

---

## First: what can NOT be done

So you don't waste time trying.

- **No snow maps.** Arma Reforger 1.8 ships 55 terrain surfaces and
  **none is snow or ice**: grass, mountain grass, heather, rock,
  sand, gravel, dirt, crop, asphalt and concrete. The closest you get
  is bare high mountain (rock and `MountainGrass`) above the treeline. A real
  snow map needs custom `.emat`, which is a different job: making
  textures, not generating the map.
- **No cold weather or seasons.** The `_aut` surface suffixes are
  the autumn variant, not winter.
- **Each 64 m block takes at most five materials.** Not our limit:
  Everon and Jackson both top out at exactly five. Painting is already
  per-cell (quadtree, see `docs/FORMATS.md`), but if one area joins road,
  forest, field, rock and town, the sixth material is dropped and shared
  between the five largest. In practice you barely notice; where it would
  show is a road crossing inside a town inside a forest.
- **No bridges.** Water is untraversable for routing: a settlement on an
  islet gets no road, on purpose.

---

## Sizes and times

`size_m / cell_size` is the cell count per side, and it decides whether the
map takes a minute or half an hour.

| for | `size_m` | `cell_size` | cells | time |
|---|---|---|---|---|
| trying an idea | 2048 | 2.0 | 1025 | ~30 s |
| small map, Arland-style | 4096 | 2.0 | 2049 | ~1 min |
| the normal size | 8192 | 2.0 | 4097 | ~7 min |
| big, Everon-style | 12800 | 2.0 | 6401 | ~20 min |
| fine detail | 4096 | 1.0 | 4097 | ~7 min |

If they ask for a huge map with fine detail, **raise `cell_size` before
lowering `size_m`**: 2 m per cell is what Everon and Arland use and it looks
fine.

---

## 1. Temperate island, Everon-style

The default case: gentle hills, mixed forest, towns in the valleys and on the
coast. If you don't know what you want, start here.

```json
{
  "name": "Verdania",
  "seed": 1979,
  "size_m": 8192,
  "cell_size": 2.0,
  "landform": "island",
  "relief": "rolling",
  "land_fraction": 0.66,
  "erosion": 0.9,
  "biome": "everon",
  "forest_cover": 0.34,
  "settlement_density": 1.0,
  "day_time": 9.5,
  "game_modes": ["GM"]
}
```

## 2. Atlantic island, Arland-style

Smaller, barer and more broken: heather, scattered pine and lots of coast.

```json
{
  "name": "Brannoc",
  "seed": 4711,
  "size_m": 4096,
  "cell_size": 2.0,
  "landform": "island",
  "relief": "hilly",
  "land_fraction": 0.58,
  "erosion": 0.85,
  "biome": "arland",
  "forest_cover": 0.18,
  "settlement_density": 0.9,
  "game_modes": ["GM"]
}
```

## 3. Archipelago

Several islands with no isthmus. **Each landmass gets its own road network**
and isolated settlements stay cut off, because there are no bridges. If what
you want is a big island with indentations, raise `land_fraction` to 0.6 and
use `island`.

```json
{
  "name": "Islas",
  "seed": 7,
  "size_m": 4096,
  "cell_size": 2.0,
  "landform": "archipelago",
  "relief": "hilly",
  "land_fraction": 0.38,
  "erosion": 0.9,
  "biome": "everon",
  "forest_cover": 0.28,
  "settlement_density": 1.4,
  "game_modes": ["GM"]
}
```

## 4. Mountain valley

High walls and a livable valley floor. `ridged` sharpens crests; without it
you get round hills. Almost all land, no sea.

```json
{
  "name": "ValleAlto",
  "seed": 3312,
  "size_m": 8192,
  "cell_size": 2.0,
  "landform": "valley",
  "relief": "mountainous",
  "ridged": true,
  "land_fraction": 0.97,
  "erosion": 1.0,
  "river_carve": 6.0,
  "biome": "montane",
  "forest_cover": 0.45,
  "settlement_density": 0.7,
  "game_modes": ["GM"]
}
```

## 5. High mountain

The closest to "snowy" the base game allows: above the treeline it's rock and
mountain grass. **Not snow**, bare stone.
`alpine` goes up to ~1400 m, and note the `.terr` only encodes up to 1843 m.

```json
{
  "name": "PicoNegro",
  "seed": 8080,
  "size_m": 8192,
  "cell_size": 2.0,
  "landform": "inland",
  "relief": "alpine",
  "ridged": true,
  "land_fraction": 1.0,
  "erosion": 1.0,
  "biome": "montane",
  "forest_cover": 0.30,
  "rock_density": 2.0,
  "settlement_density": 0.4,
  "day_time": 11.0,
  "game_modes": ["GM"]
}
```

## 6. Arid plateau / desert

Dry, with tablelands and gullies. Few trees, lots of rock.

```json
{
  "name": "MesetaSeca",
  "seed": 2024,
  "size_m": 8192,
  "cell_size": 2.0,
  "landform": "plateau",
  "relief": "hilly",
  "land_fraction": 1.0,
  "erosion": 0.9,
  "biome": "arid",
  "forest_cover": 0.10,
  "rock_density": 1.4,
  "field_cover": 0.06,
  "settlement_density": 0.8,
  "day_time": 13.0,
  "game_modes": ["GM"]
}
```

## 7. Urban coast: one big city and its region

Sea on one side, flat ground so the urban grid fits, and high density.
Settlements follow a hierarchy (one city, several towns, many
hamlets), so **for "one big city" raise density and
lower relief**: no city fits on a slope.

```json
{
  "name": "PuertoReal",
  "seed": 5150,
  "size_m": 8192,
  "cell_size": 2.0,
  "landform": "coastal",
  "relief": "mild",
  "land_fraction": 0.72,
  "erosion": 0.7,
  "biome": "everon",
  "forest_cover": 0.16,
  "field_cover": 0.18,
  "settlement_density": 1.8,
  "game_modes": ["GM"]
}
```

`settlement_density` above 2 overlaps towns.

## 8. Farming interior: many towns and fields

No sea, flat, crop fields and scattered hamlets.

```json
{
  "name": "LlanoDelSur",
  "seed": 6120,
  "size_m": 8192,
  "cell_size": 2.0,
  "landform": "inland",
  "relief": "mild",
  "land_fraction": 1.0,
  "erosion": 0.75,
  "river_carve": 5.0,
  "biome": "everon",
  "forest_cover": 0.14,
  "field_cover": 0.30,
  "settlement_density": 1.6,
  "game_modes": ["GM"]
}
```

## 9. Closed forest

For ambushes and low visibility. Above 0.6 `forest_cover` there is
no room left for anything else.

```json
{
  "name": "SelvaNegra",
  "seed": 9001,
  "size_m": 8192,
  "cell_size": 2.0,
  "landform": "inland",
  "relief": "rolling",
  "land_fraction": 1.0,
  "erosion": 0.9,
  "biome": "montane",
  "forest_cover": 0.58,
  "tree_spacing": 7.0,
  "max_trees": 140000,
  "settlement_density": 0.5,
  "game_modes": ["GM"]
}
```

Raising `max_trees` matters: with the default cap (90,000) 58%
cover comes out thin with no explanation why.

## 10. Peninsula

Land reaching into the sea on one side. Long coast without becoming an island.

```json
{
  "name": "CaboLargo",
  "seed": 1848,
  "size_m": 8192,
  "cell_size": 2.0,
  "landform": "coastal",
  "relief": "rolling",
  "land_fraction": 0.55,
  "erosion": 0.9,
  "biome": "arland",
  "forest_cover": 0.22,
  "settlement_density": 1.1,
  "game_modes": ["GM"]
}
```

## 11. Test map

For checking a generator change without waiting seven minutes.

```json
{
  "name": "Prueba",
  "seed": 1,
  "size_m": 2048,
  "cell_size": 2.0,
  "landform": "island",
  "relief": "rolling",
  "land_fraction": 0.7,
  "biome": "everon",
  "forest_cover": 0.25,
  "settlement_density": 1.0,
  "game_modes": ["GM"]
}
```

## 12. Big, Everon-sized

12.8 km, which is what Everon measures. Takes about twenty minutes.

```json
{
  "name": "GranIsla",
  "seed": 1122,
  "size_m": 12800,
  "cell_size": 2.0,
  "landform": "island",
  "relief": "rolling",
  "land_fraction": 0.64,
  "erosion": 0.95,
  "biome": "everon",
  "forest_cover": 0.32,
  "settlement_density": 1.1,
  "max_trees": 200000,
  "max_rocks": 12000,
  "game_modes": ["GM"]
}
```

---

## 13. Texas: dissected plateau, ranch and oak

Continental tableland **with no sea**, cut by river cuts. Oak patches in the
bottoms, juniper and pine up high, and wide ranch country between settlements.
It uses the `texas` biome, which exists precisely for this: the `arid` palette
leaves six classification rules without material and everything falls to
`Dirt_02`, which is why an arid map comes out flat brown.

The keys that make the look:

- `land_fraction: 1.0` — no sea. With less, a coast shows up where Texas has
  no business having one.
- `landform: plateau` with `erosion: 0.95` — high erosion opens the cuts and
  leaves the scarp; with low erosion it's a characterless hill.
- `max_height: 240` instead of a `relief` — 240 m of relief is Hill Country.
  With `hilly` (320 m) it becomes sierra.
- `river_carve: 6.0` — the entrenched channel, which is the shape of the land
  there.
- High `field_cover: 0.24` and low `settlement_density: 0.85` — lots of open
  country and few settlements, far apart.

```json
{
  "name": "LoneStar",
  "title": "Lone Star",
  "seed": 2024,
  "size_m": 10240,
  "cell_size": 2.0,
  "landform": "plateau",
  "relief": "rolling",
  "max_height": 240.0,
  "land_fraction": 1.0,
  "erosion": 0.95,
  "river_carve": 6.0,
  "biome": "texas",
  "forest_cover": 0.16,
  "tree_spacing": 11.0,
  "rock_density": 1.3,
  "field_cover": 0.24,
  "settlement_density": 0.85,
  "day_time": 10.0,
  "game_modes": ["GM"]
}
```

**What will not match.** Reforger vegetation is entirely Central-European:
no oak, no mesquite, no juniper, no cactus. The generators used here
(hornbeam for background patches, pine for the heights, scrub for the rest)
give the right *silhouette* — low scattered trees over dry grass — but up
close it's hornbeam. A real Texas needs its own models, which is another job.

---

## Fields, one by one

| field | default | what it does |
|---|---|---|
| `name` | required | world name. No spaces |
| `title` | = `name` | how it shows in the menu |
| `seed` | 1 | fixes **everything**: relief, towns, every tree and every GUID |
| `size_m` | 8192 | map side in meters |
| `cell_size` | 2.0 | meters per cell. 2.0 is vanilla |
| `landform` | `island` | `island`, `coastal`, `inland`, `archipelago`, `valley`, `plateau` |
| `relief` | `rolling` | `flat` ~25 m, `mild` ~70, `rolling` ~160, `hilly` ~320, `mountainous` ~750, `alpine` ~1400 |
| `max_height` | from `relief` | max height in meters, if you want to force it |
| `land_fraction` | 0.62 | emerged fraction. Quantile-calibrated, comes out exact |
| `sea_depth` | 45.0 | sea depth |
| `erosion` | 0.8 | 0 = pure noise, 1 = full erosion. What makes valleys converge |
| `ridged` | false | sharp crests instead of round hills |
| `river_carve` | 3.5 | how deep rivers dig, in meters |
| `biome` | `everon` | `everon`, `arland`, `montane`, `arid` |
| `forest_cover` | from biome | fraction of land with forest |
| `tree_spacing` | 9.0 | meters between trees. Less = denser |
| `rock_density` | from biome | loose-rock multiplier |
| `field_cover` | 0.1 | crop-field fraction |
| `settlement_density` | 1.0 | above 2 towns overlap |
| `max_trees` | 90000 | hard cap. Raise it for lots of forest on a big map |
| `max_rocks` | 6000 | hard rock cap |
| `day_time` | 9.5 | start hour, decimal hours. Below 6 or above 19 opens at night |
| `game_modes` | `["GM"]` | `GM`, `Conflict`, `Plain` |
| `dependencies` | `[]` | extra mod GUIDs. Base game always included |

`--set` overrides any field without touching the file:

```powershell
py -3 -m worldforge build recipes/everon_like.json --set seed=42 --set relief=mountainous
```

---

## Combination recipes

Things often asked and how they come out.

| asked | touch |
|---|---|
| "more realistic" | raise `erosion` to 1.0 and `river_carve` to 5-6 |
| "more dramatic" | `ridged: true` and raise `relief` |
| "sea visible from everywhere" | `landform: island` and `land_fraction` 0.45-0.55 |
| "no sea" | `inland` or `plateau` with `land_fraction: 1.0` |
| "many towns" | `settlement_density: 1.5`. Above 2 they overlap |
| "one big city" | `relief: mild` **and** high density: no city fits on a slope |
| "nothing, wilderness" | `settlement_density: 0.2`, `forest_cover: 0.5` |
| "at night" | `day_time: 22.0`. Careful reviewing: you can't see a thing |
| "sunrise" | `day_time: 6.8` |
| "the coast doesn't convince me" | change only `seed` and go back to `preview` |

---

## Change the style, not the code

If the ask is "Mediterranean houses" or "pines instead of firs", **don't touch
the code**: edit the recipe catalog (see README).

- `biomes` — binds a surface palette to roles and parameters.
  Copying a biome and changing four things is the normal way to invent one.
- `roles` — `[prefab_path, weight]` lists. Weight 0 disables without deleting.
- `surfaces` — terrain materials, **in the order they enter the `.terr`**.
- `generators` — which `ForestGeneratorEntity` and which `RoadGeneratorEntity`
  to use.

Any path you add must exist in the prefab catalog, or generation
**aborts naming the missing prefab**. On purpose: an invented GUID gives no
error, the entity just never appears.
