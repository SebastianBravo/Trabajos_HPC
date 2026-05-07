# Taller de Evaluación de Rendimiento MPI

## Comparación de algoritmos de multiplicación de matrices con MPI + OpenMP

Este repositorio contiene la implementación, automatización y análisis experimental del taller de evaluación de rendimiento MPI desarrollado para la asignatura **HPC202601 — Computación de Alto Desempeño** de la Pontificia Universidad Javeriana.

El objetivo principal del taller es comparar el rendimiento de dos variantes de multiplicación de matrices cuadradas implementadas en C, usando un enfoque híbrido de paralelismo con **MPI** para memoria distribuida y **OpenMP** para memoria compartida.

---

## Información general

| Campo | Valor |
|---|---|
| Asignatura | HPC202601 — Computación de Alto Desempeño |
| Profesor | John Corredor Franco |
| Grupo | Grupo 02 |
| Fecha de entrega | 2 de mayo de 2026 |
| Periodo académico | 2026-10 |
| Institución | Pontificia Universidad Javeriana |

### Integrantes

- Juan Sebastián Bravo Santacruz
- Felix David Cordova Garcia
- Jonatan Alejandro Gallo Martinez
- Jose Alejandro Jaime Lopez
- Josman Alexdy Ramirez Torres
- Jesús David Romero Melo
- Juan Camilo Torres Peña

---

## Descripción del problema

La multiplicación de matrices cuadradas densas es una operación fundamental en computación científica, simulación, procesamiento de señales, redes neuronales y álgebra lineal numérica. Sin embargo, su complejidad temporal es **O(N³)**, por lo que su costo computacional aumenta rápidamente cuando crece la dimensión de las matrices.

El taller busca evaluar empíricamente qué combinación de algoritmo, número de procesos MPI y número de hilos OpenMP ofrece mejor rendimiento sobre un clúster virtualizado heterogéneo.

---

## Algoritmos evaluados

Se compararon dos variantes del algoritmo clásico de multiplicación de matrices.

### 1. FxC — Filas por Columnas

Corresponde a la implementación tradicional del producto matricial. Cada elemento de la matriz resultado `C[i][j]` se calcula como el producto punto entre la fila `i` de la matriz `A` y la columna `j` de la matriz `B`.

Esta variante presenta un patrón de acceso menos eficiente para la memoria caché, debido a que la matriz `B` se recorre por columnas aunque está almacenada por filas en memoria.

### 2. FxT — Filas por Transpuesta

Esta variante transpone previamente la matriz `B` para que el producto punto se realice recorriendo dos vectores contiguos en memoria. Aunque agrega un costo adicional **O(N²)** por la transposición, mejora significativamente la localidad espacial de caché durante el cálculo principal **O(N³)**.

---

## Arquitectura de la solución

La solución utiliza una arquitectura maestro-trabajadores con MPI:

1. El proceso maestro inicializa las matrices `A`, `B` y `C`.
2. La matriz `B` se distribuye a todos los workers mediante `MPI_Bcast`.
3. La matriz `A` se divide en bandas de filas y se envía a los workers con `MPI_Send`.
4. Cada worker calcula su porción de la matriz resultado usando FxC o FxT.
5. Los resultados parciales se retornan al maestro mediante `MPI_Recv`.
6. El proceso maestro mide el tiempo total de comunicación y cómputo usando `gettimeofday()`.

---

## Plataforma utilizada

Las pruebas se ejecutaron sobre el clúster del Grupo 02, compuesto por siete máquinas virtuales con Rocky Linux 9.7 y almacenamiento compartido por NFS.

| Nodo | Rol | IP |
|---|---|---|
| cadhead02 | Central Manager | 10.43.97.146 |
| cadcliente02 | Access Point | 10.43.97.145 |
| cad02-nfs01 | Servidor NFS | 10.43.97.149 |
| cad02-w000 | Worker / Execution Point | 10.43.97.141 |
| cad02-w001 | Worker / Execution Point | 10.43.97.135 |
| cad02-w002 | Worker / Execution Point | 10.43.97.136 |
| cad02-w003 | Worker / Execution Point | 10.43.97.148 |

### Stack de software

- Sistema operativo: Rocky Linux 9.7
- Compilador: GCC con soporte OpenMP (`-fopenmp`)
- MPI: OpenMPI 4.1.6
- Sistema de archivos compartido: NFS
- Planificador: HTCondor
- Automatización de experimentos: Perl
- Análisis de resultados: Python 3, pandas, numpy y matplotlib

---

## Estructura de archivos

| Archivo | Descripción |
|---|---|
| `moduloMPI.h` | Declaraciones de funciones auxiliares, algoritmos de multiplicación, medición de tiempo y validaciones. |
| `moduloMPI.c` | Implementación de FxC, FxT, transposición, inicialización de matrices y medición de tiempos. |
| `mxmOmpMPIfxc.c` | Programa principal MPI que ejecuta la variante FxC. |
| `mxmOmpMPIfxt.c` | Programa principal MPI que ejecuta la variante FxT. |
| `Makefile` | Archivo de compilación de los ejecutables. |
| `filehost` | Archivo de hosts usado por OpenMPI. |
| `lanzador.pl` | Script que automatiza la batería de experimentos. |
| `analisis.py` | Script para procesar resultados, calcular métricas y generar gráficas. |
| `Soluciones/` | Carpeta donde se almacenan los archivos `.dat` generados por las ejecuciones. |

---

## Compilación

Para compilar los ejecutables, ubicarse en la carpeta del proyecto y ejecutar:

```bash
make
```

