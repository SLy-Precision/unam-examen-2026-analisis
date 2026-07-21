# Detección de anomalías estadísticas en el examen de admisión UNAM 2026

Análisis estadístico no paramétrico de las distribuciones de aciertos del examen de admisión de la UNAM (2019-2026), evaluando si el comportamiento del examen aplicado en línea en 2026 es consistente con el historial de exámenes presenciales de los ocho años anteriores.

## Contexto

En 2026, por primera vez en su historia, el examen de admisión de la UNAM se aplicó en modalidad en línea. Poco después de publicarse los resultados, comenzaron a circular comparaciones informales entre las distribuciones de aciertos de este examen y las de años anteriores, señalando patrones visualmente distintos a los históricos.

Este proyecto toma datos públicos de los resultados del examen (2019-2026, incluyendo las dos convocatorias de 2019 y 2020) y aplica un conjunto de pruebas estadísticas no paramétricas para evaluar, de forma rigurosa, si esas diferencias visuales tienen respaldo estadístico y qué tan extremas son.

**Importante:** este análisis no identifica personas ni certifica la causa detrás de los patrones encontrados. Mide anomalías en la distribución agregada de resultados por carrera y sede, y evalúa qué tan compatibles son esos resultados con el comportamiento histórico del examen.

## Estructura del repositorio

```
├── 00-Webscrapping.ipynb              # Obtención de los datos publicos de cada examen
├── 01-DataPrep.ipynb                  # Limpieza y transformacion de los datos crudos
├── 02-Analisis_Estadistico.ipynb      # Analisis estadistico principal
├── raw-data/                          # Datos crudos, un archivo por examen (2019-2026)
└── transformed-data/                  # Datos procesados y resultados de cada prueba
```

### `raw-data/`

Un archivo CSV por aplicación de examen, tal como se obtuvo del scraping. 2019 y 2020 tienen dos convocatorias cada uno (febrero y junio); de 2021 en adelante el examen es único por año.

### `transformed-data/`

- `aspirantes_long.csv.gz` — dataset a nivel de aspirante individual (comprimido; ver sección de uso más abajo)
- `resumen_carrera_examen.csv` — agregado por carrera-sede-examen (aspirantes, aceptados, aciertos mínimos, tasa de aceptación)
- `catalogo_carreras.csv` — catálogo de carreras con id único
- `ranking_distancia_wasserstein.csv` — ranking de distancia entre la distribución 2026 y el histórico, por carrera-sede
- `tabla_probabilidad_2026.csv` — resultados de la prueba de permutación (z-scores, p-values)
- `tabla_mezcla_componentes.csv` — estimación del modelo de mezcla (componente histórico vs. anómalo)
- `tabla_percentiles_comparacion.csv` — comparación de percentiles de corte histórico vs. 2026

## Metodología

Todas las pruebas son no paramétricas: no asumen una forma específica (como una distribución normal) para los datos, ya que las distribuciones de aciertos observadas son sesgadas y en algunos casos bimodales.

| Pregunta | Método | Qué mide |
|---|---|---|
| ¿Qué tan probable era ver la distribución de 2026 dado el histórico? | Prueba de permutación sobre la distancia de Wasserstein (5,000 remuestreos por carrera-sede) | Si el resultado de 2026 es estadísticamente compatible con la variación histórica normal |
| ¿Qué carreras-sede se desviaron más de su patrón histórico? | Distancia de Wasserstein entre la distribución 2026 y el pool histórico (2019-2025) | Magnitud del cambio en la forma completa de la distribución |
| ¿Cuántos aspirantes pudieron pertenecer a una población anómala? | Modelo de mezcla con componente conocido (contamination model, vía EM) | Proporción estimada de aspirantes cuyo puntaje no se explica por el patrón histórico |
| ¿Qué tan injusto fue para quien se preparó bajo el estándar histórico? | Comparación de percentiles cruzados (corte histórico vs. 2026) | Devaluación del nivel de conocimiento que antes garantizaba la aceptación |

Todas las pruebas se aplicaron únicamente a carreras-sede con al menos 100 aspirantes en 2026 y al menos 5 exámenes históricos disponibles, para evitar conclusiones basadas en muestras pequeñas y ruidosas.

## Cómo reproducir el análisis

1. Clona el repositorio.
2. Instala las dependencias: `pandas`, `numpy`, `scipy`, `matplotlib`.
3. Corre los notebooks en orden: `00` → `01` → `02`.
4. `aspirantes_long.csv` se distribuye comprimido por su tamaño. Para leerlo:

```python
import pandas as pd
aspirantes_long = pd.read_csv("transformed-data/aspirantes_long.csv.gz")
```

## Limitaciones

- No existen identificadores únicos por aspirante (folio), por lo que no es posible dar seguimiento a una misma persona entre convocatorias ni identificar individuos.
- Las pruebas estadísticas confirman o descartan que un patrón sea explicable por variación histórica normal, pero no establecen por sí solas la causa del fenómeno.
- Las estimaciones de "componente anómalo" son estimaciones poblacionales de masa de probabilidad en exceso, no conteos de personas identificadas.

## Fuente de los datos

Datos públicos de resultados del examen de admisión de la UNAM, obtenidos vía web scraping de las publicaciones oficiales de resultados por carrera y sede.

## Agrecimiento

Gracias a Jonathan Guevara por compartirme su trabajo en este tema en específico.
