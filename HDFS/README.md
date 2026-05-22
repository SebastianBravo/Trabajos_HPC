# Experimentos con Spark, HDFS y procesamiento distribuido

Este repositorio contiene dos notebooks desarrollados para el taller de **Procesamiento de Alto Volumen de Datos**. Los experimentos evalúan el comportamiento de un entorno distribuido basado en **Apache Spark** y **HDFS**, comparándolo con alternativas locales o secuenciales.

Los notebooks incluidos son:

```text
.
├── 01_exp_io_hdfs_vs_local.ipynb
├── 02_exp_wordcount.ipynb
└── README.md
```

## Descripción general

El repositorio está compuesto por dos experimentos principales:

1. **Rendimiento de I/O: HDFS vs. sistema local compartido**  
   Compara tiempos y throughput de escritura y lectura usando archivos CSV sintéticos de distintos tamaños.

2. **WordCount distribuido: Spark + HDFS vs. Python local**  
   Implementa un conteo de palabras sobre un corpus sintético almacenado en HDFS y compara diferentes estrategias de procesamiento: Python puro, Spark RDD, Spark DataFrame y Spark SQL.

## Requisitos previos

Para ejecutar correctamente los notebooks se requiere un entorno con:

- Apache Spark configurado en modo cluster o standalone.
- HDFS disponible y accesible desde los nodos del cluster.
- Python con soporte para PySpark.
- Jupyter Notebook o JupyterLab.
- Librerías de Python:
  - `pandas`
  - `matplotlib`
  - `pyspark`

Los notebooks están configurados para usar una IP de master y rutas específicas del entorno de ejecución. Antes de ejecutarlos, se deben revisar y ajustar las siguientes variables según la infraestructura disponible:

```python
MASTER_IP
SPARK_MASTER
HDFS_URI
CONDA_ENV
HADOOP_HOME
HADOOP_BIN
```

## Notebook 1: `01_exp_io_hdfs_vs_local.ipynb`

### Objetivo

Medir y comparar el rendimiento de lectura y escritura entre **HDFS** y un sistema local compartido mediante **NFS**, usando archivos CSV sintéticos de diferentes tamaños.

El experimento busca observar cómo cambia el rendimiento cuando aumenta el volumen de datos, especialmente en escenarios donde HDFS puede aprovechar la distribución y partición de archivos en bloques.

### Hipótesis

El notebook plantea que HDFS puede superar al almacenamiento local en la lectura de archivos grandes debido al paralelismo de sus DataNodes, aunque puede presentar mayor latencia en archivos pequeños por la sobrecarga propia del NameNode y de la coordinación distribuida.

### Flujo del experimento

El notebook sigue estos pasos:

1. Configura una sesión de Spark conectada al master del cluster.
2. Define las rutas de trabajo locales, NFS y HDFS.
3. Genera archivos CSV sintéticos con columnas como `id`, `nombre`, `edad`, `salario`, `departamento` y `score`.
4. Copia los archivos generados al directorio NFS para que sean visibles por los nodos del cluster.
5. Sube los archivos a HDFS usando el comando `hdfs dfs -put`.
6. Mide el tiempo de escritura hacia HDFS.
7. Verifica los bloques generados en HDFS mediante `hdfs fsck`.
8. Lee los archivos desde HDFS usando Spark.
9. Lee los mismos archivos desde NFS usando Spark.
10. Calcula tiempos de lectura, throughput y número de filas procesadas.
11. Genera una tabla resumen y una gráfica comparativa.

### Métricas evaluadas

- Tiempo de escritura hacia HDFS.
- Throughput de escritura hacia HDFS.
- Tiempo de lectura desde HDFS.
- Throughput de lectura desde HDFS.
- Tiempo de lectura desde NFS.
- Throughput de lectura desde NFS.
- Número de bloques HDFS generados.
- Número de filas leídas por Spark.

### Resultados principales