Esto genera los siguientes binarios:

```bash
mxmOmpMPIfxc
mxmOmpMPIfxt
```

---

## Ejecución manual

La forma general de ejecución es:

```bash
mpirun -hostfile filehost -np <procesos> ./<ejecutable> <tamaño_matriz> <hilos>
```

Ejemplo para ejecutar FxT con una matriz de tamaño 3000, 4 procesos MPI y 4 hilos OpenMP:

```bash
mpirun -hostfile filehost -np 4 ./mxmOmpMPIfxt 3000 4
```

---

## Automatización de experimentos

El script `lanzador.pl` recorre automáticamente las combinaciones de:

- Algoritmo: FxC y FxT
- Procesos MPI: 2, 3, 4 y 5
- Tamaños de matriz: 500, 1000, 2000 y 3000
- Hilos OpenMP: 1, 2 y 4
- Repeticiones: 30 por configuración válida

Para ejecutar la batería completa:

```bash
perl lanzador.pl
```

Los resultados se guardan en la carpeta:

```bash
Soluciones/
```

Cada archivo `.dat` contiene los tiempos de ejecución medidos para una configuración específica.

---

## Restricción de divisibilidad

Para que una configuración sea válida, el tamaño de la matriz `N` debe ser divisible entre el número de workers MPI, es decir:

```text
workers = procesos_MPI - 1
N % workers == 0
```

Las combinaciones que no cumplen esta condición se descartan automáticamente.

---

## Métricas de evaluación

### Tiempo de ejecución

Se mide el tiempo de pared en microsegundos con `gettimeofday()`. La medición cubre la fase de comunicación y cómputo, pero no incluye inicialización ni liberación de memoria.

### Speedup

El speedup se calcula como:

```text
S(p) = T(1) / T(p)
```

Donde `T(1)` es el tiempo con un hilo y `T(p)` es el tiempo usando `p` hilos.

### Eficiencia

La eficiencia se calcula como:

```text
E(p) = S(p) / p
```

Una eficiencia cercana a 1 indica un escalado cercano al ideal.

---

## Procesamiento de resultados

Para procesar los archivos `.dat`, calcular estadísticas y generar gráficas:

```bash
python3 analisis.py
```

El análisis incluye:

- Tiempo medio de ejecución
- Desviación estándar
- Speedup
- Eficiencia
- Comparación entre FxC y FxT
- Gráficas de tiempo medio, speedup, eficiencia y boxplots

---

## Resultados principales

Los resultados del informe muestran que:

1. **FxT supera consistentemente a FxC** en todos los tamaños evaluados.
2. La ventaja de FxT aumenta con el tamaño de la matriz, pasando aproximadamente de 22 % en `N = 500` hasta cerca de 68 % en `N = 3000`.
3. El paralelismo OpenMP escala de manera significativa únicamente cuando existe más de un worker MPI.
4. La mejor configuración global fue:

```text
Algoritmo: FxT
N: 3000
Procesos MPI: 4
Hilos OpenMP: 4
Tiempo medio: 11.063 s
```

Esta configuración logró una reducción aproximada del **96.7 %** frente al peor caso observado.

---

## Mejores configuraciones por algoritmo y tamaño

| Algoritmo | N | Mejor np | Mejor hilos | Tiempo (s) | Speedup intra-nodo |
|---|---:|---:|---:|---:|---:|
| FxC | 500 | 3 | 4 | 0.146 | 2.42x |
| FxC | 1000 | 5 | 4 | 0.542 | 2.73x |
| FxC | 2000 | 3 | 4 | 11.998 | 3.40x |
| FxC | 3000 | 3 | 4 | 44.755 | 3.48x |
| FxT | 500 | 5 | 4 | 0.077 | 2.11x |
| FxT | 1000 | 5 | 4 | 0.392 | 2.85x |
| FxT | 2000 | 3 | 4 | 5.229 | 3.06x |
| FxT | 3000 | 4 | 4 | 11.063 | 3.30x |

---

## Conclusiones

- El patrón de acceso a memoria es un factor determinante en el rendimiento de la multiplicación de matrices densas.
- La variante FxT es más eficiente que FxC porque mejora la localidad espacial de caché.
- La distribución entre nodos mediante MPI produjo mayores ganancias que el paralelismo intra-nodo con OpenMP.
- OpenMP solo resultó rentable cuando existían varios procesos MPI workers.
- Para matrices grandes, el escalado con OpenMP fue cercano al lineal en configuraciones con suficientes workers.
- Las 30 repeticiones por configuración permitieron obtener mediciones estadísticamente más confiables.

---

## Recomendaciones

1. Usar la variante **FxT** como opción preferente para multiplicación de matrices densas.
2. Evitar usar varios hilos OpenMP cuando solo existe un worker MPI.
3. Asignar el mayor número posible de workers disponibles en el clúster.
4. Para problemas más grandes, evaluar variantes por bloques o bibliotecas optimizadas como BLAS o LAPACK.
5. En experimentos futuros, medir por separado el costo de comunicación MPI y el costo de cómputo.
6. Explorar configuraciones con más slots por nodo para analizar escenarios con mayor concurrencia.

---

## Pasos para reproducir el taller

```bash
# 1. Conectarse al Access Point
ssh estudiante@10.43.97.145

# 2. Ir a la carpeta compartida
cd /nfs/condor/evalMxM_MPI/

# 3. Compilar
make

# 4. Verificar hosts
cat filehost
mpirun -hostfile filehost hostname

# 5. Ejecutar batería experimental
perl lanzador.pl

# 6. Procesar resultados
python3 analisis.py
```ß