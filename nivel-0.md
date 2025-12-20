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

# 🧪 LAB — SQL-FIRST EN NOTEBOOKS (Nivel-0 · Roadmap DE 2026)
 
**Setup → SQL puro → luego scripts acercándose al Nivel-1**.

> **SQL puro como base del Data Engineering**  
> Notebook como entorno de ejecución y documentación, **no como motor de lógica**.

Este LAB forma parte del **Nivel-0** del Roadmap **Data Engineering 2026** y tiene como objetivo **construir una base sólida en SQL**, antes de introducir pipelines, scripts y automatización (Nivel-1).

---

## 🎯 Objetivo del LAB (Nivel-0)

En este LAB aprenderás a:

- Trabajar con **SQL puro** sobre PostgreSQL
    
- Ejecutar SQL desde notebooks **sin depender de Python**
    
- Entender el notebook como:
    
    - herramienta de aprendizaje
        
    - bitácora técnica
        
    - entorno reproducible
        
- Cargar y consultar datos **directamente desde el motor SQL**
    
- Prepararse para el paso natural hacia **scripts y pipelines** en el Nivel-1
    

👉 **Aquí NO estamos construyendo pipelines**  
👉 **Aquí estamos formando criterio SQL**

---

## 🧠 Principio rector del Nivel-0: SQL-first

> _Antes de automatizar, hay que entender el motor._

Esto implica que:

- El SQL debe ser **ejecutable fuera del notebook**
    
- El notebook **no es un requisito técnico**
    
- No hay lógica de negocio en Python
    
- El foco está en:
    
    - DDL
        
    - DML
        
    - consultas
        
    - lectura y comprensión del modelo de datos
        

---

## 🥇 OPCIÓN PRINCIPAL — SQL PURO CON `psql` DESDE NOTEBOOKS

### 📌 ¿Por qué `psql` en Nivel-0?

Porque representa **la forma más directa y honesta** de interactuar con PostgreSQL:

- Es el cliente oficial
    
- Es el mismo SQL que se usa en terminal, servidores y CI
    
- No hay abstracciones
    
- No hay “magia de notebook”
    

👉 **Es SQL real ejecutado por el motor real**

---

### 🔧 Requisitos

- PostgreSQL corriendo en Docker (setup previo)
    
- `psql` instalado en el sistema operativo
    

```bash
sudo apt-get update
sudo apt install postgresql-client
```

> ⚠️ `psql` **no va en un venv**, es una herramienta del sistema.

- Jupyter Notebook (solo como runner de Bash)

#### **Crear el archivo `docker-compose.yml`**

Antes de todo lo demás, **¡creemos el archivo `docker-compose.yml`!**

📁 **Ubicación**: Asegúrate de crear este archivo en el **nivel-0** de tu proyecto (directorio raíz).

🔑 **Archivo a crear**: `docker-compose.yml`

Copia y pega el siguiente código en el archivo:

```yml
version: '3.8'
services:
  postgres:
    image: postgres:13
    container_name: postgres-container
    environment:
      POSTGRES_USER: "root"
      POSTGRES_PASSWORD: "root"
      POSTGRES_DB: "ny_taxi"
    volumes:
      - ./nyc-tlc-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - pg-network

  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin-container
    environment:
      PGADMIN_DEFAULT_EMAIL: "admin@admin.com"
      PGADMIN_DEFAULT_PASSWORD: "root"
    ports:
      - "5050:80"
    networks:
      - pg-network

networks:
  pg-network:
    driver: bridge
```

#### 💡 **Ventajas de usar Docker Compose:**

- **Configuración fácil** con un solo archivo. 📑
    
- **Escalabilidad**: Si agregas más servicios, simplemente añades más definiciones al archivo. 🔄
    
- **Gestión de redes**: Los contenedores en el mismo archivo de Compose se pueden comunicar automáticamente. 🌐

#### 📝 **Notas**

- **Redes**: Los contenedores se pueden comunicar entre sí solo si están en la **misma red**. Esto es manejado automáticamente por Docker Compose. 🕸️
    
- **Persistencia de datos**: ¡Recuerda! El volumen configurado para PostgreSQL asegura que **no perderás los datos** aunque el contenedor se reinicie. 🔒

### 🚀 **Siguiente paso: Levantar los contenedores con Docker Compose**

¡Ahora que ya tienes tu archivo `docker-compose.yml`! 🎉

- 🚀 **Para iniciar los contenedores**: Ve a la terminal y ejecuta:
    

```bash
docker-compose up
```

¡Y listo! Ahora tus contenedores **PostgreSQL** y **pgAdmin** deberían estar en funcionamiento. 🛠️

- 🔄 Si quieres detener los contenedores presiona la combinación de teclas `Ctrl+x` para detener los servicios en la terminal donde ejecutas `docker-compose.yml` y luego en la misma terminal cuando se hayan detenido los servicios ejecuta:
    

```bash
docker compose down
```


---

### ▶️ Ejecución de SQL desde el notebook

Primero crearemos un nuevo Notebook al que llamaremos `test.ipynb`

Ejemplo básico de conexión:

```bash
%%bash
psql postgresql://root:root@localhost:5432/postgres -c "SELECT 1;"
```

Ejemplo de consulta real:

```bash
%%bash
psql postgresql://root:root@localhost:5432/postgres <<'SQL'
SELECT
    table_schema,
    table_name
FROM information_schema.tables
ORDER BY table_schema, table_name
LIMIT 10;
SQL
```