En la ejecución registrada en el notebook, los resultados obtenidos fueron los siguientes:

| Tamaño | MB reales | Put HDFS (s) | TP Put (MB/s) | Read HDFS (s) | TP HDFS (MB/s) | Read NFS (s) | TP NFS (MB/s) |
|---|---:|---:|---:|---:|---:|---:|---:|
| 1 MB | 0.5 | 3.13 | 0.2 | 7.94 | 0.1 | 2.16 | 0.2 |
| 50 MB | 26.8 | 2.55 | 10.5 | 7.63 | 3.5 | 4.07 | 6.6 |
| 200 MB | 109.1 | 2.94 | 37.1 | 3.49 | 31.3 | 1.76 | 61.9 |
| 2 GB | 856.1 | 5.80 | 147.5 | 4.47 | 191.7 | 3.80 | 225.2 |

Los resultados muestran que, para archivos pequeños, la sobrecarga de HDFS es significativa. En cambio, a medida que aumenta el tamaño del archivo, el throughput de HDFS mejora considerablemente, especialmente en lectura. Sin embargo, en esta ejecución específica el NFS compartido mantuvo mejores tiempos de lectura que HDFS en todos los tamaños evaluados.

### Archivos generados

Durante la ejecución, el notebook genera los siguientes archivos de salida:

```text
/tmp/exp1_resultados.csv
/tmp/exp1_grafica.png
```

El archivo CSV almacena la tabla resumen del experimento y la imagen contiene la comparación gráfica de throughput y tiempo de lectura.

## Notebook 2: `02_exp_wordcount.ipynb`

### Objetivo

Implementar el algoritmo clásico **WordCount** sobre un corpus de texto grande almacenado en HDFS, comparando el tiempo de ejecución entre procesamiento local y procesamiento distribuido con Spark.

El notebook compara cuatro enfoques:

1. Python puro sobre archivo local.
2. Spark RDD leyendo desde HDFS.
3. Spark DataFrame leyendo desde HDFS.
4. Spark SQL leyendo desde HDFS.

### Relevancia del experimento

Este experimento permite observar la diferencia entre procesamiento secuencial y procesamiento distribuido para una tarea típica de análisis de texto. También muestra cómo Spark puede ejecutar la misma lógica mediante distintas abstracciones: RDD, DataFrame y SQL.

### Flujo del experimento

El notebook realiza los siguientes pasos:

1. Configura una sesión de Spark conectada al master del cluster.
2. Define rutas locales, NFS y HDFS.
3. Genera un corpus sintético de texto con palabras relacionadas con Hadoop, Spark, HDFS y procesamiento distribuido.
4. Copia el corpus al directorio NFS.
5. Sube el corpus a HDFS.
6. Ejecuta WordCount con Python puro como línea base secuencial.
7. Ejecuta WordCount con Spark RDD desde HDFS.
8. Ejecuta WordCount con Spark DataFrame desde HDFS.
9. Ejecuta WordCount con Spark SQL desde HDFS.
10. Compara los tiempos de ejecución y calcula el speedup frente a Python puro.
11. Genera gráficas comparativas.
12. Guarda el resultado completo del WordCount en HDFS en formato Parquet.

### Corpus utilizado

El corpus sintético generado tiene las siguientes características en la ejecución registrada:

- 500.000 líneas de texto.
- Tamaño aproximado: 56.6 MB.
- Vocabulario compuesto por términos asociados a big data y procesamiento distribuido, como `hadoop`, `spark`, `hdfs`, `cluster`, `namenode`, `datanode`, `dataframe`, `rdd`, `shuffle`, `executor`, `driver`, entre otros.

### Métodos implementados

#### Python puro

Procesa el archivo local línea por línea usando `collections.Counter`. Este método sirve como baseline secuencial porque no utiliza Spark ni procesamiento distribuido.

#### Spark RDD

