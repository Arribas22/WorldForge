# El pipeline, fase a fase

> English version: [PIPELINE_EN.md](PIPELINE_EN.md).

Una receta entra por `worldforge/spec.py` y sale una carpeta que el Workbench
abre. En medio hay nueve fases, y **el orden no es negociable**: cada una
consume mascaras que produjo la anterior.

```
receta.json
   |
   v
 MapSpec ......................... spec.py          valida y rellena defaults
   |
   v
 1. relieve ...................... terrain/heightfield.py
 2. hidrologia ................... terrain/hydrology.py
   |
   v
 3. idoneidad + asentamientos .... world/layout.py
 4. red viaria ................... world/layout.py
 5. trama urbana ................. world/settlements.py
 6. vegetacion ................... world/vegetation.py
   |
   v
 7. clasificacion del suelo ...... terrain/surfaces.py   -> satmap.png
 8. emision ...................... emit/*.py             -> .layer / .ent / .terr
 9. preflight .................... validate/preflight.py
```

---

## 1. Relieve

`landform` decide la forma general (isla, costa, interior, valle, meseta,
archipielago) mediante una mascara continental; el detalle sale de **FBM con
domain warp**; y la credibilidad la pone la **erosion hidraulica** en rejilla,
que es lo que convierte crestas de ruido en cuencas con valles que convergen.

Dos decisiones que se notan mucho:

- Las alturas salen en **metros absolutos con el nivel del mar en 0 m**, porque
  el oceano del motor esta en Y=0. Cualquier otra convencion obliga a compensar
  en cada fase siguiente.
- `land_fraction` se calibra **por cuantil** sobre el histograma de alturas, no
  por umbral fijo. Por eso la fraccion emergida sale exacta sea cual sea el
  ruido que haya tocado.

## 2. Hidrologia

- **Relleno de depresiones** (Planchon-Darboux por relajacion). Sin esto el
  flujo se queda atrapado en cada agujero del ruido: salen charcos aislados en
  vez de una red fluvial.
- **Acumulacion de flujo D8** en orden de altura descendente. De ahi salen a la
  vez los cauces (acumulacion alta) y el **mapa de humedad**, que es lo que
  decide despues donde crece bosque.

Se hace a `work_res` reducida y se sube al final. A resolucion completa esto
tardaria minutos y el resultado no seria mas creible.

El drenaje se **difumina antes de umbralizar**: el D8 solo vierte en ocho
direcciones, y sin ese suavizado los rios salen como una rejilla dibujada
encima del terreno.

## 3. Idoneidad y asentamientos

Un campo escalar dice como de bueno es cada punto para un pueblo: llano, no muy
alto, cerca del agua pero no dentro. Sobre el, muestreo de Poisson sesgado con
una jerarquia — una ciudad, varias villas, muchos caserios — y distancia minima
por rango para que no se solapen.

## 4. Red viaria

Arbol de expansion minima entre asentamientos, y luego **A\* sobre coste de
terreno** para trazar cada tramo. El coste penaliza la pendiente al cuadrado:
eso es lo que hace que las carreteras rodeen la montana en vez de treparla.

Tres cosas que costaron sangre:

- **El agua es infranqueable, no cara.** Con un coste finito el A\* la cruza en
  cuanto dos nucleos caen en islas distintas, y como WorldForge no pone puentes
  el resultado es una carretera flotando sobre el mar. Con `inf` la celda no
  entra en la cola.
- **Los asentamientos se agrupan por masa de tierra** (inundacion por
  4-vecindad) y se traza un arbol por grupo. Si no, cada isla dejaria una
  busqueda A\* completa y fallida por cada pareja.
- **Un nucleo costero puede caer en una celda de agua** a la resolucion de
  trabajo (10-40 m segun el mapa). Se arrastra a la celda de tierra mas cercana
  antes de trazar, o se queda sin ninguna carretera.

