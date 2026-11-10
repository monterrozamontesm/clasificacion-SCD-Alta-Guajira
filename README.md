# Clasificación y priorización de SCD en la Alta Guajira con YOLOv8-cls

Código y resultados del trabajo de grado *Clasificación y priorización sistemática mediante visión artificial de potenciales depresiones subcirculares (SCD-H₂) en la región de la Alta Guajira, Colombia*, presentado para optar al título de Geólogo en la Universidad del Norte (Barranquilla, Colombia), 2026.

**Autor:** Miguel Angel Monterroza Montes
**Director:** M.Sc. Iván Darío Olaya López

## De qué trata

Las depresiones subcirculares (SCD) se asocian a emanaciones de hidrógeno natural. Este repositorio contiene el flujo completo con el que se entrenó un clasificador de imágenes (YOLOv8n-cls) usando imágenes de satélite de Google Maps Static API para distinguir las SCD de otras nueve geoformas redondeadas, entre ellas los jagüeyes de La Guajira, y con el que se priorizaron 120 estructuras candidatas de la Alta Guajira en tres franjas (alta, media y baja).

## Organización

| Carpeta | Fase del trabajo | Contenido |
|---|---|---|
| [`fase_1_adquisicion/`](fase_1_adquisicion) | Fase 1 (sección 3.1) | Rutina de descarga de imágenes con Google Maps Static API y su README |
| [`fases_2_3_modelamiento/`](fases_2_3_modelamiento) | Fases 2 y 3 (secciones 3.2 y 3.3) | Seis *notebooks* de Google Colaboratory y la carpeta `salidas/` con sus resultados |

El Anexo A del trabajo de grado (Tabla A1) indica qué *notebook* corresponde a cada sección de la metodología.

## Cómo reproducirlo

1. **Fase 1, en un equipo local.** Ejecutar `fase_1_adquisicion/pipeline_adquisicion_imagenes.ipynb` con una credencial propia de Google Maps Static API. Las instrucciones están en el README de esa carpeta.
2. **Preparar Google Drive.** Crear la carpeta `MyDrive/Geologia_SCD/` con:
   - `dataset/`: las 600 imágenes en sus subcarpetas por clase y el CSV maestro;
   - `dataset_aplicacion/`: las 120 imágenes y su CSV;
   - `salidas/`: la carpeta incluida en este repositorio;
   - los seis *notebooks* de `fases_2_3_modelamiento/`.
3. **Fases 2 y 3, en Google Colaboratory.** Abrir los *notebooks* en orden, de `01` a `06`, con una GPU T4. Cada uno comprueba al iniciar que la partición no haya cambiado (suma de verificación SHA-256).

Como `salidas/` ya trae los resultados de la corrida del trabajo, los *notebooks* 05 y 06 pueden ejecutarse directamente para revisar la priorización y regenerar las figuras.

## Lo que no se incluye

- **Los modelos por pliegue de YOLOv8s-cls y ResNet-18**, por su tamaño. El *notebook* 03 los vuelve a entrenar si no los encuentra. Sí se incluyen el modelo final y los modelos de YOLOv8n-cls que usa el *notebook* 04.
- **Las imágenes satelitales.** 

## Cómo citar

Monterroza Montes, M. A. (2026). *clasificacion-SCD-Alta-Guajira* [Software]. GitHub. https://github.com/monterrozamontesm/clasificacion-SCD-Alta-Guajira
