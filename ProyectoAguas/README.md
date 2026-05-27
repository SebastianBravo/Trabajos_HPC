# ProyectoAguas — Calidad del Agua (India)

Resumen del proyecto

- Objetivo: construir un Índice de Calidad del Agua (WQI) a partir de mediciones fisicoquímicas y microbiológicas de ríos en 18 estados de India, y comparar modelos predictivos (Regresión Lineal distribuida con PySpark MLlib y Red Neuronal con TensorFlow/Keras).
- Datos: `waterquality.csv` (original: 534 registros; tras limpieza: 447 registros). Los datos brutos y procesados se encuentran en este directorio.

Notebook principal

- El análisis y la construcción del WQI están contenidos en el notebook: [Clean_ML_Water_Corregido.ipynb](Clean_ML_Water_Corregido.ipynb).
  - Sección 7: `Construcción del Índice de Calidad del Agua (WQI)` muestra cómo se definen los rangos y puntajes para cada parámetro.

Análisis y hallazgos

- Preprocesamiento: se realizó limpieza de nulos (eliminación de filas con variables predictoras nulas), manejo de tipos y EDA (estadísticas, boxplots, distribuciones).
- EDA clave:
  - Pérdida de ~16% de registros tras eliminar nulos.
  - `FECAL_COLIFORM` y `CONDUCTIVITY` presentan mayor cantidad de nulos y alta dispersión.
  - Outliers en `CONDUCTIVITY`, `BOD` y `FECAL_COLIFORM` indican puntos de contaminación puntuales.
  - pH es relativamente estable; DO muestra mayor variabilidad, relevante para la biota acuática.

WQI

- Metodología: cada parámetro se transforma a un puntaje de calidad (0 / 40 / 60 / 80 / 100) según rangos bibliográficos y luego se pondera.
- Pesos principales (ejemplo): DO 28.1%, Coliformes 28.1%, Conductividad 23.4%, pH 16.5%, Nitratos 2.8%, BOD 0.9%.
- Resultado: predominio de estaciones con calidad **Baja** (≈60.6%); solo ~2.5% caen en categoría Excelente.

Modelos

- Modelos evaluados:
  - Regresión Lineal distribuida (PySpark MLlib) — excelente desempeño para WQI por su naturaleza lineal y ponderada.
  - Red Neuronal (TensorFlow/Keras) — buena generalización, pero no superó al modelo lineal para este objetivo.
- Recomendación: usar Regresión Lineal/MLlib para despliegues a escala; redes si se añaden relaciones no lineales o más features.

Gráficas y recursos

- Todas las gráficas utilizadas en la presentación están en la carpeta `Images/`. Ejemplos:
  - `Images/Índice de Calidad del Agua (WQI) por Estado.png`
  - `Images/Regresion Lineal MLlib — Prediccion vs Real.png`
  - `Images/Red Neuronal Keras — Prediccion vs WQI Real.png`
  - `Images/Convergencia del Modelo Keras.png`

Presentación

- La presentación en HTML que resume el trabajo está en: [Presentacion_Calidad_Agua_India.html](Presentacion_Calidad_Agua_India.html).

Requisitos y ejecución

- Requisitos (sugeridos):
  - Python 3.10+ with: pandas, numpy, matplotlib, seaborn, scikit-learn, pyspark, tensorflow, keras, jupyter
  - Apache Spark + Hadoop HDFS si se desea reproducir la carga desde HDFS

- Para visualizar la presentación: abrir `Presentacion_Calidad_Agua_India.html` en el navegador.

Contribuciones y autor

- Autor: Juan Sebastián Bravo Santacruz