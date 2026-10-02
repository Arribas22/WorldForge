# The pipeline, phase by phase

A recipe goes in and a folder Workbench can open comes out. In between there
are nine phases, and **the order is not negotiable**: each one consumes masks
produced by the previous one.

> Versión en español: [PIPELINE.md](PIPELINE.md).

```
receta.json
    |
    v
MapSpec ......................... validation + defaults
    |
    v
1. relief ...................... terrain heightfield
2. hydrology ................... terrain hydrology
    |
    v
3. suitability + settlements .... layout
4. road network ................. layout
5. urban fabric ................. settlements
6. vegetation ................... vegetation
    |
    v
7. ground classification ........ surfaces   -> satmap.png
8. emission ..................... emit       -> .layer / .ent / .terr
9. preflight .................... validate/preflight
```

---

## 1. Relief

`landform` decides the general shape (island, coast, interior, valley,
plateau, archipelago) through a continental mask; detail comes from **FBM with
domain warp**; and believability comes from grid **hydraulic erosion**,
which is what turns noise crests into basins with converging valleys.

Two decisions that show a lot:

- Heights come out in **absolute meters with sea level at 0 m**, because
  the engine ocean sits at Y=0. Any other convention forces compensation
  in every later phase.
- `land_fraction` is calibrated **by quantile** over the height histogram, not
  by fixed threshold. That's why the emerged fraction comes out exact no
  matter what noise showed up.

## 2. Hydrology

- **Depression filling** (Planchon-Darboux by relaxation). Without it flow
  gets trapped in every noise hole: isolated ponds instead of a river
  network.
- **D8 flow accumulation** in descending height order. From it come both the
  channels (high accumulation) and the **wetness map**, which later decides
  where forest grows.

Done at reduced `work_res` and upscaled at the end. At full resolution this
would take minutes and the result would be no more believable.

Drainage is **blurred before thresholding**: D8 only pours in eight
directions, and without that smoothing rivers come out like a grid drawn
over the terrain.

## 3. Suitability and settlements

A scalar field says how good each point is for a town: flat, not too
high, near water but not in it. On it, biased Poisson sampling with
a hierarchy — one city, several towns, many hamlets — and minimum distance
per tier so they don't overlap.

## 4. Road network

Minimum expansion tree between settlements, then **A\* over terrain cost**
to trace each stretch. Cost penalizes slope squared:
that's what makes roads go around the mountain instead of climbing it.

Three things that cost blood:

- **Water is impassable, not expensive.** With finite cost A\* crosses it as
  soon as two settlements fall on different islands, and since WorldForge
  builds no bridges the result is a road floating over the sea. With `inf`
  the cell never enters the queue.
- **Settlements group by landmass** (4-neighborhood flood) and one tree is
  traced per group. Otherwise each island would leave one complete, failed
  A\* search per pair.
- **A coastal settlement can fall on a water cell** at working resolution
  (10-40 m depending on the map). It is dragged to the nearest land cell
  before tracing, or it gets no road at all.

Afterwards: Douglas-Peucker to drop the per-cell point, **Chaikin** to
round (8-neighbor A\* only turns in 45-degree steps, and without this roads
look like what they are, a grid path), and a final pass pulling back into
the water any vertex rounding pushed off the coast.

## 5. Urban fabric

A generator's typical mistake is scattering houses with noise inside a
circle. That yields a camp, not a town: buildings face anywhere
and there are no streets. Here it's backwards — **streets first**:

1. A main street crosses the settlement with its orientation.
2. Side streets branch off at irregular intervals.
3. In cities a ring is added, which is what gives an old-town silhouette.
4. Each street is walked in "plot width + gap" steps, and at each step
   a building is planted on each side, set back and **rotated 90 degrees to
   the street**: that's what makes facades face the road.

Everything vetoed against slope, water and overlap with what's placed.

## 6. Vegetation

Each patch is emitted the way the World Editor stores it:

```
SplineShapeEntity          <- the patch outline
  ForestGeneratorEntity    <- the biome's vanilla generator
    $grp Tree : prefab     <- trees ALREADY baked
```

