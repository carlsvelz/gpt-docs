# 🚀 Bienvenido al Nivel 1 del Roadmap Lakehouse

**El primer paso real hacia dominar la ingeniería de datos moderna.**

Dejamos atrás los ejemplos triviales: a partir de ahora **pensarás y trabajarás como un Data Engineer real**.  
Aprenderás a usar **Python como el motor central del proceso ETL**, procesando datos masivos con propósito, eficiencia y mentalidad de producción.

Cada módulo, función y script tendrá un objetivo claro dentro del flujo **Extract → Transform → Load**, y se integrará dentro del modelo **Lakehouse (Bronze → Silver → Gold)** — la arquitectura base de las plataformas modernas de datos.

---

## 🧩 Nivel 1 — ETL Básico con Python + MinIO

**“De scripts aislados a un pipeline estructurado en el Lakehouse.”**

---

### 💡 Introducción

Aquí comienza tu entrenamiento práctico 💪:  
aprenderás a **diseñar y construir un pipeline ETL real con Python**, conectado a **MinIO como Data Lake local**, y estructurado bajo el modelo **Bronze / Silver / Gold**.

Esta es la misma arquitectura utilizada por compañías como Databricks, Snowflake o Uber — pero adaptada a un entorno controlado, local y 100 % reproducible.

---

### 🎯 Objetivo del Nivel 1

> Construir un **pipeline ETL completo con Python**, utilizando datos reales (NYC Taxi), almacenados en **MinIO** a través de las capas **Bronze, Silver y Gold**, con control de memoria, manejo eficiente de datos y almacenamiento columnar (Parquet).

Este nivel desarrolla tu **mentalidad de ingeniería de datos**: crear pipelines **modulares, trazables y configurables**, antes de escalar hacia sistemas distribuidos.

---

### ⚙️ Enfoque general

Aprenderás a:

1. **Diseñar un pipeline ETL por capas (Bronze → Silver → Gold)**.
    
2. **Procesar datasets grandes (≥ 1 GB)** enfrentando límites reales de hardware.
    
3. **Optimizar rendimiento y memoria** con herramientas nativas de Python (`csv`, `gzip`, `pyarrow`, `multiprocessing`, etc.).
    
4. **Persistir resultados** en **MinIO (Parquet)** y **PostgreSQL**.
    
5. **Monitorear, versionar y medir** tu pipeline como en un entorno productivo.
    

---

## 🔨 Estructura del Lab — “ETL Lakehouse con Python”

### 🌰 Capa Bronze — Raw Zone (Extract)

- Fuente: archivos `yellow_taxi_data_*.csv` ubicados en tu **bucket `lakehouse/bronze/`** en MinIO o desde un origen público.
    
- Extracción con librerías estándar (`csv`, `requests`, `gzip`) o herramientas ETL en Python.
    
- **Medición de rendimiento y memoria** con `perf_counter` (tiempos) y `psutil` o `memory_profiler` (RAM).

👉 **Objetivo:** extraer y almacenar los datos crudos sin alterarlos en la capa _Bronze_ del Lakehouse.

---

### ⚙️ Capa Silver — Clean Zone (Transform + Validate)

Aplicarás transformaciones y validaciones para limpiar y estandarizar los datos antes de modelarlos.  
Aquí puedes usar **Pandas**, **Polars** o funciones nativas de Python según el tamaño del dataset.

#### 🧩 Transformaciones básicas

- Conversión de timestamps → `datetime`.
    
- Cálculo de duración del viaje (minutos).
    
- Filtrado de outliers (viajes negativos, distancias > 100 km).
    
- Creación de columnas derivadas (`fare_per_km`, `is_airport_trip`).
    
- Agrupaciones y agregaciones simples.
    
#### 🧪 Validación de calidad

- Comprobación de esquema y tipos.
    
- Detección de nulos, duplicados o valores fuera de rango.
    
- Validación de reglas de negocio (`fare_amount > 0`).
    
- Detección de anomalías (`describe()`, IQR, z-score).

#### ⚡ Medición de rendimiento y memoria

Así como en la extracción, esta capa también debe medirse:

- Usa `perf_counter` para identificar cuellos de botella en las transformaciones.
    
- Emplea `psutil` o `memory_profiler` para revisar el uso de RAM durante operaciones intensivas como `merge` o `groupby`.


👉 **Objetivo:** generar una versión **depurada, estructurada y lista para análisis**.

---

### 🏆 Capa Gold — Curated Zone (Load)

Con los datos ya limpios, crearás tablas analíticas y datasets curados.

**Ejercicios:**

- Generar agregados por zona, hora o tipo de pago.
    
- Exportar resultados a **`lakehouse/gold/`** (Parquet).
    
- Cargar la tabla final `fact_trips` en **PostgreSQL**.
    
- Comparar tamaños y tiempos frente a CSV.
    

