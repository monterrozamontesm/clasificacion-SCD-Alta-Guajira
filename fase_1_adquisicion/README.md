# Pipeline de Adquisición de Imágenes Satelitales

Notebook de adquisición automatizada del banco de 600 imágenes satelitales que sustenta el entrenamiento del modelo de clasificación de depresiones sub-circulares en la Alta Guajira. Las imágenes se descargan desde el servicio Google Maps Static API mediante consultas HTTP directas, se recortan a un tamaño uniforme de 1184×1184 píxeles y se organizan automáticamente en subcarpetas por clase geomorfológica.

---

## Contexto del proyecto

Este pipeline constituye la Fase 1 de la metodología del trabajo de grado en Geología de Miguel Ángel Monterroza Montes (Universidad del Norte, Barranquilla), sobre clasificación sistemática de depresiones sub-circulares asociadas a hidrógeno natural (SCD-H₂) mediante YOLOv8. 

El mismo notebook se reutiliza posteriormente (con ajustes menores documentados más abajo) para adquirir el banco de imágenes de aplicación sobre el cual el modelo entrenado emite su clasificación.

---

## Requisitos

### Software

- **Sistema operativo:** Windows 10 u 11 (probado). Compatible con Linux y macOS con adaptaciones mínimas.
- **Miniconda o Anaconda** con `conda` 26.x o superior.
- **Python 3.11** (instalado dentro del entorno conda dedicado).
- **VS Code** con las extensiones de Python y Jupyter.

### Librerías Python

| Librería | Versión probada | Función |
|---|---|---|
| `pandas` | 2.2.3 | Lectura y manipulación tabular del CSV |
| `pyproj` | 3.7.0 | Transformación entre sistemas de coordenadas |
| `requests` | 2.33.1 | Consultas HTTP al servicio Static API |
| `Pillow` (PIL) | 11.0.0 | Procesamiento y recorte de imágenes |
| `ipykernel` | — | Kernel de Jupyter para ejecución en VS Code |

Las demás dependencias (`os`, `sys`, `time`, `csv`, `datetime`, `pathlib`, `io`) forman parte de la biblioteca estándar de Python y no requieren instalación adicional.

### Credenciales

- **API key de Google Maps Static API**, obtenida a través de Google Cloud Console con facturación habilitada (opera bajo cuotas mensuales gratuitas a lo usuarios).

### Archivos de entrada

- CSV maestro con la nómina de las 600 estructuras a descargar, en formato específico documentado más abajo (`Dataset_Maestro_600_CSV_Static_API.csv`).

---

## Instalación

Se creó un entorno conda dedicado llamado `hidrogeno` desde Anaconda Prompt, sobre el que se instalaron las librerías listadas en la sección de requisitos utilizando el canal `conda-forge`. Una vez creado el entorno y verificada la disponibilidad de las librerías, el pipeline queda listo para ejecutarse desde VS Code apuntando al kernel de dicho entorno.

---

## Uso del script

### Preparación previa

Antes de la primera ejecución, se debe configurar la API key de Google Maps Static API. Para ello, abrir el notebook y localizar la Celda 2 (`Configuración de la API Key`). Reemplazar la cadena `TU_API_KEY_AQUI` por la clave real.

### Ejecución estándar

Con el notebook abierto en VS Code y el kernel `hidrogeno (Python 3.11)` activo, ejecutar las celdas secuencialmente con `Shift+Enter`, o usar `Run All` para procesar todo de una vez. En condiciones normales de red, la descarga completa de las 600 estructuras toma aproximadamente 17 minutos.

### Reanudación tras interrupciones

El pipeline es idempotente: si se interrumpe a mitad de la ejecución, al volver a ejecutar la Celda 12 detecta automáticamente qué estructuras ya fueron descargadas (consultando el log y verificando presencia en disco) y salta esas descargas. Solo procesa las pendientes o las que fallaron en intentos previos.

### Re-descarga selectiva

Para forzar la re-descarga de estructuras específicas (por ejemplo, tras corregir coordenadas o zoom en el CSV):

1. Borrar el archivo PNG correspondiente de la carpeta de salida.
2. Eliminar el registro del log del CSV de trazabilidad.
3. Re-ejecutar la Celda 12.