**Baking the trees** is what makes the world work the moment you open it,
without pressing *Generate*. Keeping the spline and generator on top is what
lets you regenerate the patch by hand afterwards.

The outline is traced with **Moore** (edge following), not convex hull:
a convex forest swallows the roads bordering it as soon as
someone presses regenerate.

## 7. Ground classification

From height, slope, wetness and the road and old-town masks comes
an index map inside the biome palette, and from it the satellite image.

Rules run **in cascade from weakest to strongest**: what paints later
wins. Asphalt and concrete go last, because a road overrules
whatever biome was underneath.

## 8. Emission

Every previous phase only decided geometry; here it becomes files. The
concrete shapes (which child hangs from whom, which property goes at which
level) are **traced from a map that works in game**, not deduced from
documentation.

All text goes through a writer guaranteeing UTF-8 without BOM, `\n`
line breaks and zero `//`. All GUIDs come from a single deterministic
allocator.

## 9. Preflight

Eleven rules, and **each maps to a silent engine failure**: something that
errors nowhere, appears in no log, and only shows when you open the world
and half the island is missing.

1. BOM at the start of a text file
2. Invalid UTF-8
3. `//` comments in a resource file
4. `.meta` without a `Name "{GUID}path"` line
5. Duplicated GUID (this is what hangs Workbench at startup)
6. Resource without `.meta`, and references whose GUID doesn't match the catalog
7. `.terr` coherence and `.meta` `TilesCount` vs the grid
8. `HeightMap.desc` points at a file that exists
9. Empty layers
10. Road points below sea level
11. The `default.layer` `SCR_MapEntity`: no `Map Geometry Data` (**warning**,
    `cook` sets it when creating the `.topo`), and a dangling
    reference (**error**, that breaks the whole map screen)

`build` passes it alone when done. If an `ERROR` shows, the map is **not**
ready, no matter that the files exist.

Rule 11 splits into warning and error on purpose because they're different
failures with different fixes: "points to no `.topo`" on a freshly generated
addon is normal until you run `cook`; "points to a `.topo` that isn't there"
is a broken link to fix now.

---

## The player map: why `cook` sets the `.topo` reference

`SCR_MapEntity` needs two things for the M map to have content:

```
SCR_MapEntity MapEntity1 : "{731564B66F91B107}Prefabs/World/Game/MapEntity.et" {
 coords 2048 5 2048
 "Map Geometry Data" "worlds/W/W.topo"
 "Satellite background image" "{F4F70E92...}UI/Textures/Map/worlds/W/WSatellite.edds"
}
```

Each is set by someone different, hence the split:

| property | set by | why |
|---|---|---|
| `Satellite background image` | `build` | the satellite is computed there, with its `.png`, `.edds` and `.meta` |
| `Map Geometry Data` | `cook` | the `.topo` is created by Workbench, in the `2DMap` step, which runs **after** `build` |

**The satellite `.meta` is not optional.** An `.edds` without `.meta` has no
GUID, and without GUID `SCR_MapEntity` can't link it: the file shows in
Resource Manager and can't be referenced. Vanilla ships them
(`UI/Textures/Map/worlds/Arland/ArlandRasterized.edds`,
GUID `F98F3D2CBA523091`).

**The `.topo` is referenced by bare path**, no `{GUID}`: it has no `.meta`
and on a hand-made map its `.rdb` entry shows zero GUID. A wrong GUID
there errors nowhere, it just doesn't load.

History: `build` rewrites `default.layer` whole on each pass, so before only
a reference dragged from the previous generation could be used. On the first
there was none, and none would ever appear: the map opened forever without 2D
geometry, with the engine warning

```
RESOURCES (W): No 2d topographic geometry provided to the map.
                Roads, buildings, power lines, etc. won't be visible.
```

`cook` now patches `default.layer` as soon as it has the `.topo`, with
Workbench already closed and touching only those two lines — regenerating the
layer would throw away the `BSP` the last step writes.

---

## Navmesh: three meshes and the GUID, in this order

