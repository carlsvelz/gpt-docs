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

# 🧩 ETL PIPELINE — Dependencias Previas a `extract.py`


Perfecto, antes de construir **`extract.py`**, es clave entender **el orden lógico y las dependencias mínimas** del pipeline.  
Tu estructura está bien pensada para un **ETL modular**, pero no todos los archivos necesitan existir en detalle antes de `extract.py`.  
Veamos exactamente **qué necesitas tener listo y en qué orden**, para que `extract.py` pueda funcionar sin romper dependencias 👇

## 1. `config.yml` ✅ (Requisito previo)

**Propósito:** Centraliza rutas, parámetros y conexiones.  
`extract.py` lo usará para leer de qué fuente extraer datos y a qué bucket escribirlos.

📄 Ejemplo mínimo:

```yaml
# ================================================================
# ⚙️ CONFIGURACIÓN GENERAL DEL PIPELINE ETL
# ================================================================

etl:
  name: "taxi_etl_pipeline"
  mode: "local"                     # opciones: local | dev | prod
  chunksize: 500000
  log_dir: "logs/"
  temp_dir: "tmp/"

# ================================================================
# 🗃️ FUENTES DE DATOS (Nivel Bronze - Extracción)
# ================================================================
sources:
  yellow_taxi_data:
    type: "parquet"
    input_path: "data/raw/yellow_tripdata_2024-01.parquet"
    description: "Dataset público NYC Yellow Taxi Trips 2024-01"

# ================================================================
# 🪣 MINIO / LAKEHOUSE (Nivel Bronze → Silver → Gold)
# ================================================================
lakehouse:
  endpoint: "http://minio:9000"
  bucket_bronze: "bronze"
  bucket_silver: "silver"
  bucket_gold: "gold"
  secure: false

# Nota: las credenciales (MINIO_ROOT_USER y MINIO_ROOT_PASSWORD)
# se obtendrán desde el entorno del contenedor (minio.env),
# y no deben definirse aquí por seguridad.

# ================================================================
# 🧩 ESTRUCTURA DE CAPAS
# ================================================================
paths:
  bronze: "lakehouse/bronze/"
  silver: "lakehouse/silver/"
  gold: "lakehouse/gold/"

# ================================================================
# 🧠 LOGGING Y MONITOREO
# ================================================================
logging:
  level: "INFO"
  format: "%(asctime)s [%(levelname)s] %(message)s"

# ================================================================
# 🧰 CONEXIONES ADICIONALES (opcional para siguiente nivel)
# ================================================================
postgres:
  host: "postgres"
  port: 5432
  database: "warehouse"
  user: "${POSTGRES_USER}"     # variable de entorno
  password: "${POSTGRES_PASSWORD}"  # variable de entorno
  schema: "public"
```

🧠 _Este archivo debe existir primero_, porque `extract.py` tomará su configuración de aquí.

#### 🧩 Explicación rápida

|Sección|Propósito|
|---|---|
|`etl`|Parámetros globales (modo, logs, tamaño de chunk).|
|`sources`|Define los orígenes de datos (en este caso un `.parquet` local).|
|`lakehouse`|Configuración de MinIO (endpoint y buckets, **sin credenciales**).|
|`paths`|Estructura de carpetas dentro del data lake (bronze/silver/gold).|
|`logging`|Control de nivel y formato de logs.|
|`postgres`|Definido con variables de entorno — preparado para futuras cargas a SQL.|

---

#### 🧠 Cómo se conectará con `minio.env`

Tu archivo `minio.env` ya tiene:

```bash
MINIO_ROOT_USER=minio-access-key
MINIO_ROOT_PASSWORD=minio-secret-key
```

Cuando tu `extract.py` o `lakehouse.py` lean el `config.yml`, podrán acceder a las credenciales así:

```python
import os
import yaml

with open("etl_pipeline/config.yml", "r") as f:
    config = yaml.safe_load(f)

minio_conf = config["lakehouse"]

endpoint = minio_conf["endpoint"]
access_key = os.getenv("MINIO_ROOT_USER")
secret_key = os.getenv("MINIO_ROOT_PASSWORD")
```

✅ De esta manera:

- `config.yml` define **dónde conectar**.
    
- `minio.env` define **con qué credenciales**.
    
- No se expone información sensible en ningún archivo de configuración versionado.

---

## 2. `utils.py` ✅ (Necesario antes de `extract.py`)