👉 **Objetivo:** construir la **capa de datos modelada y lista para consumo analítico.**

---

### 🧰 Configuración del pipeline (Config-driven)

Gestiona parámetros y rutas desde un `config.yml`:

```yaml
lakehouse:
  endpoint: "http://localhost:9000"
  access_key: "minio"
  secret_key: "minio123"
  buckets:
    bronze: "lakehouse/bronze/"
    silver: "lakehouse/silver/"
    gold: "lakehouse/gold/"
etl:
  chunksize: 500000
  output_format: "parquet"
  validation: true
```

👉 **Objetivo:** crear un pipeline **parametrizable y reutilizable**, sin hardcoding.

---

### 🧩 Modularización del código

Estructura tu proyecto siguiendo una arquitectura limpia:

```
etl_pipeline/
├── extract.py
├── transform.py
├── validate.py
├── load.py
├── lakehouse.py      # gestión de MinIO y conexión S3
├── utils.py
└── main.py           # orquestador del flujo ETL
```

👉 **Objetivo:** crear código **limpio, desacoplado y mantenible**, propio de un entorno profesional.

---

### 💾 Versionamiento y control de datos

Cada ejecución del pipeline:

- Genera una versión en cada capa (`bronze`, `silver`, `gold`) con timestamp.
    
- Registra metadatos (fecha, filas, tamaño, hash) en `lakehouse/_metadata.csv`.
    

👉 **Objetivo:** aplicar **data lineage y trazabilidad** entre las capas del Lakehouse.

---

### ⚡ Benchmarking comparativo

Evalúa distintos enfoques técnicos:

|Escenario|Tiempo (s)|Memoria (MB)|Tamaño (MB)|
|---|---|---|---|
|Bronze (CSV)|220|1400|900|
|Silver (Parquet)|95|680|180|
|Gold (Agregado)|45|500|60|

👉 **Objetivo:** medir el impacto de tus decisiones técnicas a lo largo del pipeline.

---

### 🔍 Observabilidad básica

- Logging con `logging` o `structlog`.
    
- Barra de progreso con `tqdm`.
    
- Monitoreo de consumo de CPU/RAM (`psutil`).
    
- Exportación de métricas de ejecución (tiempo, tamaño, versión).
    

👉 **Objetivo:** aprender a **observar y diagnosticar** tu pipeline como un sistema vivo.

---

### ⚙️ Bonus avanzado

- **Parallel processing** con `ThreadPoolExecutor` o `multiprocessing`.
    
- **Checkpointing** por capa (reanudar desde Silver).
    
- **Data sampling** para pruebas rápidas.
    

👉 **Objetivo:** incorporar **resiliencia y eficiencia** desde el inicio.

---

## 🧠 Conceptos introducidos

|Concepto|Descripción|
|---|---|
|**Lakehouse architecture**|Capas Bronze / Silver / Gold|
|**ETL modular en Python**|Diseño por funciones y módulos|
|**Chunked processing**|Procesamiento por bloques controlados|
|**Data validation**|Control de integridad y negocio|
|**Config-driven pipeline**|Separar configuración del código|
|**Data lineage**|Versionado y trazabilidad entre capas|
|**Benchmarking**|Evaluación de rendimiento|
|**MinIO integration**|Simulación de un Data Lake S3-compatible|

---

## 🧪 Ejercicio Final — `etl_nyc_lakehouse.py`

> Construye tu **pipeline ETL Lakehouse completo con Python y MinIO.**

Tu script deberá:

1. Extraer datos crudos → **Bronze**.
    
2. Transformar y validar → **Silver**.
    
3. Agregar y publicar → **Gold**.
    
4. Cargar la capa Gold a **PostgreSQL**.
    
5. Registrar métricas y versiones.
    

🎯 **Resultado esperado:** un pipeline **estructurado, trazable y reproducible**, aplicando buenas prácticas de ingeniería de datos con Python.

---

## 📊 Evaluación del Nivel

|Criterio|Peso|Logrado|
|---|---|---|
|Pipeline por capas (Bronze/Silver/Gold)|25 %|☐|
|Transformaciones y validaciones reales|20 %|☐|
|Carga en MinIO y PostgreSQL|15 %|☐|
|Optimización y benchmarking|15 %|☐|
|Observabilidad y métricas|10 %|☐|
|Modularización y configuración externa|10 %|☐|
|Versionamiento y lineage|5 %|☐|

---

## 🚀 Transición al Nivel 2 — ETL Optimizado con Polars

> Al finalizar este nivel, habrás construido un **pipeline ETL real con Python**, basado en el modelo Lakehouse y respaldado por MinIO.  
> En el **Nivel 2**, escalarás este diseño con **Polars**, **paralelismo nativo** y **rendimiento extremo**.

---


# 🧱 Comenzando el Nivel 1 — Construyendo un ETL real con Python