---

## Estructura de entrada

### Formato del CSV maestro

Archivo separado por punto y coma (`;`) con la coma (`,`) como separador decimal (convención europea). Debe contener exactamente estas cinco columnas:

| Columna | Tipo | Descripción |
|---|---|---|
| `id` | entero | Identificador único de la estructura (1 a 600) |
| `clase` | texto | Nombre de la clase geomorfológica (10 valores posibles) |
| `X_3857` | decimal | Coordenada X del centroide en Web Mercator (EPSG:3857), en metros |
| `Y_3857` | decimal | Coordenada Y del centroide en Web Mercator (EPSG:3857), en metros |
| `zoom` | entero | Nivel de zoom empírico calibrado para la estructura (típicamente entre 11 y 21) |

### Clases del inventario

```
Fairy circle, Fairy fort, Farm circle, Flooded dune, H2,
Impact crater, Jaguey, Karst, Salt lake, Thermokarst
```

Distribución balanceada: 60 estructuras por clase, 600 en total.

---

## Estructura de salida

### Organización de las imágenes

```
dataset_600_static_api/
├── Fairy_circle/
│   ├── 1.png
│   ├── 2.png
│   └── ... (60 imágenes)
├── Fairy_fort/
│   └── ... (60 imágenes)
├── Farm_circle/
├── Flooded_dune/
├── H2/
├── Impact_crater/
├── Jaguey/
├── Karst/
├── Salt_lake/
└── Thermokarst/
```

- **Convención de nombres de carpetas:** los espacios en el nombre de la clase se reemplazan por guiones bajos.
- **Convención de nombres de archivos:** cada PNG toma el nombre del `id` de la estructura, sin ceros a la izquierda (`1.png`, `47.png`, `600.png`).
- **Dimensiones:** 1184×1184 píxeles, recorte centrado sobre la imagen original de 1280×1280 descargada del servicio.
- **Formato:** PNG sin pérdida (compresión lossless), 8 bits por canal, RGB.

### Log de trazabilidad

Archivo `log_descarga_static_api.csv` en la raíz del proyecto, generado y actualizado por cada ejecución. Formato estándar CSV (separador coma, decimal punto) con las siguientes columnas:

| Columna | Descripción |
|---|---|
| `timestamp` | Marca temporal ISO 8601 del momento de la descarga |
| `id` | ID de la estructura procesada |
| `clase` | Clase geomorfológica |
| `zoom` | Nivel de zoom usado |
| `lon` | Longitud EPSG:4326 (grados decimales) |
| `lat` | Latitud EPSG:4326 (grados decimales) |
| `url_sin_key` | URL completa de la petición, sin la API key (por seguridad) |
| `ruta` | Ruta relativa del PNG generado |
| `tamano_recorte` | Tamaño del recorte aplicado (1184) |
| `bytes_descargados` | Tamaño en bytes de la imagen original recibida del servicio |
| `estado` | `OK`, `ERROR` o `EXCEPCION` |
| `mensaje` | Descripción del error si `estado` es distinto de `OK` |

---

## Configuración

Todos los parámetros operativos se centralizan en el diccionario `CONFIG` de la Celda 3 del notebook.

### Parámetros que cambian según el CSV procesado

Para reutilizar el pipeline con un CSV diferente (por ejemplo el de aplicación), solo se modifican estos tres valores:

```python
"csv_entrada": "Dataset_Maestro_600_CSV_Static_API.csv",
"carpeta_dataset": "dataset_600_static_api",
"log_csv": "log_descarga_static_api.csv",
```

Cada CSV se procesa aisladamente: su carpeta de salida y su log son independientes, sin contaminación entre ejecuciones.

### Parámetros técnicos (fijos, no modificar sin razón fundamentada)

- **Formato del CSV:** separador `;`, decimal `,`.
- **Sistemas de referencia:** origen EPSG:3857 (Web Mercator), destino EPSG:4326 (WGS 84 geográfico).
- **Static API:** `size=640x640`, `scale=2` (produce imagen 1280×1280), `maptype=satellite`, `style` con etiquetas desactivadas.
- **Red:** timeout de 30 segundos por petición, pausa de 0.15 segundos entre descargas (rate limiting preventivo), hasta 3 reintentos con espera exponencial base de 5 segundos.
- **Recorte:** tamaño original 1280, recorte centrado a 1184 (múltiplo de 32 requerido por YOLOv8).

