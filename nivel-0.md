# 🧠 Data Modeling Nivel-0

Objetivo: Aprender modelado relacional y fundamentos de SQL usando NYC Yellow Taxi Trips como caso práctico.

Contexto

- Los NYC Yellow Taxi Trips son dataset público con registros de viajes de taxis amarillos de la ciudad de Nueva York, gestionado por la Taxi and Limousine Commission (TLC). Incluyen origen y destino, fechas y horas, distancia, tarifas, pago y otros atributos que permiten analizar movilidad, demanda y desempeño económico.

- Este bloque es clave porque aborda la relación entre el diseño lógico del dataset y la forma en que se carga y persiste, conectando con el Módulo 1: Data Modeling (SQL + Python) y con el Nivel 0–1 del Roadmap.

Propuesta de encaje en el Nivel-0

- El NYC Taxi sirve como punto de partida ideal para integrar la Capa de Modelado de Datos dentro del Nivel 0 del Roadmap.

- Zweck: comprender cómo pasa un dataset transaccional a un modelo analítico y preparar la base para el siguiente nivel (OLAP).

| Nivel                                                          | Foco                           | Objetivo del módulo                                                                                                                                                         | Herramientas principales                                         | Se conecta con                        |
| -------------------------------------------------------------- | ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------- |
| 🧱 **Módulo 1: Tablas simples & Data Modeling (SQL + Python)** | **Modelado relacional básico** | Comprender claves primarias, foráneas, normalización, Star Schema, índices y consultas SQL + desde Python. Usar SQLite para labs y PostgreSQL para conceptos de producción. | SQLite (labs) / PostgreSQL (prod) + Pandas y Polars + SQLAlchemy | **Nivel 0-1 (ETL básico con Python)** |

> Objetivo General

> Comprender los fundamentos del modelado relacional y aplicar SQL para explorar, estructurar y documentar datos reales usando el dataset NYC Taxi. Este módulo es la base de toda la arquitectura de datos: aquí nacen las primeras decisiones de diseño que luego evolucionan hacia modelos analíticos (OLAP).

## 🚕 **Contexto del Caso Práctico: NYC Yellow Taxi Data**

- El dataset público contiene millones de registros con información detallada sobre cada trayecto: origen, destino, distancia, tarifa, tipo de pago, proveedor, entre otros.

- Con este dataset podremos recorrer el ciclo completo del modelado de datos: de un sistema transaccional (OLTP) simulado hasta un modelo analítico (OLAP).

### Fase 1 – Fuente transaccional (datos OLTP-like)

- El dataset de viajes funciona como una fuente transaccional: cada fila es un viaje con atributos necesarios para registrar la transacción.

- No construiremos un sistema OLTP real. Usaremos estos datos como punto de partida (extracción desde Parquet) para luego procesarlos. Realizaremos staging y preparación: validación, limpieza, normalización de identificadores y enriquecimiento de atributos.

- Resultado: un modelo relacional base que simula el origen para alimentar la transformación hacia OLAP (fact table + dimensiones) y el diseño de esquemas (Star, Snowflake, OBT).

### Fase 2 – OLAP (modelo analítico para BI)

**Preguntas típicas:**

- ¿Qué zonas generan más viajes y recaudación?

- ¿Cómo varían las tarifas según proveedor u hora?

- ¿Qué métodos de pago son más usados por temporada?

- Estas preguntas requieren consultas agregadas, comparaciones históricas y análisis multidimensional.

### Fase 3 – Modelado Dimensional (Star Schema, Snowflake, OBT)

- Transformar el modelo OLTP en un modelo dimensional:

- La tabla de viajes (fact_trips) se convierte en la tabla de hechos con métricas (distancia, monto, pasajeros).

- Dimensiones como vendor, zone y payment aportan contexto descriptivo.

- Opciones de diseño:

- Star Schema, Snowflake Schema o One Big Table (desnormalizada para exploración rápida).

