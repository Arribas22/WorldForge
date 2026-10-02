# WorldForge Standard — generador de mapas para Arma Reforger

> Versión Standard: mapas procedurales **sin límites**. La versión Premium añade calcar sitios reales desde un mapa (ver abajo).

## ▶ Tu primer mapa en 3 minutos

[![Video: crear el mapa test](assets/images/preview-procedural.png)](https://streamable.com/jfb5mr)

### **[▶ Ver video: cómo crear el primer mapa test](https://streamable.com/jfb5mr)**

De la receta al addon, paso a paso.

## Qué es

Le das una receta JSON (o la haces clicando en la interfaz) y te escribe un addon que el Workbench abre: terreno cocido, hidrología, carreteras con perfil real, bosques, pueblos con calles, misiones y `.gproj`.

No copia el `.terr` de nadie: lo escribe entero desde cero con GUIDs propios que no chocan.

![Ejemplo procedural](assets/images/preview-procedural.png)
*Isla templada 8 km desde `recipes/everon_like.json`.*

## Standard vs Premium — con imágenes

### Standard: procedural + imagen tuya (incluido aquí)

- **5 caminos en Empezar:** Mapa aleatorio · Desde plantilla (13) · Procedural a medida · Desde una imagen tuya · (los 2 de sitio real son Premium y se enseñan con etiqueta).
- **13 plantillas en `recipes/`:** `everon_like`, `arland_like`, `archipelago`, `valle_montana`, `alta_montana`, `arid_plateau`, `costa_urbana`, `interior_agricola`, `bosque_cerrado`, `peninsula`, `isla_grande`, `montane_valley`, `texas_like`.
- **Modo imagen:** tu PNG/JPG define costa y arbolado; si añades relieve en grises, también la cota.
- **Todo el pipeline:** Vista previa, Construir, Cocinar (Workbench por CLI: shore map, ríos, generadores, `.topo`, navmesh triple, BSP), Preflight, Catálogo, Herramientas, Documentación.
- **Biomas:** `everon`, `arland`, `montane`, `arid`, `texas`. Semilla fija todo (mismo JSON = mismo mapa byte a byte).

Capturas reales del programa (tema oscuro, en castellano):

- `assets/images/standard-empezar.png` — Empezar (los 2 caminos Premium con etiqueta)
- `assets/images/standard-recetas.png` — Recetas (plantillas + detalle JSON)
- `assets/images/standard-editor.png` — Editor de receta (Verdania cargada)
- `assets/images/standard-preview.png` — Vista previa
- `assets/images/standard-build.png` — Construir
- `assets/images/standard-realista-bloqueado.png` — lo que ve Standard en Mapa realista (cartel Premium)

### Premium: mapa realista (elige la zona en el mapa)

![Premium mapa realista](assets/images/premium-mapa-realista.png)
*Pantalla Mapa realista capturada del programa: arrastra para mover, rueda para acercar, el cuadrado naranja es tu mapa. Derecha: Ir a, Tamaño, Límite y “Crear la receta y previsualizar”.*

- **Fuentes reales:** relieve Copernicus DEM, cobertura ESA WorldCover, carreteras + cauces + usos + huellas de edificios de OpenStreetMap (~10.000 casas en 8 km de ciudad, planta y giro reales). Satelital Sentinel-2 para el color.
- **Tamaños:** 2×2, 4×4, 8×8 (recomendado), 12×12, 16×16 km. A 8 km ya cabe ciudad + comarca; cocinar crece con el cuadrado del lado.
- **Borde:** Automático (interior=tierra, costa=agua), Rodear de agua, Tierra hasta el borde (se ve el corte — normal en Reforger).
- **En Standard** esta pantalla muestra el cartel “Calcar un sitio real es de Premium” y te manda a plantillas/procedural. No es una pared: te dice qué sí puedes hacer hoy.

| | Standard | Premium |
|---|---|---|
| Procedural (13 plantillas, biomas, relieve, pueblos) | ✅ | ✅ |
| Modo imagen (tu satelital → costa y arbolado) | ✅ | ✅ |
| Preview, build, cocinado, preflight, catálogo | ✅ | ✅ |
| Relieve Copernicus DEM | — | ✅ |
| Cobertura ESA WorldCover | — | ✅ |
| Carreteras, cauces y casas OSM | — | ✅ |
| Import Arma 3 `.wrp` | — | ✅ |

## Descarga

En **Releases**: carpeta `WorldForge-Standard` completa (`WorldForge.exe` 49 MB + `WorldForgeTools/` + `recipes/` + `locales/` + `docs/`). No descargues solo el `.exe`.

Requisitos: Windows 10/11 64-bit, Arma Reforger Tools.

## Video guía

Generación completa del mapa `test` (Verdania, 8 km): de la receta al addon listo.

[▶ Ver video: cómo crear el primer mapa test](https://streamable.com/jfb5mr)

## Uso

```powershell
WorldForge.exe                         # interfaz
WorldForge.exe --cli preview recipes/everon_like.json --out preview.png
WorldForge.exe --cli build recipes/everon_like.json --out C:/Desarrollo/MiMapa
WorldForge.exe --cli check C:/Desarrollo/MiMapa
WorldForge.exe --cli cook C:/Desarrollo/MiMapa
```

Orden: Construir → Cocinar → Preflight → abrir en Workbench:

```powershell
& "C:/Program Files (x86)/Steam/steamapps/common/Arma Reforger Tools/Workbench/ArmaReforgerWorkbenchSteamDiag.exe" `
  -gproj "C:/Desarrollo/MiMapa/addon.gproj" `
  -addonsDir "C:/Program Files (x86)/Steam/steamapps/common/Arma Reforger/addons,C:/Desarrollo" `
  -wbmodule=WorldEditor -run -load "worlds/MiMundo/MiMundo.ent"
```

Receta mínima:

```json
{
  "name": "Verdania", "seed": 1979,
  "size_m": 8192, "cell_size": 2.0,
  "landform": "island", "relief": "rolling",
  "biome": "everon", "forest_cover": 0.34,
  "settlement_density": 1.0, "game_modes": ["GM"]
}
```

`--set` sobrescribe sin tocar el fichero: `--set seed=42 --set relief=mountainous`.

## Documentación

- `docs/PLANTILLAS.md` — las 13 plantillas + tabla de campos.
- `docs/PIPELINE.md` — fases build/cook y por qué en ese orden.
- `recipes/` — copia/pega y cambia `name` + `seed`.

## Idioma

Castellano por defecto, inglés incluido. Auto desde Windows, en Ajustes se cambia (relanza).

## Premium

Standard + sitio real + `.wrp`. Pide `worldforge.lic` junto al `.exe`. Para pedirlo abre un Issue “Premium”.

## Licencia

Ver [LICENSE](../LICENSE). Uso gratuito para generar mapas. Sin fuentes del generador en este repo.