**Propósito:** Reúne funciones auxiliares comunes (tiempo, logs, hash, etc.)  
`extract.py` usará estos helpers.

📄 Ejemplo mínimo:

```python
import os
import time
import hashlib
import logging
import yaml
from datetime import datetime
from functools import wraps


# ================================================================
# 📖 CARGA DE CONFIGURACIÓN
# ================================================================
def load_config(path="etl_pipeline/config.yml"):
    """
    Carga el archivo YAML de configuración global del pipeline.

    Args:
        path (str): Ruta al archivo config.yml.
    Returns:
        dict: Diccionario con la configuración cargada.
    """
    if not os.path.exists(path):
        raise FileNotFoundError(f"No se encontró el archivo de configuración: {path}")

    with open(path, "r") as f:
        config = yaml.safe_load(f)

    return config


# ================================================================
# 🧾 CONFIGURACIÓN DE LOGGING
# ================================================================
def setup_logger(name="etl_pipeline", log_dir="logs", level=logging.INFO, fmt=None):
    """
    Configura un logger con salida a archivo y consola.

    Args:
        name (str): Nombre del logger.
        log_dir (str): Carpeta donde se guardarán los logs.
        level: Nivel de logging.
        fmt: Formato del mensaje (opcional).
    Returns:
        logging.Logger: instancia configurada.
    """
    os.makedirs(log_dir, exist_ok=True)

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    log_file = os.path.join(log_dir, f"{name}_{timestamp}.log")

    fmt = fmt or "%(asctime)s [%(levelname)s] %(message)s"

    logging.basicConfig(
        level=level,
        format=fmt,
        handlers=[
            logging.FileHandler(log_file),
            logging.StreamHandler()
        ],
    )

    logger = logging.getLogger(name)
    logger.info(f"Logger inicializado en {log_file}")
    return logger


# ================================================================
# ⏱️ DECORADOR DE TIEMPO DE EJECUCIÓN
# ================================================================
def timer(func):
    """
    Decorador para medir el tiempo de ejecución de una función.
    """
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"⏱️ {func.__name__} ejecutado en {end - start:.2f} segundos")
        return result
    return wrapper


# ================================================================
# 🧩 HASHING DE ARCHIVOS
# ================================================================
def file_hash(path, algo="sha256", chunk_size=8192):
    """
    Calcula el hash de un archivo para control de integridad/versionado.

    Args:
        path (str): Ruta del archivo.
        algo (str): Algoritmo de hash (sha256, md5, etc.).
        chunk_size (int): Tamaño de bloque en bytes.
    Returns:
        str: Hash hexadecimal.
    """
    h = hashlib.new(algo)
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(chunk_size), b""):
            h.update(chunk)
    return h.hexdigest()


# ================================================================
# 📁 GESTIÓN DE DIRECTORIOS
# ================================================================
def ensure_dir(path):
    """
    Crea un directorio si no existe.
    """
    os.makedirs(path, exist_ok=True)


# ================================================================
# 🧠 UTILIDAD DE TIMESTAMP
# ================================================================
def timestamp(fmt="%Y%m%d_%H%M%S"):
    """
    Devuelve un timestamp con formato estándar.
    """
    return datetime.now().strftime(fmt)


# ================================================================
# 🧮 FORMATEO DE BYTES (para métricas de memoria)
# ================================================================
def format_bytes(size):
    """
    Convierte bytes a un formato legible (KB, MB, GB).
    """
    for unit in ["B", "KB", "MB", "GB", "TB"]:
        if size < 1024:
            return f"{size:.2f} {unit}"
        size /= 1024


# ================================================================
# ✅ EJEMPLO DE USO DIRECTO (para debug local)
# ================================================================
if __name__ == "__main__":
    cfg = load_config()
    logger = setup_logger("test_utils")

    logger.info("Archivo de configuración cargado correctamente.")
    logger.info(f"Config: {cfg['lakehouse']['endpoint']}")
    logger.info(f"Hash de utils.py: {file_hash(__file__)}")
```

#### 🧩 ¿Qué cubre este módulo?

|Función|Propósito|
|---|---|
|`load_config()`|Carga el archivo YAML global (`config.yml`).|
|`setup_logger()`|Inicializa un logger con archivo + consola.|
|`timer()`|Decorador para medir tiempos de funciones.|
|`file_hash()`|Genera hash SHA256 de archivos para trazabilidad.|
|`ensure_dir()`|Crea carpetas si no existen (logs, temp, etc).|
|`timestamp()`|Genera timestamps estándar para versionado.|
|`format_bytes()`|Convierte bytes en unidades legibles (para métricas de RAM).|

