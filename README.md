![WorldForge](assets/images/hero-banner.png)

[![Windows](https://img.shields.io/badge/Windows-10%2F11-blue)](https://github.com/) [![Arma Reforger](https://img.shields.io/badge/Arma-Reforger-orange)](https://reforger.armaplatform.com/) [![ES](https://img.shields.io/badge/lang-ES-green)](README.es.md) [![EN](https://img.shields.io/badge/lang-EN-green)](README.en.md) [![License](https://img.shields.io/badge/license-MIT-lightgrey)](LICENSE)

# WorldForge

> **Genera mundos completos y jugables para Arma Reforger. Procedural ilimitado, gratis.**
> **Generate complete, playable Arma Reforger worlds. Unlimited procedural, free.**

Docs completas: **[Español](README.es.md)** · **[English](README.en.md)**

De una receta JSON a un addon que abre Workbench: terreno cocido, carreteras con perfil real, bosques, pueblos con calles, misiones y `.gproj`. Sin copiar `.terr` de nadie: lo escribe entero con GUIDs propios.

From a JSON recipe to a Workbench-ready addon: baked terrain, properly-profiled roads, forests, towns with streets, missions and `.gproj`. No copied `.terr`: fully written with its own GUIDs.

## ▶ Tu primer mapa en 3 minutos / Your first map in 3 minutes

[![Video: crear el mapa test / first test map](assets/images/preview-procedural.png)](https://streamable.com/jfb5mr)

### **[▶ Ver video: cómo crear el primer mapa test](https://streamable.com/jfb5mr)**

De la receta al addon, paso a paso. / From recipe to addon, step by step.

**[⬇ Descargar en Releases](../../releases)** · **[🚀 Quickstart](#-3-pasos--3-steps)** · **[Todo incluido](#-todo-incluido--everything-included)** · **[Recetas](#-recetas--templates)**

---

## Por qué probarlo / Why try it

- **Mapa jugable en minutos:** aleatorio o plantilla → preview en 30-60 s → build → cook → jugar.
- **Playable map in minutes:** random or template → 30-60 s preview → build → cook → play.
- **No parece ruido:** erosión hidráulica, carreteras A\* con Chaikin, pueblos desde las calles, drenaje difuminado.
- **Doesn't look like noise:** hydraulic erosion, A\* roads with Chaikin, street-first towns, blurred drainage.
- **Repetible:** `seed` fija todo. Mismo JSON = mismo mapa byte a byte.
- **Reproducible:** `seed` fixes everything. Same JSON = same map byte-for-byte.

![Procedural](assets/images/preview-procedural.png)
*Isla templada 8 km desde `recipes/everon_like.json` / 8 km temperate island from `recipes/everon_like.json`.*

---

## ✅ Todo incluido / Everything included

**Gratis y completo. No hay versiones recortadas ni funciones bloqueadas.**
**Free and complete. No cut-down edition, nothing locked.**

| | |
|---|---|
| Procedural ilimitado + 13 plantillas / Unlimited procedural + 13 templates | ✅ |
| Modo imagen (tu satelital → costa y arbolado) / Image mode | ✅ |
| Preview, Build, Cook, Preflight, Catálogo / Catalog | ✅ |
| **Mapa realista: eliges la zona y se calca / Real-site: pick area, it gets traced** | ✅ |
| Relieve Copernicus DEM / Elevation | ✅ |
| Suelo ESA WorldCover / Ground cover | ✅ |
| Carreteras, cauces y casas OSM / OSM roads, rivers, houses | ✅ |
| Import Arma 3 `.wrp` | ✅ |

### Procedural — inventado desde la semilla

![Procedural](assets/images/standard-empezar.png)

- Empezar: aleatorio, plantilla, procedural a medida, desde tu imagen.
- Start: random, template, custom procedural, from your image.
- Flujo / Flow: Preview → Construir/Build → Cocinar/Cook → Preflight.
- Biomas / Biomes: `everon`, `arland`, `montane`, `arid`, `texas`.

![Editor](assets/images/standard-editor.png)
*Editor de receta con Verdania cargada: cada campo con ayuda, comprobación en vivo y resumen de lo que va a salir.*
*Recipe editor with Verdania loaded: every field with help, live validation and build summary.*

### Mapa realista — calca un sitio de verdad

![Mapa realista](assets/images/support-mapa-realista.png)

*Mapa realista: arrastra, zoom con rueda, cuadrado naranja = tu mapa 2×2–16×16 km → “Crear la receta y previsualizar”. Buscador + Everon, Fort Bragg, Alcalá de Henares, Normandía, Kyiv.*
*Realistic map: drag, wheel-zoom, orange square = your 2×2–16×16 km map → “Create recipe and preview”. Search + shortcuts.*

Calca / Traces: Copernicus DEM + ESA WorldCover + OSM (carreteras, cauces, usos, **~10.000 casas en 8 km con planta y giro reales**).

---

## 🔁 Portar un mapa de Arma 3 / Porting an Arma 3 map

Tres piezas independientes, usa las que quieras. / Three independent pieces, use whichever you want.

| En la receta / In the recipe | Qué trae / What it brings |
|---|---|
| `heightmap.wrp` | **El relieve** del `.wrp`. / **The relief** from the `.wrp`. |
| `objects_wrp` | **Viales y edificios** en sus posiciones reales (Jackson County: 2.203 piezas de calzada → 88 trazados, 855 edificios). / **Roads and buildings** in their real positions. |
| `places_file` | **Los pueblos** del `class Names` del `.hpp`: nombre, posición, radio (40 en Jackson County). / **The settlements** from the `.hpp`. |

Lo que no viene del original se genera igual: bosques, vallas, tendido eléctrico.
Whatever doesn't come from the original is still generated: forests, fences, power lines.

> ⚠️ **Escala vertical / Vertical scale.** Si el recuadro real mide 245 km y tu mapa 12,8, lo horizontal se encoge 19 veces y **sin corregir la altura todas las pendientes se multiplican por 19** — el mapa no se puede conducir y no da ningún error. En *Mapa realista* → **Calcular la escala recomendada**.
> If the real box is 245 km and your map 12.8, the horizontal shrinks 19× and **without correcting the height every slope is multiplied by 19** — undrivable, with no error. In *Realistic map* → **Work out the recommended scale**.

Más / More: atlas multi-región (España, México), trazados exactos para circuitos (`road_paths`, Nürburgring), enlaces de carretera y cruces de agua (`road_links`). Ver **[ES](README.es.md)** · **[EN](README.en.md)**.

---

## ▶ Video guía / Video guide

🇪🇸 Generación completa del mapa `test` (Verdania, 8 km): de la receta al addon listo para Workbench.
🇺🇸 Full `test` map generation (Verdania, 8 km): from recipe to Workbench-ready addon.

[▶ Ver video: cómo crear el primer mapa test](https://streamable.com/jfb5mr)

## 🚀 3 pasos / 3 steps

```powershell
WorldForge.exe   # interfaz / UI
```

```powershell
# 1. Mira antes (no escribe nada) / Look first
WorldForge.exe --cli preview recipes/everon_like.json --out preview.png
# 2. Genera / Build
WorldForge.exe --cli build recipes/everon_like.json --out C:/Desarrollo/MiMapa
# 3. Revisa + cocina / Check + cook
WorldForge.exe --cli check C:/Desarrollo/MiMapa
WorldForge.exe --cli cook C:/Desarrollo/MiMapa
```

Abrir en Workbench / Open in Workbench:

```powershell
& "C:/Program Files (x86)/Steam/steamapps/common/Arma Reforger Tools/Workbench/ArmaReforgerWorkbenchSteamDiag.exe" `
  -gproj "C:/Desarrollo/MiMapa/addon.gproj" `
  -addonsDir "C:/Program Files (x86)/Steam/steamapps/common/Arma Reforger/addons,C:/Desarrollo" `
  -wbmodule=WorldEditor -run -load "worlds/MiMundo/MiMundo.ent"
```

## 🗺 Recetas / Templates

`recipes/` — copia, cambia `name` + `seed`, genera. / copy, change `name` + `seed`, build.

| Receta | Ideal para / Best for |
|---|---|
| `everon_like` | Isla templada default / Default temperate island |
| `arland_like` | Isla atlántica pequeña / Small atlantic island |
| `archipelago` | Islas (sin puentes) / Islands (no bridges) |
| `valle_montana`, `montane_valley` | Valle encajado / Carved valley |
| `alta_montana` | Alta montaña (roca, no nieve) / High mountain |
| `arid_plateau`, `texas_like` | Desierto / meseta / Arid / plateau |
| `costa_urbana` | Ciudad + comarca / City + region |
| `interior_agricola` | Campos y caseríos / Fields and hamlets |
| `bosque_cerrado` | Emboscadas / Ambush forest |
| `peninsula`, `isla_grande` | Costa larga / Long coast |

Detalle de campos: [docs/PLANTILLAS.md](docs/PLANTILLAS.md) ([EN](docs/TEMPLATES.md)) · Pipeline: [docs/PIPELINE.md](docs/PIPELINE.md) ([EN](docs/PIPELINE_EN.md)).

## ⬇ Descarga / Download

En **[Releases](../../releases)**: carpeta completa `WorldForge` (`WorldForge.exe` + `WorldForgeTools/` + `recipes/` + `locales/` + `docs/`). El `.exe` suelto no basta.

From **[Releases](../../releases)**: full folder. Lone `.exe` is not enough.

Requisitos / Requirements: Windows 10/11 64-bit · Arma Reforger Tools + juego / game.

## ❓ FAQ

**¿Necesito Python? / Do I need Python?**
No. El `.exe` lleva dentro su propio Python, numpy, scipy y Tk. Descomprimir y doble clic. / No. The `.exe` carries its own Python, numpy, scipy and Tk. Unzip and double-click.

**No encuentra el Workbench / It can't find the Workbench**
Ajustes → *Ejecutable del Workbench* → **Detectar Steam**. **Si Steam está en otra unidad la ruta de fábrica no vale** y hay que buscarlo a mano; es la causa nº 1 de que no cocine. Y comprueba que las **Arma Reforger Tools** están instaladas: son una entrada **aparte** del juego en Steam. / Settings → *Workbench executable* → **Detect Steam**. **If Steam is on another drive the factory path is wrong** — browse to it; this is the #1 reason cooking fails. And check the **Arma Reforger Tools** are installed: they are a **separate** Steam entry from the game.

**Sale `Game addon '58D0FB3206B6F859' not found`**
La carpeta de addons del juego, en Ajustes, está mal o vacía. / The game addons folder in Settings is wrong or empty.

**El mapa sale sin tendidos, muros ni aceras / No power lines, walls or pavements**
Estás ejecutando el `.exe` fuera de su carpeta y no encuentra `WorldForgeTools/`. Pasa `WorldForge.exe --diag`. / You are running the `.exe` outside its folder, so it can't find `WorldForgeTools/`. Run `WorldForge.exe --diag`.

**¿Nieve? / Snow?**
Reforger 1.8 no trae superficie de nieve. Alta montaña = roca. / No snow surface in 1.8. High mountain = rock.

**¿Puentes? / Bridges?**
No. El agua es infranqueable para el trazado (a propósito). / No. Water is untraversable by design.

**¿Idioma? / Language?**
ES por defecto + EN. Auto desde Windows, en Ajustes (relanza). Puedes añadir el tuyo copiando `locales/en.json`. / ES default + EN. Auto from Windows, in Settings (restarts). Add your own by copying `locales/en.json`.

## ☕ Apoyar el proyecto / Support the project

WorldForge es **gratis y completo**: todo lo de arriba funciona y no se guarda nada.
WorldForge is **free and complete**: everything above works, nothing is held back.

Si te ahorra tiempo y quieres que siga adelante, **las versiones nuevas y mejoradas van antes a quien lo apoya**:
If it saves you time and you want to keep it moving, **new and improved versions go to supporters first**:

- ☕ **Ko-fi:** <https://ko-fi.com/arribas>
- 💬 **Discord:** `alv4r0` — escríbeme después de donar y te paso las versiones nuevas. / message me after donating and I'll send you the new builds.

Sin claves, sin activación y sin dar la lata. / No keys, no activation, no nagging.

## 📁 Repo

```
recipes/  docs/  assets/images/
```

Binarios por Release, nunca por commit. / Binaries via Releases, never committed.

## 📄 Licencia / License

[MIT](LICENSE) — docs, recetas y capturas libres para generar mapas (incluso comerciales). Mapas generados = tuyos (aplica EULA de Reforger).
Docs, recipes and shots free to generate maps (even commercial). Generated maps = yours (Reforger EULA applies).
