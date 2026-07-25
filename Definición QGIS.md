## ¿Qué es QGIS?

**QGIS** es un programa gratuito y de código abierto para trabajar con **información geográfica**. Es un **SIG** —Sistema de Información Geográfica— y permite crear, visualizar, editar, analizar y publicar datos asociados a posiciones sobre la Tierra. Es un proyecto oficial de la organización geoespacial OSGeo. ([QGIS](https://docs.qgis.org/latest/en/docs/user_manual/preamble/foreword.html?utm_source=chatgpt.com "Foreword — QGIS Documentation documentation"))

Podríamos verlo como una mezcla entre:

- un editor de mapas;
    
- una herramienta de análisis de datos;
    
- una base de datos geográfica;
    
- y un programa de diseño cartográfico.
    

## ¿Cómo funciona?

En QGIS, la información se organiza normalmente mediante **capas**. Cada capa representa un tipo de información diferente:

- **Puntos:** árboles, estaciones meteorológicas, accidentes o comercios.
    
- **Líneas:** carreteras, ríos, rutas o cables.
    
- **Polígonos:** municipios, parcelas, edificios o zonas protegidas.
    
- **Rásteres:** fotografías aéreas, imágenes de satélite, mapas de elevación o temperatura.
    

Por ejemplo, podrías tener simultáneamente:

```text
Capa de municipios
Capa de carreteras
Capa de ríos
Capa de estaciones meteorológicas
Imagen de satélite de fondo
```

QGIS permite superponerlas, cambiar su apariencia y analizar las relaciones entre ellas. Su documentación oficial incluye precisamente una sesión inicial basada en cargar y visualizar capas vectoriales y ráster. ([QGIS](https://docs.qgis.org/3.44/en/docs/user_manual/introduction/getting_started.html?utm_source=chatgpt.com "1. Getting Started — QGIS Documentation documentation"))

## ¿Qué se puede hacer con QGIS?

Entre otras cosas, permite:

- cargar archivos como **GeoJSON, Shapefile, GeoPackage, KML, CSV o GeoTIFF**;
    
- conectarse a bases de datos espaciales;
    
- incorporar mapas de OpenStreetMap y servicios cartográficos web;
    
- dibujar y modificar puntos, líneas y polígonos;
    
- calcular áreas, distancias y coordenadas;
    
- buscar qué elementos se encuentran dentro de una zona;
    
- crear áreas de influencia alrededor de carreteras o edificios;
    
- analizar elevaciones y modelos del terreno;
    
- diseñar mapas para imprimir o exportar como imagen o PDF;
    
- automatizar procesos mediante Python y **PyQGIS**.
    

La descripción oficial resume sus funciones como crear, editar, visualizar, analizar y publicar información geoespacial. ([qgis.org](https://www.qgis.org/project/overview/?utm_source=chatgpt.com "QGIS overview · QGIS"))

## Ejemplo práctico

Supón que tienes un archivo con estas columnas:

```csv
nombre,latitud,longitud,temperatura
Terrassa,41.5632,2.0089,31
Sabadell,41.5463,2.1086,30
Barcelona,41.3874,2.1686,29
```

En QGIS podrías:

1. Importar el CSV.
    
2. Convertir cada fila en un punto del mapa.
    
3. Colorear los puntos según la temperatura.
    
4. Añadir un mapa base.
    
5. Calcular distancias entre estaciones.
    
6. Crear un mapa final con título, leyenda y escala.
    

## QGIS frente a Google Earth

|QGIS|Google Earth|
|---|---|
|Pensado para análisis geográfico profesional|Pensado principalmente para explorar y presentar el planeta|
|Permite editar y procesar datos complejos|Trabaja especialmente bien con KML y KMZ|
|Ofrece muchas herramientas de análisis espacial|Tiene herramientas de análisis más limitadas|
|Permite controlar proyecciones y sistemas de coordenadas|Oculta gran parte de la complejidad cartográfica|
|Adecuado para proyectos científicos o técnicos|Adecuado para recorridos y visualizaciones rápidas|

**Google Earth es más sencillo para visualizar datos. QGIS es mucho más potente para prepararlos, modificarlos y analizarlos.**

Una combinación habitual sería:

```text
Datos originales
      ↓
Limpieza y análisis en GeoPandas
      ↓
Edición y diseño del mapa en QGIS
      ↓
Exportación a KML/KMZ
      ↓
Visualización en Google Earth
```

## Diferencia con GeoPandas

- **QGIS** es una aplicación gráfica: trabajas con ventanas, botones, paneles y mapas.
    
- **GeoPandas** es una biblioteca de Python: trabajas escribiendo código.
    
- **Google Earth** está más orientado a la visualización tridimensional y presentación.
    

Para empezar a trabajar con capas sin tener que programar, **QGIS suele ser la opción más sencilla y completa**. Está disponible para Windows, macOS y distribuciones GNU/Linux. ([QGIS API Documentation](https://api.qgis.org/api/?utm_source=chatgpt.com "QGIS API Documentation: QGIS"))