---

#### 🔗 Cómo se integrará más adelante

- `extract.py` → usará `setup_logger`, `load_config`, `timestamp`, `file_hash`.
    
- `lakehouse.py` → usará `load_config` y `ensure_dir`.
    
- `validate.py` y `load.py` → reutilizarán los mismos logs y paths.
    
- `main.py` → será el punto donde inicialices el logger global y pases el objeto `logger` a todos los módulos.

---

## 3. `lakehouse.py` ✅ (Necesario antes de `extract.py`)

**Propósito:** Gestiona la conexión y operaciones con MinIO/S3 (el “Lakehouse”).  
`extract.py` lo usará para **guardar los archivos extraídos** al bucket _bronze_.

📄 Ejemplo mínimo:

```python
import os
import io
import logging
from minio import Minio
from minio.error import S3Error
from utils import load_config, timestamp


class Lakehouse:
    """
    Clase que abstrae la conexión y operaciones básicas con MinIO (S3-compatible).
    Maneja lectura, escritura, creación de buckets y verificación de objetos.
    """

    def __init__(self, config_path="etl_pipeline/config.yml", logger=None):
        self.config = load_config(config_path)
        self.logger = logger or logging.getLogger(__name__)

        lakehouse_cfg = self.config["lakehouse"]

        # Variables sensibles cargadas desde entorno (no YAML)
        self.endpoint = lakehouse_cfg["endpoint"]
        self.access_key = os.getenv("MINIO_ROOT_USER")
        self.secret_key = os.getenv("MINIO_ROOT_PASSWORD")
        self.secure = lakehouse_cfg.get("secure", False)

        # Inicialización del cliente
        self.client = Minio(
            self.endpoint,
            access_key=self.access_key,
            secret_key=self.secret_key,
            secure=self.secure
        )

        self.logger.info(f"✅ Conectado a MinIO en {self.endpoint}")

        # Verifica y crea buckets principales
        self._ensure_buckets()

    # ===============================================================
    # 🪣 Verificación de buckets
    # ===============================================================
    def _ensure_buckets(self):
        """
        Crea los buckets definidos en la configuración si no existen.
        """
        buckets = self.config["lakehouse"]["buckets"]
        for layer, name in buckets.items():
            bucket_name = name.split("/")[0]  # "lakehouse/bronze/" → "lakehouse"
            if not self.client.bucket_exists(bucket_name):
                self.client.make_bucket(bucket_name)
                self.logger.info(f"🪣 Bucket creado: {bucket_name}")
            else:
                self.logger.debug(f"Bucket ya existe: {bucket_name}")

    # ===============================================================
    # 📤 Subir archivo a una capa
    # ===============================================================
    def upload_file(self, file_path, layer="bronze", object_name=None):
        """
        Sube un archivo local a la capa especificada del Lakehouse.

        Args:
            file_path (str): Ruta local del archivo.
            layer (str): Capa destino ("bronze", "silver", "gold").
            object_name (str): Nombre del objeto (opcional).
        """
        bucket = self.config["lakehouse"]["buckets"][layer].split("/")[0]
        prefix = f"{layer}/"

        if object_name is None:
            filename = os.path.basename(file_path)
            ts = timestamp()
            object_name = f"{prefix}{ts}_{filename}"

        try:
            self.client.fput_object(bucket, object_name, file_path)
            self.logger.info(f"📤 Subido a {bucket}/{object_name}")
            return f"s3://{bucket}/{object_name}"
        except S3Error as e:
            self.logger.error(f"❌ Error al subir archivo a MinIO: {e}")
            raise

    # ===============================================================
    # 📥 Descargar archivo
    # ===============================================================
    def download_file(self, object_path, dest_path):
        """
        Descarga un objeto desde MinIO.

        Args:
            object_path (str): Ruta tipo "bucket/object".
            dest_path (str): Ruta local destino.
        """
        bucket, obj = object_path.split("/", 1)
        self.client.fget_object(bucket, obj, dest_path)
        self.logger.info(f"📥 Archivo descargado: {dest_path}")

    # ===============================================================
    # 📜 Listar objetos
    # ===============================================================
    def list_objects(self, layer="bronze"):
        """
        Lista los objetos de una capa del Lakehouse.
        """
        bucket = self.config["lakehouse"]["buckets"][layer].split("/")[0]
        prefix = f"{layer}/"

        objects = self.client.list_objects(bucket, prefix=prefix, recursive=True)
        return [obj.object_name for obj in objects]

    # ===============================================================
    # 🧪 Guardar contenido en memoria (útil para DataFrames Parquet)
    # ===============================================================
    def upload_bytes(self, data_bytes, layer="silver", object_name="data.parquet"):
        """
        Sube un archivo generado en memoria (como un DataFrame en Parquet).
        """
        bucket = self.config["lakehouse"]["buckets"][layer].split("/")[0]
        prefix = f"{layer}/"
        ts = timestamp()
        object_name = f"{prefix}{ts}_{object_name}"

        try:
            self.client.put_object(
                bucket,
                object_name,
                io.BytesIO(data_bytes),
                length=len(data_bytes),
                content_type="application/octet-stream"
            )
            self.logger.info(f"📤 Subido en memoria a {bucket}/{object_name}")
            return f"s3://{bucket}/{object_name}"
        except S3Error as e:
            self.logger.error(f"❌ Error al subir bytes a MinIO: {e}")
            raise


# ===============================================================
# 🧩 Ejemplo de uso
# ===============================================================
if __name__ == "__main__":
    from utils import setup_logger

    logger = setup_logger("lakehouse_test")
    lh = Lakehouse(logger=logger)

    # Ejemplo: listar objetos en Bronze
    print(lh.list_objects("bronze"))
```