One step, the native generator on autogenerate, but it produces
**three** meshes because `SCR_AIWorld` carries the three vanilla
`NavmeshWorldComponent`:

| Component | GUID | Mesh | For |
|------------|------|-------|-----------|
| `NavmeshWorldComponent` | `{5584F30E67F617AD}` | `<World>_Navmesh_Soldiers.nmn` | infantry |
| `NavmeshWorldComponent` | `{5584F30EEFEE1223}` | `<World>_Navmesh_BTR.nmn` | vehicles |
| `NavmeshWorldComponent` | `{5C8C9B750D124A63}` | `<World>_Navmesh_Lowres.nmn` | distant AI |

`build` puts the three `NavmeshFile` in the layer, so the generator knows
where to write: without that path it invents an `autogenerated Navmesh.nmn`
at the addon root, with no `.meta` and nothing pointing at it.

The `.nmn.meta` order is the only thing to be careful with, and it is
**counter-intuitive**:

```
build    writes NavmeshFile into the layer      (generator needs the path)
         does NOT write .nmn.meta               <- truncated to 0 bytes here
cook     generator autogenerate                 (writes the three .nmn)
         verify navmesh                          (exist and non-empty)
         navmesh GUID                            <- writes the three .meta
```

Writing `.meta` before the `.nmn` exists buys nothing and actively
hurts: Workbench sees a `.meta` with no resource behind it and **truncates
it to zero bytes** on import. Measured on a build: all three `.meta` at
0 bytes **and the low-res mesh ungenerated**, with `cook` saying "ok" at
the end.

Each `.meta` GUID is read from `default.layer` itself — from the
`NavmeshFile "{FA1B...}worlds/Navmesh/X.nmn"` line — not from the
allocator. It's the only thing that matters: that line **is** the reference
the engine reads, so reading it from there matches resource and reference
by construction.

And navmesh verification checks **size**, not just existence.
The native generator exits 0 whether it made the mesh or not, and
creates the file even when it doesn't fill it, so `is_file()` approves a
0-byte resource.

---

## Road profiling

Order, and why that one:

1. **Densify** the polyline to 40 m, without moving any vertex. Before the
   20 m minimum-segment enforcement, not after: otherwise it undoes the work.
2. **Profile**: terrain height, 90 m **distance-window** smoothing, 80 m
   vertical curve, 9% grade cap and 4 m terrain-gap cap.
3. **Corridor**: opens the ground with a real road section — pavement,
   1:1.6 embankment and 1:1 cut — respecting the seabed cell by cell, so
   the shoreline embankment survives.

### The two limits, and why grade wins

Where terrain climbs faster than the allowed grade **both limits don't
fit**: either the profile lifts off the ground, or it exceeds the slope.
Measured on the test map:

| order | max grade | gap |
|-------|-----------|------------|
| cap then grade | **5.9 degrees** | 4.1 m |
| grade then cap | 24.7 degrees | 4.3 m |
| 50/50 split | 24.7 degrees | 6.9 m |

The first was chosen. The third was tried and **worsens both**: with
damping the profile sits halfway to both limits instead of meeting
either.

The gap cap measures against **debumped** terrain, with a **median**
filter, not moving average: the average skews at the ends (3.6 m with
a 90 m window at 16%) and a truncated-window median too, because it
falls at the window center, not the current sample. On the first and
last `k` points the reference is untouched terrain, where anchoring
lives and gap is zero by construction.

### Where two pavements overlap

The corridor averages both profiles, so **damage is shared** instead of
the last writer burying the earlier one. Two roads a meter apart with
5.8 m of profile difference is not a real situation: `check`
reports it separately with both layers and the length, because deciding
which one goes belongs to the map, not the tool.

Band counting must **count**, not flag, and PIL doesn't accumulate:
it draws and stamps. Two overlapping strokes give 255, not 2.

---

## What the pipeline doesn't do

Cooking terrain and player-map geometry are Workbench steps. WorldForge
writes the terrain **source** and stops there; `cook` does navmesh and
`.topo`; see the README.