Despues: Douglas-Peucker para quitar el punto por celda, **Chaikin** para
redondear (el A\* de 8 vecinos solo gira de 45 en 45 grados, y sin esto las
carreteras se ven como lo que son, un camino de rejilla), y un ultimo repaso que
saca del agua los vertices que el redondeo haya empujado fuera de la costa.

## 5. Trama urbana

El error tipico de un generador es esparcir casas con ruido dentro de un
circulo. Eso da un campamento, no un pueblo: los edificios miran a cualquier
lado y no hay calles. Aqui se hace al reves — **primero las calles**:

1. Una calle mayor cruza el nucleo con su orientacion.
2. De ella salen transversales a intervalos irregulares.
3. En las ciudades se anade un anillo, que es lo que da silueta de casco.
4. Cada calle se recorre a pasos de "ancho de parcela + hueco", y en cada paso
   se planta un edificio a cada lado, retranqueado y **girado 90 grados respecto
   a la calle**: eso es lo que hace que la fachada mire a la via.

Todo vetado contra pendiente, agua y solape con lo ya colocado.

## 6. Vegetacion

Cada mancha se emite como la guarda el World Editor:

```
SplineShapeEntity          <- el contorno de la mancha
  ForestGeneratorEntity    <- el generador vanilla del bioma
    $grp Tree : prefab     <- los arboles YA horneados
```

**Hornear los arboles** es lo que hace que el mundo funcione nada mas abrirlo,
sin pulsar *Generate*. Conservar el spline y el generador encima es lo que
permite regenerar la mancha a mano despues.

El contorno se traza con **Moore** (seguimiento de borde), no con envolvente
convexa: un bosque convexo se traga las carreteras que lo bordean en cuanto
alguien pulsa regenerar.

## 7. Clasificacion del suelo

De altura, pendiente, humedad y las mascaras de carretera y de casco urbano sale
un mapa de indices dentro de la paleta del bioma, y de ahi la satelital.

Las reglas van **en cascada de mas debil a mas fuerte**: lo que se pinta despues
gana. Asfalto y hormigon van al final, porque una carretera manda sobre
cualquier bioma que hubiera debajo.

## 8. Emision

Cada fase anterior solo decidio geometria; aqui se convierte en ficheros. Las
formas concretas (que hijo cuelga de quien, que propiedad va en que nivel) estan
**calcadas de un mapa que funciona en juego**, no deducidas de la documentacion.
Ver `docs/FORMATS.md`.

Todo el texto sale por `enfusion/io.py`, que garantiza UTF-8 sin BOM, saltos
`\n` y cero `//`. Todos los GUID salen de un unico asignador determinista.

## 9. Preflight

Once reglas, y **cada una corresponde a un fallo mudo del motor**: algo que no
da error, no aparece en el log, y solo se nota cuando abres el mundo y falta
media isla.

1. BOM al principio de un fichero de texto
2. UTF-8 invalido
3. Comentarios `//` en un fichero de recurso
4. `.meta` sin linea `Name "{GUID}ruta"`
5. GUID duplicado (esto es lo que cuelga el Workbench al arrancar)
6. Recurso sin `.meta`, y referencias con GUID que no casa con el catalogo
7. Coherencia del `.terr` y `TilesCount` del `.meta` contra la rejilla
8. El `HeightMap.desc` apunta a un fichero que existe
9. Capas vacias
10. Puntos de carretera bajo el nivel del mar
11. El `SCR_MapEntity` del `default.layer`: sin `Map Geometry Data` (**aviso**,
    lo pone `cook` al crear el `.topo`), y con una referencia colgante
    (**error**, eso rompe la pantalla del mapa entera)

`build` lo pasa solo al terminar. Si sale un `ERROR`, el mapa **no** esta listo,
por mucho que los ficheros existan.

La 11 se divide a proposito en aviso y error porque son fallos distintos con
arreglos distintos: "no apunta a ningun `.topo`" en un addon recien generado es
lo normal hasta que corres `cook`; "apunta a un `.topo` que no esta" es un
enlace roto y hay que repararlo ya.