🧠 Este script te permitirá a futuro crear buckets para **bronze/silver/gold** y guardar data procesada.

#### 🧠 Explicación de diseño

|Componente|Función|
|---|---|
|`Lakehouse.__init__`|Carga configuración, credenciales del entorno y crea el cliente MinIO.|
|`_ensure_buckets()`|Crea automáticamente los buckets definidos (`bronze`, `silver`, `gold`).|
|`upload_file()`|Sube archivos locales a una capa (por ejemplo, el parquet descargado).|
|`upload_bytes()`|Sube datos generados en memoria (ideal para transformaciones con Pandas/Polars).|
|`download_file()`|Descarga objetos de MinIO a local.|
|`list_objects()`|Lista los objetos por capa.|

---

#### 🔗 Relación con el resto del pipeline

|Archivo|Cómo usa `lakehouse.py`|
|---|---|
|`extract.py`|Llama a `Lakehouse.upload_file()` para guardar el archivo crudo en **Bronze**.|
|`transform.py`|Usa `upload_bytes()` para escribir transformaciones en **Silver**.|
|`load.py`|Puede escribir en **Gold** y cargar a PostgreSQL.|
|`main.py`|Inicializa el objeto `Lakehouse` y lo pasa a las etapas.|

---

#### ✅ Próximo paso

Con esto, ya tenemos la base para que `extract.py` pueda:

1. Descargar el parquet original.
    
2. Verificar integridad / métricas.
    
3. Subirlo a la capa **Bronze** en MinIO con timestamp.

---

## 4. `extract.py` 🚧 (puede crearse ahora)

Una vez tengas los tres anteriores (`config.yml`, `utils.py`, `lakehouse.py`), Ahora sí, podemos construir **`extract.py`**, el **primer script real del pipeline ETL Lakehouse**.

Este script marcará el inicio del flujo **Extract → Transform → Load**, y su propósito es muy claro:

> **Extraer datos crudos (raw), almacenarlos sin modificaciones en la capa Bronze del Lakehouse (MinIO) y registrar sus métricas.**

#### 🧱 Diseño conceptual — “Capa Bronze”

|Etapa|Acción|Resultado|
|---|---|---|
|🔽 **Descarga**|Descargar archivo fuente (Parquet remoto o local)|`data/raw/yellow_tripdata_2024-01.parquet`|
|🧩 **Registro**|Medir tiempo, memoria y tamaño|Logs + métricas|
|🪣 **Carga a Bronze**|Subir a MinIO (`lakehouse/bronze/<timestamp>_...`)|Capa Bronze versionada|

### ✅ `etl_pipeline/extract.py`

