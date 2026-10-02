# Plantillas de mapa

> English version: [TEMPLATES.md](TEMPLATES.md).

Copia el bloque que mas se parezca a lo que quieres, pegalo en
`recipes/<nombre>.json`, cambia `name` y `seed`, y adelante:

```powershell
py -3 -m worldforge preview recipes/<nombre>.json --out preview.png   # mira antes
py -3 -m worldforge build   recipes/<nombre>.json --out C:/Desarrollo/MiMapa
py -3 -m worldforge cook    C:/Desarrollo/MiMapa                      # shore map, navmesh, BSP
```

**Mira el `preview` antes de generar.** Cuesta la mitad, no escribe nada, y
cambiar `seed` hasta que la costa convenza es mas rapido que pelearse con los
parametros.

---

## Lo primero: lo que NO se puede hacer

Para no perder el tiempo intentandolo.

- **No hay mapas nevados.** Arma Reforger 1.8 trae 55 superficies de terreno y
  **ninguna es nieve ni hielo**: hay hierba, hierba de montana, brezo, roca,
  arena, grava, tierra, cultivo, asfalto y hormigon. Lo mas cerca que se llega
  es alta montana pelada (roca y `MountainGrass`) por encima de la linea de
  arboles. Un mapa nevado de verdad necesita `.emat` propios, que es otro
  trabajo: crear las texturas, no generar el mapa.
- **No hay clima frio ni estaciones.** Los sufijos `_aut` de las superficies son
  la variante de otono, no de invierno.
- **Cada bloque de 64 m admite como mucho cinco materiales.** No es un limite
  nuestro: Everon y Jackson topan los dos en cinco exactos. El pintado ya es por
  celda (quadtree, ver `docs/FORMATS.md`), pero si una zona junta carretera,
  bosque, campo, roca y pueblo, el sexto material se descarta y se reparte entre
  los cinco mas extensos. En la practica se nota poco; donde se veria es en un
  cruce de carretera dentro de un pueblo dentro de un bosque.
- **No hay puentes.** El agua es infranqueable para el trazado: un nucleo en un
  islote se queda sin carretera, a proposito.

---

## Tamanos y tiempos

`size_m / cell_size` es el numero de celdas por lado, y es lo que decide si el
mapa tarda un minuto o media hora.

| para que | `size_m` | `cell_size` | celdas | tiempo |
|---|---|---|---|---|
| probar una idea | 2048 | 2.0 | 1025 | ~30 s |
| mapa pequeno, tipo Arland | 4096 | 2.0 | 2049 | ~1 min |
| el tamano normal | 8192 | 2.0 | 4097 | ~7 min |
| grande, tipo Everon | 12800 | 2.0 | 6401 | ~20 min |
| detalle fino | 4096 | 1.0 | 4097 | ~7 min |

Si te piden un mapa enorme con detalle fino, **sube `cell_size` antes que bajar
`size_m`**: 2 m por celda es lo que usan Everon y Arland y se ve bien.

---

## 1. Isla templada, estilo Everon

El caso por defecto: colinas suaves, bosque mixto, pueblos en los valles y en la
costa. Si no sabes que quieres, empieza por aqui.

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

## 2. Isla atlantica, estilo Arland

Mas pequena, mas pelada y mas rota: brezo, pino disperso y mucha costa.

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

## 3. Archipielago

Varias islas sin istmo. **Cada masa de tierra tiene su propia red viaria** y los
nucleos aislados se quedan incomunicados, porque no hay puentes. Si lo que
quieres es una isla grande con recortes, sube `land_fraction` a 0.6 y usa
`island`.

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

## 4. Valle de montana

Paredes altas y un fondo de valle habitable. `ridged` afila las crestas; sin el
salen lomas redondas. Casi todo tierra, sin mar.

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

## 5. Alta montana

Lo mas parecido a "nevado" que permite el juego base: por encima de la linea de
arboles queda roca y hierba de montana. **No es nieve**, es piedra desnuda.
`alpine` sube a ~1400 m, y ojo que el `.terr` solo codifica hasta 1843 m.

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

## 6. Meseta arida / desierto

Seca, con mesetas y barrancos. Poco arbol, mucha roca.

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

## 7. Costa urbana: una ciudad grande y su comarca

Mar a un lado, terreno llano para que quepa la trama urbana, y densidad alta.
Los nucleos se reparten por jerarquia (una ciudad, varias villas, muchos
caserios), asi que **para "una ciudad grande" lo que se sube es la densidad y se
baja el relieve**: en cuesta no cabe una ciudad.

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

`settlement_density` por encima de 2 solapa los pueblos.

## 8. Interior agricola: muchos pueblos y campos

Sin mar, llano, campos de cultivo y caserios repartidos.

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

## 9. Bosque cerrado

Para emboscadas y poca visibilidad. Por encima de 0.6 de `forest_cover` no queda
hueco para nada mas.

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

Subir `max_trees` importa: con el limite por defecto (90.000) una cobertura del
58% se queda corta y el bosque sale ralo sin decir por que.

## 10. Peninsula

Tierra que entra en el mar por un lado. Costa larga sin llegar a isla.

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

## 11. Mapa de pruebas

Para comprobar un cambio del generador sin esperar siete minutos.

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

## 12. Grande, tamano Everon

12,8 km, que es lo que mide Everon. Tarda unos veinte minutos.

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

## 13. Texas: meseta disecada, rancho y encinar

Meseta continental **sin mar**, cortada por canadas de rio. Manchas de encinar en
los fondos, sabina y pinar en lo alto, y rancho extenso entre nucleo y nucleo.
Usa el bioma `texas`, que existe justo para esto: la paleta `arid` deja seis
reglas de clasificacion sin material y todo cae a `Dirt_02`, que es por lo que un
mapa arido sale marron plano.