---

### 📦 Carga de datos en el LAB (SQL puro)

En este LAB **los datos se cargan usando SQL**, normalmente desde CSV, por ejemplo con `COPY`.

👉 Esto es **intencional**.

**Objetivo de aprendizaje:**

- Entender cómo el motor:
    
    - lee archivos
        
    - valida tipos
        
    - falla
        
    - reporta errores
        
- Practicar SQL real:
    
    - `CREATE TABLE`
        
    - `COPY`
        
    - `INSERT`
        
    - `SELECT`
        

👉 Aquí **ves el motor trabajar**, sin intermediarios.

---

### ✅ Ventajas de este enfoque en Nivel-0

- SQL 100% real
    
- Misma experiencia que en producción
    
- Ideal para:
    
    - fundamentos
        
    - Data Engineering
        
    - transición a scripts `.sql`
        
- No depende de Python
    
- Forma criterio técnico sólido
    

---

### ⚠️ Limitaciones

- Cada celda es un proceso nuevo
    
- No hay sesión persistente
    
- Menos ergonomía visual
    

👉 Estas limitaciones **son parte del aprendizaje**, no un problema.

### 🧩 Casos ideales

- Aprendizaje SQL 
    
- Pipelines SQL
    
- Data Engineering
    
- CI / automatización
    
- SQL Server en Docker

---

## 🥈 OPCIÓN COMPLEMENTARIA — JupySQL (`%%sql`)

> ⚠️ **Secundaria en Nivel-0**  
> Útil para exploración y documentación, no como base.

### 📌 Qué es

JupySQL permite ejecutar SQL en notebooks usando `%%sql`.

- Más legible
    
- Mejor DX
    
- Sigue siendo SQL-first
    
- Python solo actúa como host del kernel
    

---

### 📦 Instalación

```python
pip install jupysql
```

### ⚠️ Importante: cómo se carga

❌ Incorrecto:

```python
%load_ext jupysql
```

✅ Correcto:

```python
%load_ext sql
```

JupySQL **expone la extensión como `sql`**, no como `jupysql`.

---

### 🔗 Conexión

```python
%sql postgresql://root:root@localhost:5432/postgres
```

```python
%load_ext sql
%sql postgresql://root:root@localhost:5432/postgres
%sql select 1;
```
---

### ✍️ SQL identado (multilinea)

👉 **Sin identación**

```python
%sql SELECT table_schema, table_name FROM information_schema.tables WHERE table_type = 'BASE TABLE' ORDER BY table_schema, table_name;
```

👉 **Con identación: Siempre usar `%%sql`**

```sql
%%sql
SELECT
    table_schema,
    table_name
FROM information_schema.tables
WHERE table_type = 'BASE TABLE'
ORDER BY
    table_schema,
    table_name;
```

---

### ❌ Qué NO usar

```sql
%sql
SELECT
  *
FROM table;
```

`%sql` es **line magic** → no soporta multilinea.

---

### ✅ Ventajas

- SQL legible y documentado
    
- Ideal para **Data Analytics**
    
- Multilinea y comentarios
    
- Mejor DX que `psql`
---

### ⚠️ Advertencia

- JupySQL **no reemplaza** a `psql`
    
- No se usa para:
    
    - CI
        
    - pipelines
        
    - producción
        
- Es una **herramienta de apoyo**, no el estándar
    

---

## 🆚 Comparación (en contexto Nivel-0)

|Criterio|psql|JupySQL|
|---|---|---|
|SQL real|⭐⭐⭐⭐⭐|⭐⭐⭐⭐|
|Dependencia Python|❌|⚠️ mínima|
|Fundamentos DE|⭐⭐⭐⭐⭐|⭐⭐⭐|
|Exploración|⭐⭐|⭐⭐⭐⭐|
|Escala a producción|⭐⭐⭐⭐⭐|❌|

---

## 🧠 Reglas de oro

1. **SQL vive en `.sql` si el proyecto es SQL‑first**
	
2. `%sql` solo para pruebas rápidas
	
3. `%%sql` para SQL real
	
4. `psql` para ingeniería y automatización
    
5. El notebook **no es requisito técnico**
    
6. Si puedes ejecutar el SQL en terminal, vas bien
    
7. Primero entiendes el motor
    
8. Luego automatizas (Nivel-1)
    

---

## 🔜 Conexión con el Nivel-1

En el **Nivel-1**:

- La carga de datos se hará mediante **scripts**
    
- Aparecen:
    
    - Python como orquestador
        
    - automatización
        
    - pipelines
        
- El SQL que aprendiste aquí **no cambia**
    

👉 **Primero entiendes el motor**  
👉 **Luego construyes ingeniería sobre él**

## 🎯 Recomendación final

- **Data Engineering** → `psql + .sql`
    
- **Data Analytics** → `JupySQL (%%sql)`
    
- **Mixto** → usar ambos conscientemente
    

> _**NOTA:** Si mañana quitas el notebook y todo sigue funcionando, el diseño es correcto._


## 📎 Nota sobre SQL Server

Todo lo anterior aplica también a:

- SQL Server local
    
- SQL Server en Docker
    

Cambiando únicamente:

- Cliente (`sqlcmd` o drivers)
    
- URI de conexión
    

La **arquitectura mental no cambia**