---

## El mapa del jugador: por que la referencia al `.topo` la pone `cook`

El `SCR_MapEntity` necesita dos cosas para que el mapa de la M tenga contenido:

```
SCR_MapEntity MapEntity1 : "{731564B66F91B107}Prefabs/World/Game/MapEntity.et" {
 coords 2048 5 2048
 "Map Geometry Data" "worlds/W/W.topo"
 "Satellite background image" "{F4F70E92...}UI/Textures/Map/worlds/W/WSatellite.edds"
}
```

Las dos las pone alguien distinto, y por eso estan repartidas:

| propiedad | la pone | por que |
|---|---|---|
| `Satellite background image` | `build` | la satelital se calcula ahi, y sale con su `.png`, su `.edds` y su `.meta` |
| `Map Geometry Data` | `cook` | el `.topo` lo crea el Workbench, en el paso `2DMap`, que corre **despues** de `build` |

**El `.meta` de la satelital no es opcional.** Un `.edds` sin `.meta` no tiene
GUID, y sin GUID el `SCR_MapEntity` no lo puede enlazar: el fichero aparece en el
Resource Manager y no se puede referenciar. El vanilla los lleva
(`UI/Textures/Map/worlds/Arland/ArlandRasterized.edds`,
GUID `F98F3D2CBA523091`), y el `.meta` declara `PNGResourceClass` con el
`TextureColorMap.conf` de cada plataforma. El `.png` se escribe al lado porque
ese `.meta` dice que el origen de la importacion es el png.

**El `.topo` se referencia con la ruta pelada**, sin `{GUID}`: no lleva `.meta`
y en un mapa hecho a mano su entrada en el `.rdb` sale con GUID cero. Un GUID
equivocado ahi no da error, simplemente no carga.

El historial de esto: `build` reescribe el `default.layer` entero en cada
pasada, asi que antes solo se podia arrastrar una referencia de la generacion
anterior. En la primera no habia ninguna, y ninguna volveria a aparecer: el mapa
abria para siempre sin geometria 2D, con el motor avisando

```
RESOURCES (W): No 2d topographic geometry provided to the map.
               Roads, buildings, power lines, etc. won't be visible.
```

`cook` parchea ahora el `default.layer` en cuanto tiene el `.topo`, con el
Workbench ya cerrado y tocando solo esas dos lineas -- regenerar la capa tiraria
el `BSP` que escribe el ultimo paso.

---

## El navmesh: tres mallas y el GUID, en este orden

El paso es uno solo, `NavmeshGeneratorMain -run -autogenerate`, pero produce
**tres** mallas porque el `SCR_AIWorld` lleva los tres `NavmeshWorldComponent` de
vanilla:

| Componente | GUID | Malla | Para que |
|------------|------|-------|-----------|
| `NavmeshWorldComponent` | `{5584F30E67F617AD}` | `<Mundo>_Navmesh_Soldiers.nmn` | infanteria |
| `NavmeshWorldComponent` | `{5584F30EEFEE1223}` | `<Mundo>_Navmesh_BTR.nmn` | vehiculos |
| `NavmeshWorldComponent` | `{5C8C9B750D124A63}` | `<Mundo>_Navmesh_Lowres.nmn` | IA lejana |

`build` pone los tres `NavmeshFile` en la capa, para que el generador sepa donde
escribir: sin esa ruta se inventa un `autogenerated Navmesh.nmn` en la raiz del
addon, sin `.meta` y sin que nada lo apunte.

El orden del `.nmn.meta` es lo unico que hay que tener cuidado, y es
**contra-intuitivo**:

```
build    escribe los NavmeshFile en la capa      (el generador necesita la ruta)
         NO escribe el .nmn.meta                  <- aqui se trunca a 0 bytes
cook     NavmeshGeneratorMain -autogenerate      (escribe los tres .nmn)
         _verificar_navmesh                       (existen y no estan vacios)
         _guid_navmesh                            <- escribe los tres .meta
```

