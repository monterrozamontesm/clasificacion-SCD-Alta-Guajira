# Fases 2 y 3: modelamiento, evaluación y priorización

Seis *notebooks* de Google Colaboratory que se ejecutan en orden. Se comunican entre sí únicamente a través de la carpeta `salidas/`: cada uno lee lo que dejaron los anteriores y agrega lo suyo. Si a uno le falta un archivo, se detiene e indica qué *notebook* debe ejecutarse antes.

## Notebooks

| Notebook | Sección del trabajo de grado | Qué hace | Archivos que deja en `salidas/` |
|---|---|---|---|
| `01_particion.ipynb` | 3.2.1 | Agrupa las 600 imágenes en sitios, aparta el holdout y forma los cinco pliegues | `particion.csv`, `particion.hash` |
| `02_eda.ipynb` | 3.2.2 | Examina el *zoom*, el color y la ubicación de cada clase y entrena las líneas base | `lineas_base.csv` |
| `03_modelamiento.ipynb` | 3.2.3 | Entrena y compara YOLOv8n-cls, YOLOv8s-cls y ResNet-18 en validación cruzada | `oof_predicciones.csv`, `historia_entrenamiento.csv`, `resumen_modelamiento.json`, `modelos/` |
| `04_interpretaciones.ipynb` | 3.2.4 a 3.2.6 | Entrena el modelo final, lo calibra, lo evalúa en el holdout, hace las pruebas de contexto y de región nueva y fija las franjas | `modelo_final.pt`, `holdout_predicciones.csv`, `calibracion.json`, `umbrales_prioridad.json`, `resumen_interpretaciones.json`, `modelos/region_fuera_*.pt` |
| `05_aplicacion.ipynb` | 3.3 | Aplica el modelo final a las 120 candidatas y construye la matriz de priorización | `aplicacion_predicciones.csv`, `matriz_priorizacion.csv`, `matriz_priorizacion.geojson` |
| `06_figuras_tdg.ipynb` | — | Genera las figuras del documento que no son mapas | `figuras_tdg/` |

## Requisitos

- Google Colaboratory con GPU (la corrida del trabajo usó una NVIDIA T4).
- La librería de YOLOv8 queda fijada en cada *notebook* (`ultralytics==8.3.203`).
- En Google Drive, la carpeta `MyDrive/Geologia_SCD/` con `dataset/`, `dataset_aplicacion/` y `salidas/` (ver el README principal). Si se usa otra carpeta, basta con cambiar la variable `CARPETA_DRIVE` al inicio de cada *notebook*.

## Archivos clave de `salidas/`

- `particion.hash`: suma de verificación de la partición oficial. Los *notebooks* 02 a 06 la comparan antes de trabajar.
- `modelo_final.pt`: el modelo YOLOv8n-cls aplicado a la Alta Guajira. El *notebook* 05 comprueba su identificador SHA-256 antes de predecir.
- `umbrales_prioridad.json`: temperatura de calibración (T = 1,03) y cortes de las franjas (alta desde P(H2) = 0,710 y media desde 0,092).
- `matriz_priorizacion.geojson`: la matriz de priorización de las 120 estructuras, lista para abrir en ArcGIS Pro o QGIS (EPSG:4326).

En los *notebooks* y en los archivos de salida, la clase SCD aparece con la etiqueta `H2`.