En el **Nivel 0** ya descubriste las bases del procesamiento de datos en Python:  
aprendiste a trabajar con grandes volúmenes de información, a manipularlos con eficiencia (`FULL LOAD` vs `chunksize`) y a manejar la memoria como lo haría un ingeniero de datos.

Ahora vamos un paso más allá.  
En este **Nivel 1**, transformarás ese conocimiento en un **pipeline ETL completo y estructurado**, aplicando principios reales de ingeniería de datos — y usando **Pandas como motor de ejecución**, no como el objetivo principal.

---

## 🎯 De “procesar datos” a “construir pipelines”

Hasta ahora, el foco estuvo en cargar y explorar datos dentro de Python.  
A partir de este punto, aprenderás a **diseñar, modularizar y ejecutar un flujo ETL completo**:

> **Extract → Transform → Load**

Cada etapa se construye con propósito, control de recursos, validaciones y configuraciones externas, tal como se haría en un entorno productivo.

---

## 🔍 ¿Qué cambia con respecto al Nivel 0?

|Nivel 0 (Fundamentos de Python)|Nivel 1 (Ingeniería de datos con Python)|
|---|---|
|Ejecución manual en notebooks|Pipeline modular y automatizado|
|Procesamiento secuencial simple|Extracción y carga optimizadas|
|Sin reglas de negocio|Transformaciones tipadas y validadas|
|Sin control de calidad|Métricas y validaciones automáticas|
|Scripts rígidos|Código configurable y parametrizable|
|Resultados temporales|Persistencia estructurada (Parquet / SQL)|

---

## 🧩 Estructura del Lab — “ETL con Python”

El laboratorio de este nivel te guiará por las tres etapas principales de un flujo profesional:

### 1️⃣ Extracción (Extract)

- Fuente: archivos `yellow_taxi_data_*.csv` desde un bucket público a tu MinIO local.
    
- Lectura por **bloques (`chunksize`)** para simular _streaming batch_.
    
- **Medición de rendimiento y memoria** con `perf_counter` (tiempos) y `psutil` o `memory_profiler` (RAM).
    
- Implementación de logs básicos (`logging`) para registrar progreso y eventos.
    

👉 **Objetivo:** comprender que **la extracción es una fase crítica del ETL**, donde el control de recursos, la limpieza inicial y el registro de métricas determinan la estabilidad del pipeline.

---

### 🧩 Modularización aplicada a la Extracción

A diferencia del Nivel 0 —donde escribías código en un solo notebook— aquí comenzarás a pensar **como un ingeniero de datos en Python**, dividiendo responsabilidades por módulo.

Tu estructura base será:

```
etl_pipeline/
├── extract.py       # Lógica de lectura y extracción (chunksize, logs, métricas)
├── transform.py     # Transformaciones del dataset
├── validate.py      # Validaciones y control de calidad
├── load.py          # Carga a PostgreSQL o Parquet
├── utils.py         # Funciones comunes (rutas, tiempos, logs)
└── main.py          # Orquestador general del pipeline
```

> En esta primera etapa trabajaremos principalmente en **extract.py** y **main.py**, dejando el resto preparado para extenderlo más adelante.

👉 **Objetivo:** construir código **modular, reutilizable y legible**, donde cada parte tenga una función clara dentro del pipeline.

---

## ⚙️ Diseño con configuración externa

Evita valores fijos en tu código.  
A partir de este nivel, aprenderás a **desacoplar la configuración** del pipeline mediante un archivo `config.yml`:

```yaml
etl:
  chunksize: 500000
  input_path: "lakehouse/bronze/"
  output_path: "lakehouse/silver/"
  output_format: "parquet"

lakehouse:
  endpoint: "http://localhost:9000"
  access_key: "minio"
  secret_key: "minio123"
```

👉 **Objetivo:** convertir tu pipeline Python en un sistema **parametrizable y portable**, fácilmente reutilizable por otros entornos o datasets.

---

## 🧠 Antes de continuar

Antes de avanzar hacia la **Transformación**, asegúrate de:

- Haber probado correctamente la lectura incremental (`chunksize`).
    
- Medir y registrar tiempos y consumo de memoria.
    
- Confirmar que los tipos de columnas y nombres se mantienen consistentes.
    
- Contar con un sistema de logging que capture cada ejecución.
    
- Tener tu estructura modular operativa: `main.py` debe orquestar la extracción.
    

---

> 💬 **Reflexión:**  
> Python es hoy el lenguaje central de la ingeniería de datos moderna.  
> Librerías como Pandas, Polars, PyArrow o Dask son solo motores:  
> lo importante es **entender los principios de diseño, modularización, validación y trazabilidad**.
> 
> Con este nivel, estás aprendiendo a pensar como un ingeniero de datos que domina **Python como plataforma de orquestación**, no como una simple herramienta de análisis.

---