```python
import os
import time
import psutil
import requests
from utils import setup_logger, load_config, ensure_dir, file_hash, timestamp, format_bytes
from lakehouse import Lakehouse


def download_source_file(url, output_dir="data/raw", logger=None):
    """
    Descarga un archivo remoto (por ejemplo, Parquet) y lo almacena localmente.

    Args:
        url (str): URL de origen.
        output_dir (str): Carpeta destino local.
        logger (logging.Logger): Logger para registrar eventos.
    Returns:
        str: Ruta local del archivo descargado.
    """
    ensure_dir(output_dir)
    local_filename = os.path.join(output_dir, url.split("/")[-1])
    logger.info(f"⬇️ Descargando archivo desde {url}")

    start_time = time.perf_counter()
    response = requests.get(url, stream=True)
    response.raise_for_status()

    with open(local_filename, "wb") as f:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)

    elapsed = time.perf_counter() - start_time
    size = os.path.getsize(local_filename)
    logger.info(f"✅ Archivo descargado: {local_filename} ({format_bytes(size)} en {elapsed:.2f}s)")
    return local_filename


def collect_metrics(file_path):
    """
    Genera métricas básicas del archivo extraído.
    """
    size = os.path.getsize(file_path)
    hash_value = file_hash(file_path)
    process = psutil.Process(os.getpid())
    mem_usage = process.memory_info().rss

    return {
        "file_name": os.path.basename(file_path),
        "size_bytes": size,
        "hash": hash_value,
        "memory_used": format_bytes(mem_usage),
        "timestamp": timestamp(),
    }


def extract_to_bronze():
    """
    Orquesta el proceso de extracción:
      1️⃣ Descarga archivo remoto.
      2️⃣ Mide métricas.
      3️⃣ Sube a MinIO (Bronze).
    """
    logger = setup_logger("extract")
    config = load_config()
    lakehouse = Lakehouse(logger=logger)

    url = config["etl"]["source_url"]
    local_file = download_source_file(url, logger=logger)

    # Recolección de métricas
    metrics = collect_metrics(local_file)
    logger.info(f"📊 Métricas de extracción: {metrics}")

    # Subida a la capa Bronze
    s3_uri = lakehouse.upload_file(local_file, layer="bronze")
    logger.info(f"🪣 Archivo cargado a Bronze: {s3_uri}")

    logger.info("✅ Extracción completada correctamente.")


if __name__ == "__main__":
    extract_to_bronze()
```


#### 🧠 Explicación por bloques

|Bloque|Propósito|
|---|---|
|`download_source_file()`|Descarga el dataset original (puede ser `.parquet`, `.csv`, `.gz`, etc.) y guarda en `data/raw/`.|
|`collect_metrics()`|Calcula tamaño, hash y uso de memoria para trazabilidad.|
|`extract_to_bronze()`|Es el **entrypoint**: descarga, mide y sube a MinIO.|
|`Lakehouse.upload_file()`|Guarda en MinIO bajo `lakehouse/bronze/<timestamp>_...`|
|`logger` + `metrics`|Cada paso genera trazas (logs) y métricas para observabilidad.|
#### 📁 Flujo esperado de archivos

Después de ejecutar:

```bash
python etl_pipeline/extract.py
```

Tendrás algo como:

```
etl_pipeline/
├── logs/
│   └── extract_20251111_113045.log
├── data/
│   └── raw/
│       └── yellow_tripdata_2024-01.parquet
├── lakehouse/
│   └── bronze/
│       └── 20251111_113046_yellow_tripdata_2024-01.parquet  (en MinIO)
```

---

### 5. (Opcional todavía)

- `transform.py`, `validate.py` y `load.py` pueden venir después.
    
- `main.py` será el orquestador final, pero puede crearse una vez `extract` funcione.
    
---

### 🧭 Resumen de orden de construcción

|Orden|Script|Estado|Necesario para|
|:--|:--|:--|:--|
|1️⃣|`config.yml`|✅ Crear primero|Todos los módulos|
|2️⃣|`utils.py`|✅ Crear primero|Logs, métricas|
|3️⃣|`lakehouse.py`|✅ Crear primero|Guardar data raw en MinIO|
|4️⃣|`extract.py`|🚧 Próximo paso|Extraer data a Bronze|
|5️⃣|`transform.py`|⏳ Posterior|Transformaciones Silver|
|6️⃣|`validate.py`|⏳ Posterior|Checks de calidad|
|7️⃣|`load.py`|⏳ Posterior|Carga a PostgreSQL/Parquet|
|8️⃣|`main.py`|🔚 Final|Orquestador|

---

