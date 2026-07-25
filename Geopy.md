# Guía básica de GeoPy

**GeoPy** es una biblioteca de Python que proporciona una interfaz común para utilizar servicios de geocodificación como Nominatim/OpenStreetMap, Google Maps, Bing, ArcGIS y otros. También permite calcular distancias geográficas entre coordenadas. GeoPy **no contiene por sí misma una base de datos de direcciones**: envía las consultas al servicio que elijas. ([geopy.readthedocs.io](https://geopy.readthedocs.io/en/latest/ "Welcome to GeoPy’s documentation! — GeoPy 2.5.0 documentation"))

La versión actual es **GeoPy 2.5.0**, publicada el 12 de julio de 2026, y requiere Python 3.8 o posterior. ([PyPI](https://pypi.org/project/geopy/ "geopy · PyPI"))

## 1. Instalación

```bash
pip install geopy
```

En un entorno Conda también puedes usar:

```bash
conda install -c conda-forge geopy
```

Comprueba la instalación:

```python
import geopy

print(geopy.__version__)
```

La instalación oficial recomendada utiliza `pip install geopy`. ([geopy.readthedocs.io](https://geopy.readthedocs.io/en/latest/ "Welcome to GeoPy’s documentation! — GeoPy 2.5.0 documentation"))

---

## 2. Conceptos fundamentales

### Geocodificación directa

Convierte una dirección o nombre de lugar en coordenadas:

```text
"Terrassa, Barcelona, España"
            ↓
(41.56, 2.01)
```

Se realiza mediante el método:

```python
geolocator.geocode(...)
```

### Geocodificación inversa

Convierte unas coordenadas en una dirección aproximada:

```text
(41.56, 2.01)
        ↓
"Terrassa, Vallès Occidental, Catalunya..."
```

Se realiza mediante:

```python
geolocator.reverse(...)
```

Los geocodificadores de GeoPy disponen normalmente de `geocode()` y, cuando el servicio lo permite, de `reverse()`. ([geopy.readthedocs.io](https://geopy.readthedocs.io/en/latest/ "Welcome to GeoPy’s documentation! — GeoPy 2.5.0 documentation"))

---

# 3. Geocodificación con Nominatim

Nominatim utiliza los datos de OpenStreetMap. Es cómodo para aprender y realizar consultas ocasionales porque no requiere una clave API.

```python
from geopy.geocoders import Nominatim

geolocator = Nominatim(
    user_agent="curso-geopy-ivan/1.0",
    timeout=10
)

location = geolocator.geocode(
    "Ajuntament de Terrassa, Barcelona, España",
    language="es"
)

if location is not None:
    print(location.address)
    print(location.latitude)
    print(location.longitude)
else:
    print("No se encontró la dirección")
```

Es obligatorio utilizar un `user_agent` que identifique tu aplicación. No debes dejar el identificador predeterminado ni utilizar uno genérico. ([OSM Fundación](https://operations.osmfoundation.org/policies/nominatim/ "Nominatim Usage Policy (aka Geocoding Policy)"))

## El objeto `Location`

Cuando encuentra un resultado, GeoPy devuelve un objeto `Location`.

Sus propiedades principales son:

```python
location.address
location.latitude
location.longitude
location.altitude
location.raw
```

Por ejemplo:

```python
print("Dirección:", location.address)
print("Coordenadas:", location.latitude, location.longitude)
print("Respuesta completa:", location.raw)
```

`location.raw` contiene el diccionario original devuelto por el proveedor y puede incluir campos como el tipo de elemento, identificadores de OpenStreetMap o componentes de la dirección. ([geopy.readthedocs.io](https://geopy.readthedocs.io/en/latest/?utm_source=chatgpt.com "Welcome to GeoPy's documentation!"))

---

# 4. Obtener varios resultados

Por defecto, `geocode()` devuelve solamente un resultado porque `exactly_one=True`.

Para obtener distintas coincidencias:

```python
results = geolocator.geocode(
    "Sabadell",
    exactly_one=False,
    limit=5,
    language="es"
)

if results:
    for location in results:
        print(location.address)
        print(location.latitude, location.longitude)
        print()
```

El resultado puede ser:

- `None`, si no se encontró nada.
    
- Un objeto `Location`, si `exactly_one=True`.
    
- Una lista de objetos `Location`, si `exactly_one=False`. ([geopy.readthedocs.io](https://geopy.readthedocs.io/en/latest/ "Welcome to GeoPy’s documentation! — GeoPy 2.5.0 documentation"))
    

Por ello, no conviene hacer esto directamente:

```python
location = geolocator.geocode("dirección inexistente")
print(location.latitude)  # Puede producir AttributeError
```

Es mejor comprobar siempre:

```python
if location is not None:
    print(location.latitude)
```

---

# 5. Geocodificación inversa

Para obtener una dirección a partir de coordenadas:

```python
from geopy.geocoders import Nominatim

geolocator = Nominatim(
    user_agent="curso-geopy-ivan/1.0",
    timeout=10
)

coordinates = (41.5632, 2.0089)

location = geolocator.reverse(
    coordinates,
    language="es"
)

if location is not None:
    print(location.address)
    print(location.raw)
```

Las coordenadas deben escribirse normalmente en este orden:

```python
(latitud, longitud)
```

Es decir:

```python
(41.5632, 2.0089)
```

y no:

```python
(2.0089, 41.5632)
```

GeoPy trabaja con el orden `(latitud, longitud)`, aunque otras bibliotecas y formatos GIS utilizan `(longitud, latitud)`. ([geopy.readthedocs.io](https://geopy.readthedocs.io/en/latest/ "Welcome to GeoPy’s documentation! — GeoPy 2.5.0 documentation"))

---

# 6. Limitar los resultados a un país

Cuando buscas nombres ambiguos, puedes limitar la consulta mediante `country_codes`.

```python
location = geolocator.geocode(
    "Victoria",
    country_codes="es",
    language="es"
)

if location:
    print(location.address)
```

También puedes añadir información directamente a la consulta:

```python
location = geolocator.geocode(
    "Victoria, Álava, España",
    language="es"
)
```

Cuanta más información tenga la dirección, normalmente menos ambigua será la búsqueda:

```text
calle + número + ciudad + provincia + país
```

---

# 7. Calcular distancias

GeoPy puede calcular la distancia geodésica entre dos puntos.

```python
from geopy.distance import geodesic

terrassa = (41.5632, 2.0089)
barcelona = (41.3874, 2.1686)

distance = geodesic(terrassa, barcelona)

print(distance.km)
print(distance.meters)
print(distance.miles)
```

También puedes utilizar:

```python
from geopy.distance import distance

result = distance(terrassa, barcelona)

print(result.km)
```

`geopy.distance.distance` utiliza actualmente la distancia geodésica, calculada sobre un modelo elipsoidal de la Tierra. Es más precisa que `great_circle`, que aproxima la Tierra mediante una esfera y puede presentar un error de hasta aproximadamente el 0,5 %. ([geopy.readthedocs.io](https://geopy.readthedocs.io/en/latest/ "Welcome to GeoPy’s documentation! — GeoPy 2.5.0 documentation"))

## Distancia geodésica frente a distancia por carretera

El resultado representa aproximadamente la distancia más corta sobre la superficie terrestre:

```text
Punto A ───────── Punto B
```

No representa:

- La distancia conduciendo.
    
- La longitud de una carretera.
    
- El tiempo de viaje.
    
- Una ruta de transporte público.
    

Para esas operaciones necesitarías un servicio de rutas, como OSRM, OpenRouteService, Google Directions o Mapbox Directions.

---

# 8. Geocodificar dos lugares y medir su distancia

```python
from geopy.geocoders import Nominatim
from geopy.distance import geodesic
from geopy.extra.rate_limiter import RateLimiter

geolocator = Nominatim(
    user_agent="curso-geopy-ivan/1.0",
    timeout=10
)

# Introduce como mínimo un segundo entre solicitudes.
geocode = RateLimiter(
    geolocator.geocode,
    min_delay_seconds=1
)

terrassa = geocode("Terrassa, Barcelona, España", language="es")
girona = geocode("Girona, España", language="es")

if terrassa is not None and girona is not None:
    punto_terrassa = (terrassa.latitude, terrassa.longitude)
    punto_girona = (girona.latitude, girona.longitude)

    resultado = geodesic(punto_terrassa, punto_girona)

    print(f"Distancia geodésica: {resultado.km:.2f} km")
else:
    print("No se pudo localizar alguna de las ciudades")
```

---

# 9. Límites de Nominatim

> **Importante:** el servidor público de Nominatim no debe utilizarse como si fuera una API ilimitada.

Su política establece, entre otras restricciones:

- Máximo absoluto de **una solicitud por segundo**.
    
- Uso de un `User-Agent` que identifique la aplicación.
    
- Almacenamiento en caché de los resultados.
    
- Prohibición de utilizarlo para autocompletado.
    
- Restricciones importantes para geocodificación masiva y consultas sistemáticas.
    
- No deben enviarse datos personales ni información confidencial. ([OSM Fundación](https://operations.osmfoundation.org/policies/nominatim/ "Nominatim Usage Policy (aka Geocoding Policy)"))
    

Para respetar el límite en consultas pequeñas puedes utilizar `RateLimiter`:

```python
from geopy.extra.rate_limiter import RateLimiter

geocode = RateLimiter(
    geolocator.geocode,
    min_delay_seconds=1
)

location = geocode("Terrassa, España")
```

GeoPy incluye `RateLimiter` precisamente para espaciar las solicitudes y puede configurar reintentos y tratamiento de excepciones. ([geopy.readthedocs.io](https://geopy.readthedocs.io/en/latest/ "Welcome to GeoPy’s documentation! — GeoPy 2.5.0 documentation"))

Para proyectos grandes o comerciales debes utilizar un proveedor apropiado, contratar una API o instalar tu propia instancia de Nominatim. ([OSM Fundación](https://operations.osmfoundation.org/policies/nominatim/ "Nominatim Usage Policy (aka Geocoding Policy)"))

---

# 10. Geocodificar una columna de Pandas

Para un conjunto pequeño de datos educativos:

```python
import pandas as pd
from functools import partial
from geopy.geocoders import Nominatim
from geopy.extra.rate_limiter import RateLimiter

df = pd.DataFrame({
    "ciudad": [
        "Terrassa, España",
        "Girona, España",
        "Tarragona, España"
    ]
})

geolocator = Nominatim(
    user_agent="curso-geopy-ivan/1.0",
    timeout=10
)

geocode_es = partial(
    geolocator.geocode,
    language="es"
)

geocode = RateLimiter(
    geocode_es,
    min_delay_seconds=1
)

df["location"] = df["ciudad"].apply(geocode)

df["latitud"] = df["location"].apply(
    lambda location: location.latitude
    if location is not None
    else None
)

df["longitud"] = df["location"].apply(
    lambda location: location.longitude
    if location is not None
    else None
)

print(df[["ciudad", "latitud", "longitud"]])
```

Para evitar repetir consultas, guarda los resultados:

```python
df.to_csv("ciudades_geocodificadas.csv", index=False)
```

Este enfoque no debe emplearse con el servidor público de Nominatim para procesar grandes bases de datos, realizar trabajos periódicos o lanzar consultas paralelas. Su política exige caché y desaconseja la geocodificación masiva. ([OSM Fundación](https://operations.osmfoundation.org/policies/nominatim/ "Nominatim Usage Policy (aka Geocoding Policy)"))

---

# 11. Tratamiento de errores

Una consulta depende de Internet y de un servicio externo, por lo que puede fallar por tiempo de espera, problemas de conexión, límites de uso o errores del servidor.

```python
from geopy.exc import (
    GeocoderTimedOut,
    GeocoderUnavailable,
    GeocoderServiceError
)

try:
    location = geolocator.geocode(
        "Terrassa, España",
        language="es"
    )

    if location is None:
        print("No se encontró ningún resultado")
    else:
        print(location.address)

except GeocoderTimedOut:
    print("El servicio tardó demasiado en responder")

except GeocoderUnavailable:
    print("El servicio no está disponible")

except GeocoderServiceError as error:
    print(f"Error del servicio: {error}")
```

El parámetro `timeout` determina cuántos segundos espera GeoPy antes de producir una excepción `GeocoderTimedOut`. ([geopy.readthedocs.io](https://geopy.readthedocs.io/en/latest/ "Welcome to GeoPy’s documentation! — GeoPy 2.5.0 documentation"))

También puedes configurar reintentos:

```python
geocode = RateLimiter(
    geolocator.geocode,
    min_delay_seconds=1,
    max_retries=2,
    error_wait_seconds=5,
    swallow_exceptions=False
)
```

---

# 12. Utilizar otros proveedores

GeoPy proporciona una interfaz semejante para distintos servicios.

Por ejemplo, con Google:

```python
from geopy.geocoders import GoogleV3

geolocator = GoogleV3(api_key="TU_CLAVE_API")

location = geolocator.geocode("Terrassa, España")
```

Con ArcGIS:

```python
from geopy.geocoders import ArcGIS

geolocator = ArcGIS()

location = geolocator.geocode("Terrassa, España")
```

Cada proveedor tiene sus propias condiciones, precios, límites, parámetros, base de datos y requisitos de autenticación. Cambiar de proveedor puede cambiar los resultados incluso cuando se utiliza exactamente la misma dirección. ([geopy.readthedocs.io](https://geopy.readthedocs.io/en/latest/ "Welcome to GeoPy’s documentation! — GeoPy 2.5.0 documentation"))

---

# 13. Errores frecuentes

### Confundir latitud y longitud

Incorrecto:

```python
punto = (longitud, latitud)
```

Correcto para GeoPy:

```python
punto = (latitud, longitud)
```

### No comprobar si se obtuvo un resultado

Incorrecto:

```python
location = geolocator.geocode("algo")
print(location.latitude)
```

Correcto:

```python
location = geolocator.geocode("algo")

if location is not None:
    print(location.latitude)
```

### No especificar `user_agent`

Incorrecto:

```python
geolocator = Nominatim()
```

Correcto:

```python
geolocator = Nominatim(
    user_agent="mi_aplicacion/1.0"
)
```

### Hacer peticiones en un bucle sin límite

Incorrecto:

```python
for address in addresses:
    geolocator.geocode(address)
```

Para pequeñas pruebas:

```python
geocode = RateLimiter(
    geolocator.geocode,
    min_delay_seconds=1
)

for address in addresses:
    location = geocode(address)
```

### Esperar una ruta por carretera

```python
geodesic(origen, destino).km
```

calcula una distancia sobre la superficie terrestre, no una ruta de navegación.

---

# 14. Resumen práctico

```python
from geopy.geocoders import Nominatim
from geopy.distance import geodesic

geolocator = Nominatim(
    user_agent="curso-geopy-ivan/1.0",
    timeout=10
)

# Dirección → coordenadas
location = geolocator.geocode(
    "Terrassa, España",
    language="es"
)

if location:
    print(location.address)
    print(location.latitude, location.longitude)

# Coordenadas → dirección
location = geolocator.reverse(
    (41.5632, 2.0089),
    language="es"
)

if location:
    print(location.address)

# Distancia entre coordenadas
punto_a = (41.5632, 2.0089)
punto_b = (41.3874, 2.1686)

print(geodesic(punto_a, punto_b).km)
```

La idea fundamental es:

```text
GeoPy
├── geocode()       Dirección → coordenadas
├── reverse()       Coordenadas → dirección
├── Location        Resultado de una consulta
├── geodesic()      Distancia entre puntos
└── RateLimiter     Control del ritmo de solicitudes
```