## 🧮 **Dataset Base**

Usaremos el dataset público de **NYC Yellow Taxi Trips**, disponible en:  
👉 [https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

Ejemplo de columnas principales:

|Columna|Tipo|Descripción|
|---|---|---|
|`tpep_pickup_datetime`|timestamp|Fecha/hora inicio del viaje|
|`tpep_dropoff_datetime`|timestamp|Fecha/hora fin del viaje|
|`passenger_count`|int|Número de pasajeros|
|`trip_distance`|float|Distancia del viaje (millas)|
|`PULocationID`|int|ID zona de origen|
|`DOLocationID`|int|ID zona de destino|
|`fare_amount`|float|Tarifa base|
|`total_amount`|float|Total final|
|`payment_type`|int|Tipo de pago|
|`VendorID`|int|Identificador del proveedor|

---

## 📐 **Diagrama ERD Simple**

>En este punto no creamos aún un modelo estrella completo, sino un **modelo relacional base (OLTP-like)**.


```mermaid

erDiagram

trip {

int trip_id PK

timestamp pickup_datetime

timestamp dropoff_datetime

float trip_distance

float total_amount

int passenger_count

int VendorID FK

int PULocationID FK

int DOLocationID FK

int payment_type FK

}

vendor {

int VendorID PK

string vendor_name

}

zone {

int LocationID PK

string borough

string zone_name

}

trip ||--o{ vendor : "belongs to"

trip ||--o{ zone : "pickup zone"

trip ||--o{ zone : "dropoff zone"

```
🧠 **Interpretación:**

- `trip` es una tabla de hechos _transaccional_ (cada fila = 1 viaje).
    
- `vendor` y `zone` actúan como _tablas de referencia_ (dimensionales en potencia).
    
- Este ERD será el punto de partida para evolucionar a un **Star Schema** en el Nivel 2.


## 🧩 **Evolución hacia un Modelo Estrella (conceptual)**

>Antes de entrar en Polars o DuckDB (Niveles 2–3), introducimos el concepto de **modelo analítico**:

```mermaid

erDiagram

fact_trips {

int trip_id PK

float trip_distance

float total_amount

int passenger_count

int pickup_zone_id FK

int dropoff_zone_id FK

int vendor_id FK

int payment_type_id FK

date trip_date

}

dim_zone {

int zone_id PK

string borough

string zone_name

}

dim_vendor {

int vendor_id PK

string vendor_name

}

dim_payment {

int payment_type_id PK

string payment_description

}

fact_trips }o--|| dim_zone : "pickup/dropoff"

fact_trips }o--|| dim_vendor : "provider"

fact_trips }o--|| dim_payment : "payment"

```

📊 **Comentarios:**

- Este mini Star Schema es coherente con NYC Taxi. La tabla trips se convierte en la fact table; vendor, zone y payment funcionan como dimensiones.

- Se puede implementar localmente con SQLite, DuckDB o Polars.

Keys (aprendizaje clave)

- Comprender la evolución de un dataset transaccional a un modelo analítico.

- Identificar llaves, relaciones y el beneficio de un esquema dimensional para BI.

Checklist de laboratorio (resumen)

- Crear tablas base con PK/FK en SQLite o PostgreSQL.

- Cargar un subconjunto de NYC Taxi (muestra).

- Realizar consultas de agregación (ingresos por zona, viajes por hora).

- Diseñar el esquema dimensional y validar con 2–3 consultas.

Notas finales

- Este Nivel-0 sienta las bases para entender la transición de OLTP a OLAP y la importancia de un diseño adecuado desde el inicio.

- Sirve de puente conceptual hacia los siguientes niveles donde se introducen Polars, DuckDB, DBT y tablas distribuidas.





# 🐳 Docker + PostgreSQL + pgAdmin 
---

## **Conexión entre pgAdmin y PostgreSQL**

🎯 **Objetivo:**  
Tener una **interfaz gráfica (pgAdmin)** conectada a un **servidor PostgreSQL**.

---

### **¿Qué puedes hacer con pgAdmin?**

➡️ Administrar bases de datos  
➡️ Crear tablas, vistas y esquemas  
➡️ Ejecutar consultas SQL  
➡️ Gestionar usuarios y roles  
➡️ Ver estadísticas y rendimiento

---

#### 📝📌 **NOTA**

> Esta configuración es ideal para Data Modeling

---

## **Docker Network — Conecta tus contenedores**

Para que PostgreSQL y pgAdmin se comuniquen, deben estar en la **misma red Docker**.

---

### **Crear la red Docker**

```bash
docker network create pg-network
```

---

### **Levantar el contenedor PostgreSQL**

```bash
docker run -it \
  --name postgres-container \
  --network pg-network \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/nyc-tlc-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```

---

#### ✔️ Resumen del contenedor PostgreSQL

|Parámetro|Explicación|
|---|---|
|`--name`|Nombre del contenedor|
|`--network pg-network`|Conecta el contenedor a la red creada|
|`-v`|Persistencia de datos|
|`-p`|Publica el puerto PostgreSQL|

---

### **Levantar el contenedor pgAdmin**

```bash
docker run -it \
  --name pgadmin-container \
  --network pg-network \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 5050:80 \
  dpage/pgadmin4
```

---

## **Configuración dentro de pgAdmin**

### **Crear servidor**

Ruta: **Servers → Register → Server**

---

### **Pestaña General**

- Name: `Postgres Docker`
    

---

### **Pestaña Connection**

|Campo|Valor|
|---|---|
|Hostname/address|`postgres-container`|
|Port|`5432`|
|Username|`root`|
|Password|`root`|

---

### **Guardar y conectar**

👉 Clic en **Save**, luego doble clic para conectar 🎉

---

## **¡Todo Listo!**

Ya tienes:

✅ PostgreSQL funcionando en Docker  
✅ pgAdmin en la misma red  
✅ Datos persistentes  
✅ Entorno completo para ejecutar SQL

---

## 🐍 **Script `full-load.py`**

---

### 🗂️ **1. Ubicación del submódulo en Codespaces**

### 📍 _Moverse dentro del proyecto_

```bash
cd data-engineering-roadmap-2026/
mkdir nivel-0
cd nivel-0/
```

📌 **NOTA:** Aquí es donde crearás tu script `full-load.py`.

Ahora ejecutamos el comando `wget` para descargar el  `yellow_tripdata_2024-01.parquet` en el folder `nivel-0/`

```bash
wget https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-01.parquet
```

---

### 🧾 **2. Crear el script `full-load.py`**

👉 Dentro de la carpeta `nivel-0`, crea el archivo y pega el siguiente contenido:

---

#### 🐍 **Script: `full-load.py`**

_(No se modifica, solo se presenta con estilo)_

```python
import pyarrow.parquet as pq
import pandas as pd
import psycopg2
from sqlalchemy import create_engine
import io

# --------------------------------
# CONFIG
# --------------------------------
PARQUET_FILE = "yellow_tripdata_2024-01.parquet"
TABLE_NAME = "yellow_taxi_data"

engine = create_engine("postgresql://root:root@localhost:5432/ny_taxi")

# --------------------------------
# 1) Crear tabla automáticamente con to_sql
# --------------------------------
df_schema = pq.read_table(PARQUET_FILE, columns=None).slice(0, 0).to_pandas()

df_schema.to_sql(
    name=TABLE_NAME, 
    con=engine,
    if_exists="replace",
    index=False
)

print("✔ Tabla creada automáticamente con to_sql\n")

# --------------------------------
# 2) Procesar parquet por row groups y cargar con COPY
# --------------------------------
pqfile = pq.ParquetFile(PARQUET_FILE)
num_groups = pqfile.num_row_groups

conn = psycopg2.connect("host=localhost dbname=ny_taxi user=root password=root")
cursor = conn.cursor()

print(f"Cargando {num_groups} row groups...\n")

for i in range(num_groups):
    print(f"→ Procesando row group {i+1}/{num_groups}")

    # Leer row group
    table = pqfile.read_row_group(i)
    
    # Convertir a pandas
    df = table.to_pandas()

    # ---------------------------------------
    # FIX: Normalización automática de tipos
    # ---------------------------------------
    # 1) Convertir floats sin decimales → enteros (Int64, nullable)
    for col in df.columns:
        if df[col].dtype == "float64":
            # Si todos los valores float son equivalentes a enteros (o hay NaN), cast seguro
            if ((df[col] % 1 == 0) | df[col].isnull()).all():
                df[col] = df[col].astype("Int64")

    # 2) Convertir automáticamente columnas datetime
    for col in df.columns:
        if "datetime" in col.lower():
            df[col] = pd.to_datetime(df[col], errors="coerce")

    # ---------------------------------------
    # COPY: exportar chunk a Postgres
    # ---------------------------------------
    buffer = io.StringIO()
    df.to_csv(buffer, index=False, header=False)
    buffer.seek(0)

    cursor.copy_expert(
        f"COPY {TABLE_NAME} FROM STDIN WITH CSV",
        buffer
    )
    conn.commit()

print("\n✔ Carga completa usando COPY + to_sql para el esquema.")

cursor.close()
conn.close()
```

---

### 🌱 **3. Crear el entorno virtual**

#### 🧱 _Paso indispensable_

```bash
python3 -m venv venv
```

---

### ⚡ **4. Activar el entorno virtual**

```bash
source venv/bin/activate
```

💡 **TIP:** En Codespaces, tu terminal debe mostrar un prefijo así:

```
(venv) z2h@codespaces-xxxx:~/...
```

---

### 📦 **5. Instalar dependencias**

#### 🔧 _Con el entorno activo:_

```bash
pip install pyarrow psycopg2-binary pandas sqlalchemy
```

---

### ▶️ **6. Ejecutar el script**

```bash
python3 full-load.py
```

---

### 🎉 **7. Salida esperada**

```bash
(venv) z2h@codespaces-2075b2 /w/s/d/nivel-0 (main)> python3 full-load.py
✔ Table automatically created with to_sql

Loading 3 row groups...

→ Processing row group 1/3
→ Processing row group 2/3
→ Processing row group 3/3

✔ Load completed using COPY + to_sql for schema.
```

---

### 📝 **Notas**

|⚠️ Situación|💡 Consejo|
|---|---|
|DB no responde|Verifica que el contenedor de Postgres esté corriendo|
|Error de módulo|Asegúrate de tener el entorno virtual activo `(venv)`|
|Archivo parquet no encontrado|Confirma que está en la carpeta `nivel-0`|

---

#### 🧭 **Checklist final**

✅ Carpeta `nivel-0` creada  
✅ Script `full-load.py` guardado  
✅ Entorno virtual creado  
✅ Dependencias instaladas  
✅ Base de datos levantada  
✅ Script ejecutado sin errores

---


# LAB

Perfecto — pensando en el contexto Nivel-0 (modelo relacional primero, OLTP-like) y en que usarás **PostgreSQL** para practicar, aquí tienes un paquete coherente y listo para copiar/pegar en un notebook/psql. Incluye:

- DDL para las tablas `trip`, `vendor`, `zone` (PKs, FKs, checks).
    
- Índices recomendados para consultas típicas del dataset NYC Taxi.
    
- Plantilla `COPY` / `\copy` para cargar CSVs.
    
- Consultas básicas de EDA y validación (qué todo principiante debe practicar).
    
- Ejemplos de vistas/materialized views para preparar el salto a OLAP (nivel 2).
    
- Comandos `EXPLAIN ANALYZE` y recomendaciones.
    

---

## 1) DDL — Crear esquema relacional (OLTP-like)

```sql
-- 1. Crear esquema (opcional)
CREATE SCHEMA IF NOT EXISTS nyc_taxi;
SET search_path = nyc_taxi, public;

-- 2. Tabla vendor (dimensión pequeña)
CREATE TABLE IF NOT EXISTS vendor (
  vendorid    SMALLINT PRIMARY KEY,
  vendor_name TEXT
);

-- 3. Tabla zone (dimensión geográfica / referencia)
CREATE TABLE IF NOT EXISTS zone (
  locationid  INTEGER PRIMARY KEY,
  borough     TEXT,
  zone_name   TEXT
);

-- 4. Tabla trip (hecho/transaccional)
CREATE TABLE IF NOT EXISTS trip (
  trip_id                BIGSERIAL PRIMARY KEY,
  tpep_pickup_datetime   TIMESTAMP WITH TIME ZONE NOT NULL,
  tpep_dropoff_datetime  TIMESTAMP WITH TIME ZONE NOT NULL,
  passenger_count        SMALLINT,
  trip_distance          DOUBLE PRECISION CHECK (trip_distance >= 0),
  pulocationid           INTEGER REFERENCES zone(locationid),
  dolocationid           INTEGER REFERENCES zone(locationid),
  vendorid               SMALLINT REFERENCES vendor(vendorid),
  fare_amount            NUMERIC(10,2) CHECK (fare_amount >= 0),
  total_amount           NUMERIC(10,2) CHECK (total_amount >= 0),
  payment_type           SMALLINT,
  -- opcional: columnas derivadas para Nivel-0 (date, hour)
  pickup_date            DATE GENERATED ALWAYS AS (tpep_pickup_datetime::date) STORED,
  pickup_hour            SMALLINT GENERATED ALWAYS AS (EXTRACT(HOUR FROM tpep_pickup_datetime)::INT) STORED
);
```

**Notas:**

- Uso `BIGSERIAL` para `trip_id` para evitar colisiones en cargas.
    
- Las columnas derivadas `pickup_date` y `pickup_hour` facilitan consultas comunes y son enseñables en Nivel-0. (Postgres 12+ soporta columnas `GENERATED ... STORED`.)
    

---

## 2) Índices recomendados (Nivel-0: priorizar lecturas frecuentes)

```sql
-- Índices para filtros y agrupaciones comunes
CREATE INDEX IF NOT EXISTS idx_trip_pickup_datetime ON trip (tpep_pickup_datetime);
CREATE INDEX IF NOT EXISTS idx_trip_pickup_date_hour ON trip (pickup_date, pickup_hour);
CREATE INDEX IF NOT EXISTS idx_trip_pulocation ON trip (pulocationid);
CREATE INDEX IF NOT EXISTS idx_trip_dolocation ON trip (dolocationid);
CREATE INDEX IF NOT EXISTS idx_trip_vendor ON trip (vendorid);
CREATE INDEX IF NOT EXISTS idx_trip_payment ON trip (payment_type);

-- Índice compuesto útil para consultas OLAP simples (ejemplo: agregaciones por date+zone)
CREATE INDEX IF NOT EXISTS idx_trip_date_puloc ON trip (pickup_date, pulocationid);
```

---

## 3) Carga de datos (plantillas)

Si tienes CSV descargado localmente, usa `\copy` en psql (permite acceso a archivos en tu máquina):

```sql
-- Ejemplo: columnas exactas deben coincidir con el CSV
\copy nyc_taxi.trip(tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance, pulocationid, dolocationid, fare_amount, total_amount, payment_type, vendorid) 
FROM '/ruta/a/yellow_tripdata_2023-01.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',');
```

O si prefieres `COPY` desde servidor (archivo en servidor DB):

```sql
COPY nyc_taxi.trip(tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance, pulocationid, dolocationid, fare_amount, total_amount, payment_type, vendorid)
FROM '/var/lib/postgresql/data/yellow_tripdata_2023-01.csv' WITH (FORMAT csv, HEADER true);
```

**Recomendación Nivel-0:** carga una porción (ej. 10k–100k filas) para practicar antes de cargar TB.

---

## 4) Inserts de ejemplo para vendor y zone (pequeñas referencias)

```sql
INSERT INTO vendor (vendorid, vendor_name) VALUES
(1, 'Creative Mobile Technologies'),
(2, 'VeriFone Inc');

-- Ejemplo zone (usa el mapping TPEP Location IDs según taxi zone lookup)
INSERT INTO zone (locationid, borough, zone_name) VALUES
(1, 'EWR', 'Newark Airport'),
(237, 'Manhattan', 'Upper East Side'),
(236, 'Manhattan', 'Upper West Side')
-- ... agregar más o cargar desde CSV de lookup
;
```

---

## 5) Consultas básicas de EDA (Nivel-0 — practicar y entender los datos)

### 5.1 Conteos y sanity checks

```sql
-- Total de viajes
SELECT COUNT(*) AS total_trips FROM trip;

-- Rango de fechas
SELECT MIN(tpep_pickup_datetime) AS first_pickup, MAX(tpep_pickup_datetime) AS last_pickup FROM trip;

-- Viajes con valores sospechosos (negativos o 0)
SELECT COUNT(*) AS invalid_fares FROM trip WHERE fare_amount <= 0 OR total_amount <= 0;
```

### 5.2 Distribución básica

```sql
-- Número de viajes por día (top 10 días)
SELECT pickup_date, COUNT(*) AS trips
FROM trip
GROUP BY pickup_date
ORDER BY trips DESC
LIMIT 10;

-- Viajes por zona de pickup (top 10)
SELECT z.zone_name, z.borough, COUNT(*) AS trips
FROM trip t
JOIN zone z ON t.pulocationid = z.locationid
GROUP BY z.zone_name, z.borough
ORDER BY trips DESC
LIMIT 10;

-- Distancia media y tarifa media por borough
SELECT z.borough,
       AVG(trip_distance) AS avg_distance,
       AVG(total_amount) AS avg_total
FROM trip t
JOIN zone z ON t.pulocationid = z.locationid
GROUP BY z.borough
ORDER BY avg_total DESC;
```

### 5.3 Horas pico

```sql
-- Viajes por hora del día
SELECT pickup_hour, COUNT(*) AS trips
FROM trip
GROUP BY pickup_hour
ORDER BY pickup_hour;
```

### 5.4 Duración y anomalías

```sql
-- Añadir columna temporal de duración (segundos) en una consulta
SELECT trip_id,
       EXTRACT(EPOCH FROM (tpep_dropoff_datetime - tpep_pickup_datetime)) AS duration_secs,
       trip_distance,
       total_amount
FROM trip
WHERE EXTRACT(EPOCH FROM (tpep_dropoff_datetime - tpep_pickup_datetime)) > 0
ORDER BY duration_secs DESC
LIMIT 20;

-- Viajes con distancia 0 pero duración > 0 (posible ruido)
SELECT COUNT(*) FROM (
  SELECT trip_id
  FROM trip
  WHERE trip_distance = 0 AND tpep_dropoff_datetime > tpep_pickup_datetime
) t;
```

---

## 6) Vistas y Materialized Views para preparar salto a OLAP (Nivel-0 → Nivel-1)

### 6.1 Vista simple denormalizada (útil para análisis rápido)

```sql
CREATE OR REPLACE VIEW vw_trip_denorm AS
SELECT t.trip_id,
       t.tpep_pickup_datetime,
       t.tpep_dropoff_datetime,
       t.pickup_date,
       t.pickup_hour,
       t.passenger_count,
       t.trip_distance,
       t.total_amount,
       t.payment_type,
       v.vendor_name,
       zp.zone_name AS pickup_zone,
       zp.borough   AS pickup_borough,
       zd.zone_name AS dropoff_zone,
       zd.borough   AS dropoff_borough
FROM trip t
LEFT JOIN vendor v ON t.vendorid = v.vendorid
LEFT JOIN zone zp ON t.pulocationid = zp.locationid
LEFT JOIN zone zd ON t.dolocationid = zd.locationid;
```

### 6.2 Materialized view agregada por día+zone (refrescar periódicamente)

```sql
CREATE MATERIALIZED VIEW mv_daily_zone AS
SELECT pickup_date,
       pulocationid,
       COUNT(*) AS trips,
       SUM(total_amount) AS total_revenue,
       AVG(trip_distance) AS avg_distance
FROM trip
GROUP BY pickup_date, pulocationid;

-- Para refrescar manualmente
REFRESH MATERIALIZED VIEW mv_daily_zone;
```

**Uso pedagógico:** explicar por qué materializar (costo de refresh vs velocidad de consulta) — buen punto en Nivel-0/1.

---

## 7) EXPLAIN ANALYZE y comprobación de performance (básico, enseñable)

```sql
EXPLAIN ANALYZE
SELECT z.zone_name, COUNT(*) AS trips
FROM trip t
JOIN zone z ON t.pulocationid = z.locationid
WHERE t.pickup_date = '2023-01-15'
GROUP BY z.zone_name
ORDER BY trips DESC
LIMIT 10;
```

- Enseña a los estudiantes a leer el plan (sequential scan vs index scan, cost, rows, time).
    
- En Nivel-0, buscar si la consulta usa índices esperados y qué parte es el cuello de botella.
    

---

## 8) Ejemplo de transformación simple (normalización vs desnormalización)

### 8.1 Normalizar: crear tabla `payment_type` (si queremos metadatos)

```sql
CREATE TABLE IF NOT EXISTS payment_type (
  payment_type SMALLINT PRIMARY KEY,
  payment_desc TEXT
);

INSERT INTO payment_type (payment_type, payment_desc) VALUES
(1, 'Credit card'),
(2, 'Cash'),
(3, 'No charge'),
(4, 'Dispute'),
(5, 'Unknown');
```

### 8.2 Desnormalizar: crear tabla resumen para BI (ejemplo OLAP-ready)

```sql
CREATE TABLE IF NOT EXISTS fact_trips_day_zone AS
SELECT
  pickup_date,
  pulocationid,
  COUNT(*) AS trips,
  SUM(total_amount) AS revenue,
  AVG(trip_distance) AS avg_distance
FROM trip
GROUP BY pickup_date, pulocationid
WITH NO DATA;  -- crear estructura vacía

-- Llenar con datos iniciales
INSERT INTO fact_trips_day_zone
SELECT pickup_date, pulocationid, COUNT(*), SUM(total_amount), AVG(trip_distance)
FROM trip
GROUP BY pickup_date, pulocationid;
```

**Enseñanza:** explicar trade-offs entre espacio (duplicación) y velocidad (consultas agregadas rápidas).

---

## 9) Buenas prácticas Nivel-0 (resumen corto)

- Empieza modelando correctamente (tablas, PK, FK).
    
- Valida datos con queries sencillas (min/max, counts, anomalies).
    
- Crea índices sólo después de entender consultas frecuentes.
    
- Usa `EXPLAIN ANALYZE` para enseñar el plan de ejecución.
    
- Usa `views` y `materialized views` para separar OLTP y necesidades OLAP.
    
- Carga sólo muestras al principio y documenta el pipeline en notebooks.
    

---

Si quieres, te entrego ahora:

- Un **notebook (SQL + markdown)** con estas consultas ordenadas como clase paso-a-paso.
    
- Un **script de carga psql** listo para ejecutar en un entorno local con ejemplos de `\copy`.
    
- Un **mini-lab** con ejercicios (3 ejercicios de EDA + 3 de modelado) para entregar a estudiantes.
    

¿Cuál prefieres siguiente?
