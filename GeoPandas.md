# Guía básica de GeoPandas

**GeoPandas** es una biblioteca de Python para trabajar con **datos geográficos vectoriales**: puntos, líneas, polígonos y multipolígonos. Extiende `pandas`, por lo que permite filtrar, agrupar y combinar tablas, pero añade una columna especial llamada normalmente `geometry` y operaciones espaciales. ([geopandas.org](https://geopandas.org/en/stable/getting_started/introduction.html "Introduction to GeoPandas — GeoPandas 1.1.4+0.g91ec4af.dirty documentation"))

---

## 1. Instalación

La documentación recomienda usar un entorno independiente de Conda, especialmente porque GeoPandas depende de bibliotecas geoespaciales como GDAL, GEOS y PROJ. ([geopandas.org](https://geopandas.org/en/latest/getting_started/install.html?utm_source=chatgpt.com "Installation — GeoPandas 1.1.2.dev109+g2836d432f.d20260707 documentation"))

### Con Conda

```bash
conda create -n geo_env -c conda-forge python geopandas
conda activate geo_env
```

### Con pip

```bash
pip install geopandas
```

Para mapas estáticos e interactivos:

```bash
pip install matplotlib folium mapclassify
```

Importación habitual:

```python
import pandas as pd
import geopandas as gpd
import matplotlib.pyplot as plt
```

Puedes comprobar la versión instalada con:

```python
print(gpd.__version__)
```

---

## 2. Conceptos fundamentales

### `GeoSeries`

Es similar a una `pandas.Series`, pero contiene objetos geométricos:

```python
from shapely.geometry import Point

geometrias = gpd.GeoSeries([
    Point(2.17, 41.38),
    Point(-3.70, 40.42)
])

print(geometrias)
```

### `GeoDataFrame`

Es una tabla de Pandas que contiene una columna geométrica activa:

```python
datos = {
    "ciudad": ["Barcelona", "Madrid"],
    "poblacion": [1_700_000, 3_300_000]
}

gdf = gpd.GeoDataFrame(
    datos,
    geometry=[
        Point(2.17, 41.38),
        Point(-3.70, 40.42)
    ],
    crs="EPSG:4326"
)

print(gdf)
```

Conceptualmente:

```text
GeoDataFrame = DataFrame normal + columna geometry + CRS
```

GeoPandas utiliza geometrías de Shapely y aplica las operaciones espaciales sobre la columna geométrica activa. ([geopandas.org](https://geopandas.org/en/stable/getting_started/introduction.html "Introduction to GeoPandas — GeoPandas 1.1.4+0.g91ec4af.dirty documentation"))

---

## 3. Crear puntos desde longitud y latitud

Supongamos que tienes un CSV o un `DataFrame` con coordenadas:

```python
df = pd.DataFrame({
    "nombre": ["Barcelona", "Madrid", "Valencia"],
    "longitud": [2.17, -3.70, -0.38],
    "latitud": [41.38, 40.42, 39.47]
})
```

Puedes convertirlo en un `GeoDataFrame`:

```python
gdf = gpd.GeoDataFrame(
    df,
    geometry=gpd.points_from_xy(
        df["longitud"],
        df["latitud"]
    ),
    crs="EPSG:4326"
)
```

En `points_from_xy()`:

- `x` corresponde normalmente a la **longitud**.
    
- `y` corresponde normalmente a la **latitud**.
    

Este orden es importante: `longitud, latitud`, no al revés. ([geopandas.org](https://geopandas.org/en/stable/docs/reference/api/geopandas.points_from_xy.html "geopandas.points_from_xy — GeoPandas 1.1.4+0.g91ec4af.dirty documentation"))

---

## 4. Leer archivos geográficos

La función principal es:

```python
gdf = gpd.read_file("archivo.geojson")
```

También permite leer formatos como:

```python
gdf = gpd.read_file("municipios.shp")
gdf = gpd.read_file("datos.gpkg")
```

Para seleccionar una capa concreta de un GeoPackage:

```python
gdf = gpd.read_file(
    "datos.gpkg",
    layer="municipios"
)
```

GeoPandas utiliza `read_file()` para cargar numerosos formatos vectoriales mediante Pyogrio o Fiona y devuelve un `GeoDataFrame`. ([geopandas.org](https://geopandas.org/en/stable/docs/user_guide/io.html "Reading and writing files — GeoPandas 1.1.4+0.g91ec4af.dirty documentation"))

### Inspección inicial

Después de cargar un archivo, conviene revisar:

```python
print(gdf.head())
print(gdf.columns)
print(gdf.shape)
print(gdf.crs)
print(gdf.geom_type.value_counts())
print(gdf.total_bounds)
```

`total_bounds` devuelve:

```text
[min_x, min_y, max_x, max_y]
```

---

## 5. Seleccionar y filtrar datos

Como un `GeoDataFrame` hereda de `pandas.DataFrame`, puedes usar las operaciones habituales de Pandas.

### Seleccionar columnas

```python
resultado = gdf[["nombre", "poblacion", "geometry"]]
```

Conviene conservar `geometry` cuando quieras que el resultado siga siendo espacial.

### Filtrar filas

```python
grandes = gdf[gdf["poblacion"] > 100_000]
```

### Filtrar por texto

```python
terrassa = gdf[gdf["nombre"] == "Terrassa"]
```

### Crear columnas

```python
gdf["densidad"] = gdf["poblacion"] / gdf["superficie_km2"]
```

### Ordenar

```python
gdf = gdf.sort_values(
    "poblacion",
    ascending=False
)
```

---

## 6. Sistemas de coordenadas: CRS

El **CRS**, o sistema de referencia de coordenadas, indica cómo deben interpretarse las coordenadas y dónde se sitúan sobre la Tierra. ([docs.geopandas.org](https://docs.geopandas.org/en/stable/docs/user_guide/projections.html?utm_source=chatgpt.com "Projections — GeoPandas 1.1.4+0.g91ec4af.dirty documentation"))

Puedes consultarlo con:

```python
print(gdf.crs)
```

### `EPSG:4326`

Es el sistema habitual de longitud y latitud:

```python
gdf = gdf.set_crs("EPSG:4326")
```

Sus coordenadas están expresadas en **grados**, no en metros.

### Diferencia entre `set_crs()` y `to_crs()`

Esta diferencia es esencial.

#### `set_crs()`

Declara qué CRS tienen las coordenadas actuales:

```python
gdf = gdf.set_crs("EPSG:4326")
```

No cambia los números de las coordenadas.

#### `to_crs()`

Transforma realmente las coordenadas:

```python
gdf_proyectado = gdf.to_crs("EPSG:3857")
```

GeoPandas requiere que el CRS original esté definido antes de reproyectar. ([geopandas.org](https://geopandas.org/en/stable/docs/reference/api/geopandas.GeoDataFrame.to_crs.html "geopandas.GeoDataFrame.to_crs — GeoPandas 1.1.4+0.g91ec4af.dirty documentation"))

### Elegir automáticamente una proyección UTM

Para cálculos locales puedes estimar una zona UTM apropiada:

```python
utm_crs = gdf.estimate_utm_crs()
gdf_utm = gdf.to_crs(utm_crs)

print(utm_crs)
```

Para calcular **distancias, áreas o buffers**, utiliza normalmente un CRS proyectado, expresado en metros u otra unidad lineal, y no `EPSG:4326`, que utiliza grados. ([geopandas.org](https://geopandas.org/en/latest/getting_started/introduction.html?utm_source=chatgpt.com "Introduction to GeoPandas — GeoPandas 1.1.2.dev109+g2836d432f.d20260707 documentation"))

---

## 7. Operaciones geométricas básicas

Supongamos que `gdf_utm` utiliza un CRS expresado en metros.

### Área

```python
gdf_utm["area_m2"] = gdf_utm.geometry.area
gdf_utm["area_km2"] = gdf_utm.geometry.area / 1_000_000
```

### Longitud o perímetro

```python
gdf_utm["perimetro_m"] = gdf_utm.geometry.length
```

### Centroide

```python
centros = gdf_utm.geometry.centroid
```

### Buffer

Un buffer crea una zona a una distancia determinada de cada geometría:

```python
zonas_500m = gdf_utm.geometry.buffer(500)
```

### Límite de los polígonos

```python
limites = gdf_utm.geometry.boundary
```

### Unir todas las geometrías

```python
geometria_total = gdf_utm.geometry.union_all()
```

GeoPandas expone operaciones geométricas como `area`, `centroid`, `boundary`, `buffer`, `intersection` y `difference` mediante las geometrías de Shapely. ([geopandas.org](https://geopandas.org/en/stable/docs/user_guide/geometric_manipulations.html?utm_source=chatgpt.com "Geometric manipulations — GeoPandas 1.1.4+0.g91ec4af.dirty documentation"))

---

## 8. Relaciones espaciales

Las relaciones espaciales devuelven normalmente valores booleanos.

```python
punto = Point(430000, 4590000)

gdf_utm["contiene_punto"] = gdf_utm.geometry.contains(punto)
```

Operaciones frecuentes:

```python
gdf.geometry.intersects(otra_geometria)
gdf.geometry.contains(otra_geometria)
gdf.geometry.within(otra_geometria)
gdf.geometry.touches(otra_geometria)
gdf.geometry.overlaps(otra_geometria)
```

Interpretación:

- `contains`: la geometría contiene a la otra.
    
- `within`: la geometría está dentro de la otra.
    
- `intersects`: ambas tienen algún punto en común.
    
- `touches`: comparten un límite, pero no el interior.
    

---

## 9. Uniones espaciales

Una unión normal de Pandas relaciona filas mediante una columna común. Una **unión espacial** relaciona geometrías según su posición. ([geopandas.org](https://geopandas.org/en/stable/docs/user_guide/mergingdata.html "Merging data — GeoPandas 1.1.4+0.g91ec4af.dirty documentation"))

Por ejemplo, asignar a cada tienda el barrio en el que se encuentra:

```python
tiendas_con_barrio = tiendas.sjoin(
    barrios[["nombre_barrio", "geometry"]],
    how="left",
    predicate="within"
)
```

Los dos `GeoDataFrame` deben utilizar el mismo CRS:

```python
tiendas = tiendas.to_crs(barrios.crs)
```

Predicados habituales:

```python
predicate="within"
predicate="contains"
predicate="intersects"
predicate="touches"
```

También existe una unión con el elemento más cercano:

```python
resultado = tiendas.sjoin_nearest(
    estaciones,
    how="left",
    distance_col="distancia"
)
```

GeoPandas distingue entre `sjoin()`, basada en relaciones geométricas, y `sjoin_nearest()`, basada en proximidad. ([geopandas.org](https://geopandas.org/en/stable/docs/user_guide/mergingdata.html "Merging data — GeoPandas 1.1.4+0.g91ec4af.dirty documentation"))

---

## 10. Recortar e intersectar capas

### `clip()`

Recorta una capa utilizando otra como máscara:

```python
carreteras_municipio = gpd.clip(
    carreteras,
    municipio
)
```

Ambas capas deben tener el mismo CRS. ([geopandas.org](https://geopandas.org/en/stable/docs/reference/api/geopandas.clip.html?utm_source=chatgpt.com "geopandas.clip"))

### `overlay()`

Crea nuevas geometrías combinando dos capas:

```python
interseccion = gpd.overlay(
    municipios,
    zonas_protegidas,
    how="intersection"
)
```

Valores posibles de `how`:

```python
"intersection"
"union"
"difference"
"identity"
"symmetric_difference"
```

A diferencia de una unión espacial, `overlay()` puede cortar y generar nuevas geometrías. ([geopandas.org](https://geopandas.org/en/stable/docs/reference/api/geopandas.overlay.html?utm_source=chatgpt.com "geopandas.overlay - GeoDataFrame"))

---

## 11. Dibujar mapas

### Mapa sencillo

```python
gdf.plot()
plt.show()
```

### Mapa coloreado según una columna

```python
gdf.plot(
    column="poblacion",
    legend=True,
    edgecolor="black"
)

plt.show()
```

### Varias capas

```python
fig, ax = plt.subplots(figsize=(10, 8))

barrios.plot(
    ax=ax,
    color="lightgray",
    edgecolor="black"
)

tiendas.plot(
    ax=ax,
    markersize=30
)

plt.show()
```

El método `plot()` proporciona una interfaz de alto nivel sobre Matplotlib y permite crear mapas coropléticos mediante el argumento `column`. ([geopandas.org](https://geopandas.org/en/stable/docs/user_guide/mapping.html "Mapping and plotting tools — GeoPandas 1.1.4+0.g91ec4af.dirty documentation"))

### Mapa interactivo

```python
mapa = gdf.explore(
    column="poblacion",
    legend=True,
    tooltip=["nombre", "poblacion"]
)

mapa
```

Para guardarlo:

```python
mapa.save("mapa.html")
```

`explore()` crea mapas interactivos mediante Folium y Leaflet. ([docs.geopandas.org](https://docs.geopandas.org/en/latest/docs/user_guide/interactive_mapping.html?utm_source=chatgpt.com "Interactive mapping — GeoPandas 1.1.2.dev109+g2836d432f.d20260707 documentation"))

---

## 12. Guardar resultados

### GeoJSON

```python
gdf.to_file(
    "resultado.geojson",
    driver="GeoJSON"
)
```

### GeoPackage

```python
gdf.to_file(
    "resultado.gpkg",
    layer="municipios",
    driver="GPKG"
)
```

### Shapefile

```python
gdf.to_file("resultado.shp")
```

### GeoParquet

```python
gdf.to_parquet("resultado.parquet")
```

---

# Ejemplo completo

Supongamos que tienes:

- `barrios.geojson`, con los polígonos de los barrios.
    
- `tiendas.csv`, con las columnas `nombre`, `longitud` y `latitud`.
    

```python
import pandas as pd
import geopandas as gpd
import matplotlib.pyplot as plt

# 1. Leer los barrios
barrios = gpd.read_file("barrios.geojson")

# 2. Leer las tiendas
df_tiendas = pd.read_csv("tiendas.csv")

# 3. Convertir las coordenadas en puntos
tiendas = gpd.GeoDataFrame(
    df_tiendas,
    geometry=gpd.points_from_xy(
        df_tiendas["longitud"],
        df_tiendas["latitud"]
    ),
    crs="EPSG:4326"
)

# 4. Igualar los CRS
tiendas = tiendas.to_crs(barrios.crs)

# 5. Asignar un barrio a cada tienda
tiendas_barrios = tiendas.sjoin(
    barrios[["nombre_barrio", "geometry"]],
    how="left",
    predicate="within"
)

# 6. Contar tiendas por barrio
conteo = (
    tiendas_barrios
    .groupby("nombre_barrio")
    .size()
    .rename("numero_tiendas")
)

# 7. Incorporar el resultado a los barrios
barrios = barrios.merge(
    conteo,
    on="nombre_barrio",
    how="left"
)

barrios["numero_tiendas"] = (
    barrios["numero_tiendas"]
    .fillna(0)
    .astype(int)
)

# 8. Dibujar el mapa
barrios.plot(
    column="numero_tiendas",
    legend=True,
    edgecolor="black",
    figsize=(10, 8)
)

plt.title("Número de tiendas por barrio")
plt.axis("off")
plt.show()

# 9. Guardar el resultado
barrios.to_file(
    "barrios_con_tiendas.gpkg",
    layer="barrios",
    driver="GPKG"
)
```

---

## Errores frecuentes

1. **Intercambiar latitud y longitud.**  
    `points_from_xy()` recibe normalmente longitud como `x` y latitud como `y`.
    
2. **Combinar capas con CRS diferentes.**
    

```python
capa2 = capa2.to_crs(capa1.crs)
```

3. **Calcular distancias en `EPSG:4326`.**  
    Los resultados estarían expresados en grados, no en metros.
    
4. **Usar `set_crs()` para reproyectar.**  
    `set_crs()` solamente etiqueta las coordenadas; `to_crs()` las transforma.
    
5. **Perder el tipo `GeoDataFrame`.**  
    Si seleccionas columnas y eliminas `geometry`, el resultado deja de tener información espacial útil.
    
6. **Confundir `sjoin()` y `overlay()`.**  
    `sjoin()` añade atributos según una relación espacial; `overlay()` crea geometrías nuevas mediante operaciones como intersección o diferencia.
    

## Chuleta rápida

```python
gpd.read_file(...)                 # Leer una capa
gdf.to_file(...)                   # Guardar una capa
gdf.plot(...)                      # Mapa estático
gdf.explore(...)                   # Mapa interactivo
gdf.crs                            # Consultar CRS
gdf.set_crs(...)                   # Declarar CRS
gdf.to_crs(...)                    # Reproyectar
gdf.geometry.area                  # Área
gdf.geometry.length                # Longitud
gdf.geometry.centroid              # Centroide
gdf.geometry.buffer(100)           # Buffer
gdf.sjoin(...)                     # Unión espacial
gdf.sjoin_nearest(...)             # Vecino más cercano
gpd.clip(...)                      # Recortar
gpd.overlay(...)                   # Intersección/diferencia
gpd.points_from_xy(x, y)           # Crear puntos
```