Lee el corpus desde HDFS con `sc.textFile`, separa las palabras con `flatMap`, crea pares `(palabra, 1)` y agrupa con `reduceByKey`.

#### Spark DataFrame

Lee el corpus como DataFrame de texto, separa las palabras con funciones de Spark SQL como `split`, `explode`, `lower`, `groupBy` y `count`.

#### Spark SQL

Registra el corpus como vista temporal y ejecuta una consulta SQL para separar, agrupar, contar y ordenar las palabras por frecuencia.

### Resultados principales

En la ejecución registrada en el notebook, los resultados fueron:

| Método | Tiempo (s) | Fuente | Speedup vs Python |
|---|---:|---|---:|
| Python puro (local) | 3.316 | Local | 1.00 |
| Spark RDD (HDFS) | 9.432 | HDFS | 0.35 |
| Spark DataFrame (HDFS) | 10.520 | HDFS | 0.32 |
| Spark SQL (HDFS) | 1.565 | HDFS | 2.12 |

El enfoque más rápido en esta ejecución fue **Spark SQL**, con un tiempo de 1.565 segundos y un speedup de 2.12 veces frente a Python puro. En cambio, Spark RDD y Spark DataFrame fueron más lentos que Python puro para este tamaño de corpus, probablemente por la sobrecarga de inicialización, planificación y ejecución distribuida frente a un archivo relativamente pequeño.

### Palabras más frecuentes

El Top 20 de palabras generado por Spark SQL incluye, entre otras:

| Palabra | Frecuencia |
|---|---:|
| hadoop | 159830 |
| spark | 159779 |
| standalone | 159763 |
| transformacion | 159697 |
| fichero | 159695 |
| distribuido | 159522 |
| dato | 159470 |
| sistema | 159444 |
| python | 159433 |
| proceso | 159395 |
| rendimiento | 159321 |
| nodo | 159285 |
| archivo | 159259 |
| yarn | 159252 |
| bloque | 159239 |
| dataframe | 159185 |
| datanode | 159180 |
| red | 159172 |
| memoria | 159153 |
| pyspark | 159128 |

### Archivos generados

Durante la ejecución, el notebook genera los siguientes archivos de salida:

```text
/tmp/exp2_resultados.csv
/tmp/exp2_grafica.png
/tmp/exp2_top_palabras.png
```

Además, persiste el resultado completo del WordCount en HDFS en formato Parquet:

```text
hdfs://10.43.97.145:9000/experimentos/resultados/wordcount_completo.parquet
```

## Ejecución recomendada

Para reproducir los experimentos, se recomienda ejecutar los notebooks en el siguiente orden:

```text
1. 01_exp_io_hdfs_vs_local.ipynb
2. 02_exp_wordcount.ipynb
```

Antes de ejecutar, se deben validar las rutas, permisos y variables de entorno del cluster. También se recomienda confirmar que los nodos worker puedan acceder correctamente a las rutas compartidas por NFS y que HDFS esté disponible desde el driver.

## Estructura de salidas esperadas

Al finalizar la ejecución, se esperan salidas locales temporales y salidas persistidas en HDFS:

```text
/tmp/
├── exp1_resultados.csv
├── exp1_grafica.png
├── exp2_resultados.csv
├── exp2_grafica.png
└── exp2_top_palabras.png

hdfs://<MASTER_IP>:9000/experimentos/
├── exp1_io/
├── exp2_wordcount/
└── resultados/wordcount_completo.parquet
```

## Conclusión

Los experimentos permiten comparar de forma práctica el comportamiento de HDFS y Spark frente a alternativas locales. El primer notebook evidencia que HDFS tiene una sobrecarga inicial importante, pero mejora su throughput cuando aumenta el tamaño de los archivos. El segundo notebook muestra que Spark SQL puede superar al procesamiento local incluso en un corpus moderado, mientras que RDD y DataFrame pueden verse afectados por la sobrecarga de ejecución distribuida cuando el volumen de datos no es lo suficientemente grande.