Escribir el `.meta` antes de que exista el `.nmn` no sirve de nada y actively
perjudica: el Workbench ve un `.meta` sin recurso detras y **lo trunca a cero
bytes** al importar. Medido en un build: los tres `.meta` a 0 bytes **y la malla
de baja resolucion sin generar**, con el `cook` diciendo "ok" al final.

El GUID de cada `.meta` se lee del propio `default.layer` -- de la linea
`NavmeshFile "{FA1B...}worlds/Navmesh/X.nmn"` -- y no del `GuidAllocator`. Es lo
unico que importa: esa linea **es** la referencia que lee el motor, asi que
leyendola de ahi el recurso y la referencia casan por construccion.

Y `_verificar_navmesh` comprueba el **tamanio**, no solo que el fichero exista.
El generador nativo sale con codigo 0 tanto si ha hecho la malla como si no, y
crea el fichero aunque no lo rellene, asi que `is_file()` da el visto bueno a un
recurso de 0 bytes.

---

## El perfil de calzada

`worldforge/world/perfil.py`. El orden, y por que es ese:

1. **Densificar** la polilinea a 40 m, sin mover ningun vertice. Antes de
   `enforce_min_segment` (20 m), no despues: si no, este se lleva lo que ha hecho.
2. **Perfil** (`alineacion`): cota del terreno, suavizado con ventana de 90 m
   **en distancia**, curva vertical de 80 m, recorte de rasante al 9 % y tope de
   separacion al terreno de 4 m.
3. **Corredor** (`flatten_roads`): abre el suelo con la seccion real de una
   carretera -- calzada, terraplen 1:1,6 y corte 1:1 -- y respeta el fondo marino
   celda a celda, para no perder el terraplen de la orilla.

### Los dos limites, y por que la rasante manda

Donde el terreno sube mas rapido que la rasante admitida **los dos limites no
caben**: o el perfil se despega del suelo, o se pasa de pendiente. Medido en el
mapa de prueba:

| orden | rasante maxima | separacion |
|-------|----------------|------------|
| tope y luego rasante | **5,9 grados** | 4,1 m |
| rasante y luego tope | 24,7 grados | 4,3 m |
| reparto al 50/50 | 24,7 grados | 6,9 m |

Se eligio la primera. La tercera se probo y **empeora las dos**: con
amortiguacion el perfil se queda a medio camino de los dos limites en vez de
cumplir ninguno.

El tope de separacion se mide contra el terreno **sin baches**, con un filtro de
**mediana** y no de media movil: la media se sesga en los extremos (3,6 m con
una ventana de 90 m al 16 %) y una mediana de ventana truncada tambien, porque
cae en el centro de la ventana y no en la muestra actual. En los primeros y
ultimos `k` puntos la referencia es el terreno sin tocar, que es donde va el
anclaje y la separacion es cero por construccion.

### Donde se solapan dos calzadas

El corredor promedia los dos perfiles, de modo que **el dano se reparte** en vez
de que la ultima en escribirse entierre a la anterior. Que dos viales esten a un
metro con 5,8 m de diferencia de perfil no es una situacion real: `check` la
avisa por separado con las dos capas y el largo, porque decidir cual sobra es
cosa del mapa y no de la herramienta.

El conteo de bandas tiene que **contar**, no poner un si-o-no, y PIL no acumula:
dibuja y pisa. Dos trazos superpuestos dan 255, no 2.

---

## Lo que el pipeline no hace

Cocinar el terreno (`.bterr`/`.ttile`/`.edds`) y la geometria del mapa del
jugador (`.topo`) son pasos del Workbench. WorldForge escribe la **fuente** del
terreno y se para ahi; el navmesh y el `.topo` los hace `cook`; ver el README.