### Parámetros de prueba

Para pruebas rápidas sin procesar el CSV completo, se pueden activar temporalmente:

- `limite_estructuras`: número entero para procesar solo las primeras N estructuras (`None` para procesar todo).
- `filtro_clase`: nombre exacto de una clase para procesar solo esa (`None` para todas).

---

## Decisiones metodológicas

Las decisiones documentadas a continuación fueron adoptadas tras validación empírica durante el desarrollo del pipeline y quedan registradas aquí para trazabilidad.

**Uso de Static API en lugar de captura de pantalla automatizada.** Un trabajo previo (Ginzburg et al., 2025) empleaba automatización de capturas de pantalla sobre la interfaz web de Google Maps. La Static API se prefiere por tres razones: resolución nativa superior, ausencia de elementos de interfaz gráfica superpuestos, y reproducibilidad plena en cualquier entorno independientemente del hardware de visualización.

**Recorte a 1184×1184.** El servicio devuelve imágenes de 1280×1280 con una franja inferior con atribución del servicio. El recorte centrado a 1184 elimina esa franja simétricamente para preservar la centralidad geográfica de la estructura de interés.

**Coordenadas en EPSG:3857 desde ArcGIS Pro.** El CSV maestro se genera exportando la capa de puntos desde ArcGIS Pro, que trabaja nativamente en Web Mercator. El pipeline transforma internamente a EPSG:4326 (geográfico) porque es el sistema requerido por la Static API.

**Zoom por estructura.** Cada fila del CSV lleva un valor de zoom calibrado empíricamente en función del tamaño típico de la estructura representada. Zoom demasiado alejado dificulta el reconocimiento visual; zoom demasiado cercano corta la estructura. La calibración se validó estructura por estructura durante la construcción del CSV maestro.

**Log CSV en modo append con flush por fila.** El log se escribe fila a fila y no se mantiene en memoria hasta el final. Esto garantiza que una interrupción abrupta (corte de energía, cierre forzado) preserve todo el progreso hasta la última descarga completada.

---

## Solución de problemas comunes

**Error `API key no configurada`.**
La Celda 2 aún contiene el placeholder `TU_API_KEY_AQUI`. Reemplazar por la clave real.

**Error `CSV faltante`.**
El archivo indicado en `csv_entrada` no existe en la ruta esperada. Verificar que el CSV está en la raíz del proyecto y que el nombre coincide exactamente.

**Error `CSV con columnas faltantes`.**
El CSV no tiene alguna de las columnas requeridas (`id`, `clase`, `X_3857`, `Y_3857`, `zoom`). Verificar los encabezados de la primera fila.

**Descargas con estado `ERROR` y código HTTP 403.**
La API key está mal configurada, no tiene habilitado el servicio de Static API, o la cuenta de Google Cloud no tiene facturación activada. Revisar Google Cloud Console.

**Descargas con estado `ERROR` y código HTTP 429.**
Se está superando el límite de tasa del servicio. Aumentar `pausa_entre_descargas_s` en `CONFIG` (por ejemplo a 0.5) y re-ejecutar; la lógica de reanudación garantiza que no se pierda el progreso.

**Discrepancias entre disco y log al final de la ejecución.**
Indica que hubo una interrupción anómala entre la escritura del PNG y la escritura del log (raro, pero posible). Inspeccionar la Celda 14 para ver qué IDs están desalineados y corregir manualmente (borrar el PNG huérfano o completar el registro del log).

**El notebook no encuentra las funciones definidas en celdas previas.**
El kernel se reinició. Volver a ejecutar todas las celdas desde el inicio con `Run All Above` sobre la celda actual.

---

## Autor y contacto

**Miguel Ángel Monterroza Montes**  
Estudiante de Geología, Universidad del Norte  
Barranquilla, Colombia  

Trabajo de grado acerca clasificación y priorización de SCD-H₂ en la Alta Guajira (Colombia) con YOLOv8-cls.

**Dirección académica:**  
- Director: M.Sc. Iván Darío Olaya López  