Las claves que hacen el aspecto:

- `land_fraction: 1.0` -- sin mar. Con menos aparece una costa que en Texas no
  pinta nada.
- `landform: plateau` con `erosion: 0.95` -- la erosion alta es la que abre las
  canadas y deja el escarpe; con erosion baja queda una loma sin caracter.
- `max_height: 240` en vez de un `relief` -- 240 m de desnivel es Hill Country.
  Con `hilly` (320 m) se va a sierra.
- `river_carve: 6.0` -- el cauce encajado, que es la forma del terreno alli.
- `field_cover: 0.24` alto y `settlement_density: 0.85` bajo -- mucho campo
  abierto y pocos nucleos, lejos unos de otros.

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

**Lo que no va a parecerse.** La vegetacion de Reforger es centroeuropea entera:
no hay encina, ni mezquite, ni sabina, ni nopal. Los generadores que se usan aqui
(carpe para las manchas de fondo, pino para lo alto, matorral para el resto) dan
la *silueta* correcta -- arbol bajo y disperso sobre pasto seco -- pero de cerca
es un carpe. Un Texas de verdad necesita modelos propios, que es otro trabajo.

---

## Los campos, uno a uno

| campo | por defecto | que hace |
|---|---|---|
| `name` | obligatorio | nombre del mundo. Sin espacios |
| `title` | = `name` | como sale en el menu |
| `seed` | 1 | fija **todo**: relieve, pueblos, cada arbol y cada GUID |
| `size_m` | 8192 | lado del mapa en metros |
| `cell_size` | 2.0 | metros por celda. 2.0 es lo vanilla |
| `landform` | `island` | `island`, `coastal`, `inland`, `archipelago`, `valley`, `plateau` |
| `relief` | `rolling` | `flat` ~25 m, `mild` ~70, `rolling` ~160, `hilly` ~320, `mountainous` ~750, `alpine` ~1400 |
| `max_height` | del `relief` | altura maxima en metros, si quieres forzarla |
| `land_fraction` | 0.62 | fraccion emergida. Se calibra por cuantil, sale exacta |
| `sea_depth` | 45.0 | profundidad del mar |
| `erosion` | 0.8 | 0 = ruido puro, 1 = erosion completa. Es lo que hace que los valles converjan |
| `ridged` | false | crestas afiladas en vez de lomas |
| `river_carve` | 3.5 | cuanto excavan los rios, en metros |
| `biome` | `everon` | `everon`, `arland`, `montane`, `arid` |
| `forest_cover` | del bioma | fraccion de tierra con bosque |
| `tree_spacing` | 9.0 | metros entre arboles. Menos = mas denso |
| `rock_density` | del bioma | multiplicador de rocas sueltas |
| `field_cover` | 0.1 | fraccion de campos de cultivo |
| `settlement_density` | 1.0 | por encima de 2 los pueblos se solapan |
| `max_trees` | 90000 | tope duro. Subelo si pides mucho bosque en mapa grande |
| `max_rocks` | 6000 | tope duro de rocas |
| `day_time` | 9.5 | hora al arrancar, en horas decimales. Por debajo de 6 o encima de 19 abre de noche |
| `game_modes` | `["GM"]` | `GM`, `Conflict`, `Plain` |
| `dependencies` | `[]` | GUID de mods adicionales. El juego base va siempre |

`--set` sobrescribe cualquier campo sin tocar el fichero:

```powershell
py -3 -m worldforge build recipes/everon_like.json --set seed=42 --set relief=mountainous
```

---

## Recetas de combinacion

Cosas que se piden a menudo y como salen.

| lo que te piden | que tocar |
|---|---|
| "mas realista" | sube `erosion` a 1.0 y `river_carve` a 5-6 |
| "mas dramatico" | `ridged: true` y sube `relief` |
| "que se vea el mar desde todas partes" | `landform: island` y `land_fraction` 0.45-0.55 |
| "sin mar" | `inland` o `plateau` con `land_fraction: 1.0` |
| "muchos pueblos" | `settlement_density: 1.5`. Por encima de 2 se solapan |
| "una ciudad grande" | `relief: mild` **y** densidad alta: en cuesta no cabe |
| "que no haya nada, salvaje" | `settlement_density: 0.2`, `forest_cover: 0.5` |
| "de noche" | `day_time: 22.0`. Ojo para revisarlo: no se ve nada |
| "amanecer" | `day_time: 6.8` |
| "la costa no me convence" | cambia solo `seed` y vuelve al `preview` |

---

## Cambiar el estilo, no el codigo

Si lo que piden es "casas mediterraneas" o "pinos en vez de abetos", **no toques
el codigo**: edita `worldforge/catalog/palettes.json`.

- `biomes` — ata una paleta de superficies a unos roles y unos parametros.
  Copiar un bioma y cambiarle cuatro cosas es la forma normal de inventar uno.
- `roles` — listas de `[ruta_de_prefab, peso]`. Peso 0 desactiva sin borrar.
- `surfaces` — materiales del terreno, **en el orden en que entran al `.terr`**.
- `generators` — que `ForestGeneratorEntity` y que `RoadGeneratorEntity` usar.

Cualquier ruta que anadas tiene que estar en `worldforge/catalog/prefabs.json`, o
la generacion **aborta con el nombre del prefab que falta**. Es a proposito: un
GUID inventado no da error, la entidad simplemente no aparece.

Para ver que hay:

```powershell
py -3 -m worldforge biomes
py -3 -c "import json; p=json.load(open('worldforge/catalog/prefabs.json'))['prefabs']; print([k for k in p if 'Houses/Village' in k])"
```
