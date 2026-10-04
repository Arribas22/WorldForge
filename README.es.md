# WorldForge Standard — generador de mapas para Arma Reforger

> Versión Standard: mapas procedurales **sin límites**. La versión Support añade calcar sitios reales desde un mapa (ver abajo).

## ▶ Tu primer mapa en 3 minutos

[![Video: crear el mapa test](assets/images/preview-procedural.png)](https://streamable.com/jfb5mr)

### **[▶ Ver video: cómo crear el primer mapa test](https://streamable.com/jfb5mr)**

De la receta al addon, paso a paso.

## Qué es

Le das una receta JSON (o la haces clicando en la interfaz) y te escribe un addon que el Workbench abre: terreno cocido, hidrología, carreteras con perfil real, bosques, pueblos con calles, misiones y `.gproj`.

No copia el `.terr` de nadie: lo escribe entero desde cero con GUIDs propios que no chocan.

![Ejemplo procedural](assets/images/preview-procedural.png)
*Isla templada 8 km desde `recipes/everon_like.json`.*

## Todo incluido

**Gratis y completo.** No hay versión recortada ni funciones bloqueadas: todo
lo que hay aquí funciona.

| | |
|---|---|
| Procedural ilimitado + 13 plantillas | ✅ |
| Modo imagen (tu satelital → costa y arbolado) | ✅ |
| Preview, Construir, Cocinar, Preflight, Catálogo | ✅ |
| **Mapa realista: eliges la zona y se calca** | ✅ |
| Relieve Copernicus DEM | ✅ |
| Suelo ESA WorldCover | ✅ |
| Carreteras, cauces y casas de OSM | ✅ |
| Import Arma 3 `.wrp` | ✅ |

### Procedural

![Procedural](assets/images/standard-empezar.png)

Empezar: aleatorio, plantilla, procedural a medida, desde tu imagen. Biomas:
`everon`, `arland`, `montane`, `arid`, `texas`.

![Editor](assets/images/standard-editor.png)
*Editor de receta: cada campo con su ayuda, comprobación en vivo y resumen de
lo que va a salir.*

### Mapa realista

![Mapa realista](assets/images/support-mapa-realista.png)

Arrastra, zoom con la rueda, el cuadrado naranja es tu mapa (2×2 – 16×16 km) →
*Crear la receta y previsualizar*. Calca el relieve del Copernicus DEM, la
cobertura del suelo de ESA WorldCover y, de OpenStreetMap, las carreteras, los
cauces y las huellas de los edificios: **unas 10.000 casas en 8 km sobre una
ciudad, con su planta y su giro reales**.

## Descarga

En **Releases**: carpeta `WorldForge` completa (`WorldForge.exe` 82 MB + `WorldForgeTools/` + `recipes/` + `locales/` + `docs/`). No descargues solo el `.exe`.

Requisitos: Windows 10/11 64-bit, Arma Reforger Tools.

## Primer arranque

### No hace falta Python

El `.exe` lleva dentro su propio Python, numpy, scipy y Tk. Descomprimir y
doble clic: no hay nada que instalar.

Lo que **sí** hace falta:

| | Para qué | ¿Obligatorio? |
|---|---|---|
| **Arma Reforger** | Abrir el mapa que has hecho | Sí, para jugarlo |
| **Arma Reforger Tools** | Cocinar (navmesh, mapa del jugador, generadores) | Sí, para terminar un mapa |
| Python | — | **No** |

> **Las Tools son una entrada aparte en Steam**, no vienen con el juego.
> Búscalas como *Arma Reforger Tools*. Sin ellas se genera el addon entero y
> el terreno, pero no se puede cocinar, y un mapa sin cocinar abre sin
> navegación de IA y con el mapa del jugador vacío.

### Descomprime la carpeta entera

El `.exe` suelto no funciona: necesita `WorldForgeTools/` (el plugin que
hornea los generadores) y `recipes/` al lado. Déjala donde puedas escribir,
**no** en `C:\Program Files`.

### Ajustes: las cuatro rutas

Este es el paso que la gente se salta, y luego no cocina nada.

- **Proyecto WorldForge** — la carpeta de arriba. La encuentra sola.
- **Carpeta de addons generados** — donde se escriben tus mapas. Ponla al lado
  de la de WorldForge: así el `-addonsDir` del Workbench los ve sin tocar nada.
- **Ejecutable del Workbench** — pulsa **Detectar Steam**. Si no, la ruta suele
  ser `C:\Program Files (x86)\Steam\steamapps\common\Arma Reforger Tools\Workbench\ArmaReforgerWorkbenchSteamDiag.exe`.
  **Si Steam está en otra unidad la ruta de fábrica no vale**, y es con
  diferencia la causa nº 1 de que el cocinado falle.
- **Addons del juego base** — normalmente
  `C:\Program Files (x86)\Steam\steamapps\common\Arma Reforger\addons`.
  Sin ella el Workbench muere con `Game addon '58D0FB3206B6F859' not found`.

Pulsa **Guardar rutas**: los puntos de *Estado del entorno* se ponen en verde.
Las dos últimas valen también como variables de entorno `AR_WB_EXE` y
`AR_GAME_ADDONS`.

### Comprueba que está todo bien

**Empezar → Mapa aleatorio → Generar y construir**, y después **Cocinar**. El
Workbench se abre y se cierra solo, una vez por paso. Marca *Saltar el
navmesh* si solo quieres mirarlo, y *Abrir el mundo al terminar* para que el
World Editor se abra con él. Si eso funciona de punta a punta, está todo bien.

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

## Apoyar el proyecto

WorldForge es **gratis y completo**: todo lo de arriba funciona y no se guarda
nada.

Si te ahorra tiempo y quieres que siga adelante, **las versiones nuevas y
mejoradas van antes a quien lo apoya**:

- ☕ **Ko-fi:** <https://ko-fi.com/arribas>
- 💬 **Discord:** `alv4r0` — escríbeme después de donar y te paso las versiones
  nuevas.

Sin claves, sin activación y sin dar la lata.

## Licencia

Ver [LICENSE](../LICENSE). Uso gratuito para generar mapas. Sin fuentes del generador en este repo.
