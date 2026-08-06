# Guion de texto — Level 1 Lab 01: FastAPI + PostgreSQL con Docker Compose

> **Cómo usar este guion:** cada título de este archivo es idéntico a un título de `level-1/docs/section-3.md`. Se leen lado a lado: la section muestra los archivos y los comandos; este guion explica lo mismo con nuestras propias palabras, en estilo for dummies, definiendo cada término técnico la primera vez que aparece. Si en algún momento algo suena a chino, la definición está un poco más arriba, en el bloque anterior.

---

# 🧪 Lab 01: FastAPI + PostgreSQL con Docker Compose

Bienvenidos al momento de la verdad del nivel. Hasta acá, teoría: vimos el problema que resuelve Docker, abrimos el motor (capas, namespaces, cgroups), y cerramos el sistema completo (redes, persistencia, Dockerfile, Compose, producción). Todo eso era la parte conceptual.

Ahora toca lo que realmente importa: **construir algo real**. Este lab es la primera vez que vamos a montar una plataforma completa funcionando — no un "hola mundo" suelto, sino una API conectada a una base de datos, empaquetada con Docker y operada como si estuviera en producción.

Un repaso rápido de lo que vamos a usar, porque todo ya lo vimos:

- **Dockerfile multi-stage** — la receta de la imagen en dos etapas, para que la imagen final sea chica y limpia.
- **Docker Compose con redes y volúmenes** — el sistema completo en un archivo: API + base de datos, en su propia red privada, con datos persistentes.
- **HEALTHCHECK y restart policies** — el sistema se vigila solo y se recupera solo.
- **docker logs, inspect, exec, stats** — las herramientas de diagnóstico que usan los profesionales para ver qué pasa adentro.
- **Persistencia con named volumes** — los datos sobreviven a los contenedores.

**Nota:** un "lab" es una práctica guiada: una actividad con pasos definidos donde se construye algo real. Acá el resultado es una API REST funcionando contra una base de datos PostgreSQL, todo en contenedores.

---

## 🎯 Objetivo

La meta concreta de este lab: construir y desplegar una **API REST** en **FastAPI** conectada a **PostgreSQL**, empaquetada completamente con Docker, y validar que funciona, que persiste datos y que puede debuggearse como en producción.

Desglosemos los términos, porque acá hay varios nuevos:

- **API** (Application Programming Interface) — en criollo: un servicio que responde a pedidos. Nuestra API va a recibir pedidos por la red y responder con datos. Es la "cara" de la base de datos hacia el mundo.
- **API REST** — un estilo de diseño de APIs que usa las operaciones HTTP (las mismas de la web) para manipular datos: crear, leer, actualizar y borrar. Ese patrón de cuatro operaciones se llama **CRUD** (Create, Read, Update, Delete — crear, leer, actualizar, borrar).
- **FastAPI** — un framework web de Python, o sea, una base de código ya hecha para construir APIs rápidas y con menos esfuerzo.
- **PostgreSQL** — el motor de base de datos relacional que ya conocemos del guion anterior.
- **Endpoints** — los "puntos de entrada" de la API: cada URL que la API expone (por ejemplo `/items` o `/health`) es un endpoint.

Y el verbo que lo cierra todo: **desplegar** — poner el sistema a correr en su entorno definitivo, que acá es el entorno de Docker. Al final del lab, esa API y esa base van a estar corriendo, persistiendo y dejándose inspeccionar como un servicio de verdad.

---

## 📦 Repositorio

Todo el código del lab está en un repositorio de GitHub que vamos a clonar. En la terminal del Codespace:

```bash
git clone https://github.com/z2h-academy/devops-engineering-bootcamp-2026.git
cd devops-engineering-bootcamp-2026/labs/lab-01-api-db
```

**Nota:** `git clone` descarga una copia completa de un repositorio remoto a nuestra máquina. Un "repositorio" (repo) es el proyecto con todo su historial de versiones — ya lo conocemos del Level 0. Y `cd` nos mete en la carpeta del lab: a partir de acá, todos los comandos del lab se ejecutan desde `lab-01-api-db/`.

Dentro encontramos esta estructura:

```text
lab-01-api-db/
├── api/
│   ├── Dockerfile         # Imagen multi-stage + healthcheck
│   ├── requirements.txt   # Dependencias Python
│   └── app/               # Código fuente de la API
│       ├── __init__.py    # Marca el directorio como paquete
│       ├── main.py        # Endpoints FastAPI
│       ├── database.py    # Conexión a PostgreSQL
│       ├── models.py      # Modelos SQLAlchemy
│       └── schemas.py     # Esquemas Pydantic
├── docker-compose.yml     # Orquestación de servicios
├── README.md              # Guía detallada del lab
└── scripts/
    ├── setup.sh           # Deploy automático
    ├── validate.sh        # Pruebas de endpoints
    └── cleanup.sh         # Destruir stack
```

Hagamos el recorrido de la estructura, porque cada pieza tiene un rol:

- `api/Dockerfile` — la receta de la imagen de la API (la vamos a leer completa en el Paso 1).
- `api/requirements.txt` — la lista de dependencias de Python, el "package.json de Python" que vimos antes.
- `api/app/` — el código fuente de la API. Dentro:
  - `main.py` — donde están definidos los endpoints.
  - `database.py` — la conexión a PostgreSQL.
  - `models.py` — los **modelos SQLAlchemy**: las estructuras de datos que la API usa para hablar con la base (SQLAlchemy es una librería de Python para trabajar con bases de datos desde código).
  - `schemas.py` — los **esquemas Pydantic**: las reglas de validación de los datos que entran y salen por la API (Pydantic es la librería que valida datos en FastAPI).
  - `__init__.py` — un archivo que le dice a Python "esta carpeta es un paquete" (una unidad de código importable).
- `docker-compose.yml` — la orquestación: define los dos servicios y cómo se relacionan (lo leemos en el Paso 2).
- `README.md` — la guía detallada del lab, con ejercicios extra.
- `scripts/` — tres scripts que automatizan el ciclo de vida: `setup.sh` (desplegar), `validate.sh` (probar), `cleanup.sh` (destruir).

**Nota:** "dependencias" son las librerías de terceros que nuestro código necesita para funcionar; "paquete" en Python es una carpeta con código organizado que se puede importar; y "orquestación" es la coordinación de varios servicios para que funcionen como un solo sistema.

---

## 🧱 Paso 1: Entender el Dockerfile

Regla de oro del lab: **antes de ejecutar nada, entender qué estamos ejecutando**. Abrimos `api/Dockerfile` en el editor del Codespace y lo leemos completo.

Este Dockerfile tiene una técnica que todavía no vimos en detalle: el **multi-stage build** (construcción multi-etapa). La idea es simple y brillante: dentro de un mismo Dockerfile definimos **dos (o más) etapas**, cada una con su `FROM`, y al final solo la **última etapa** queda como imagen definitiva. Las anteriores son talleres temporales: sirven para fabricar lo que necesitamos, y se descartan. ¿Por qué? Porque construir software deja basura: compiladores, cachés, archivos temporales. Esa basura no debe viajar a producción. Con multi-stage, la imagen final es solo lo que la app necesita para correr.

### Primera etapa: Builder

La primera etapa se llama **builder** (constructor). Su trabajo: preparar el entorno de ejecución con todas las dependencias.

```dockerfile
FROM python:3.12-slim AS builder
```

**Nota:** `FROM` es la primera instrucción de cualquier Dockerfile: "arrancá desde esta imagen base". `python:3.12-slim` es la imagen oficial de Python 3.12 sobre **Debian Slim** — Debian es una distribución de Linux muy usada en servidores, y la variante "slim" es la versión reducida, sin las herramientas de desarrollo que no hacen falta en runtime. Y `AS builder` le pone nombre a esta etapa para poder referenciarla después (eso es lo que hace el multi-stage posible).

```dockerfile
RUN python -m venv /opt/venv
```

`RUN` ejecuta un comando dentro del contenedor durante la construcción de la imagen. Acá ejecutamos `python -m venv /opt/venv` — "ejecutá el módulo `venv` de Python en la ruta `/opt/venv`".

**Nota:** `venv` crea un **entorno virtual** (virtualenv): una carpeta aislada con su propia copia de Python, donde se instalan las dependencias de la app sin mezclarse con el resto del sistema. Piensen en un cajón cerrado: todo lo de nuestra app vive ahí adentro, y nada contamina lo de afuera.

```dockerfile
ENV PATH="/opt/venv/bin:$PATH"
```

`ENV` define una variable de entorno dentro de la imagen. Acá modificamos la variable **PATH**.

**Nota:** el `PATH` es la lista de directorios donde el sistema operativo busca los programas cuando escribimos un comando. Si escribimos `pip` y el sistema no sabe dónde está, no lo encuentra. Al poner `/opt/venv/bin` al principio del PATH, cuando ejecutemos `pip` o `uvicorn` (los programas del virtualenv), el sistema los encuentra primero ahí, en el cajón de nuestra app.

```dockerfile
COPY requirements.txt .
```

`COPY` copia archivos desde nuestra máquina hacia la imagen. El primer argumento (`requirements.txt`) es el archivo en el proyecto; el segundo (`.`) es el destino: el directorio de trabajo actual de la imagen. Ojo con el detalle fino: copiamos **solo** la lista de dependencias, no todo el código todavía. ¿Por qué? Por el **caché de capas** que vimos en la Parte 2: si `requirements.txt` no cambió, Docker reutiliza la capa de instalación del caché y no reinstala dependencias en cada build. Orden en el Dockerfile = builds rápidos.

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

`pip` es el gestor de paquetes de Python: el programa que instala librerías. `-r requirements.txt` lee el archivo que copiamos e instala todo lo que está listado (fastapi, uvicorn, sqlalchemy, psycopg2, pydantic). `--no-cache-dir` le dice a pip que no guarde el caché de descargas dentro de la imagen — porque ese caché es basura que no necesitamos en producción y ocupa espacio.

Al final de esta etapa, el virtualenv de `/opt/venv` tiene todas las librerías necesarias para correr la app.

#### Resumen de la etapa builder

Para que quede claro el rol de esta etapa: toma Python 3.12 slim, crea un virtualenv, configura el PATH para usarlo, copia la lista de dependencias y las instala.

Esta etapa pesa unos **~300MB**, porque incluye el compilador de Python, el instalador de paquetes y los archivos temporales de pip. Pero no importa: **esta imagen nunca llega a producción**. Es el taller. Lo que nos llevamos del taller es solo el virtualenv terminado.

### Segunda etapa: Runner

La segunda etapa se llama **runner** (ejecutor). Esta sí es la imagen final, la que se ejecuta en producción. Y acá está la magia del multi-stage: arranca de una base **limpia**, sin nada de lo que fabricó la etapa anterior, y solo copia lo que necesita.

```dockerfile
FROM python:3.12-slim AS runner
```

Misma imagen base limpia que el builder — pero ahora sin dependencias instaladas. Empezamos de cero, con un sistema chico y sin basura.

```dockerfile
RUN adduser --disabled-password --no-create-home appuser
```

**Nota:** `adduser` es el comando de Linux para crear usuarios. `--disabled-password` crea el usuario sin contraseña (no puede iniciar sesión interactivamente, solo ejecutar procesos). `--no-create-home` no crea su carpeta personal. `appuser` es el nombre.

¿Por qué un usuario especial? Porque por defecto, los procesos de Docker corren como `root` — el superusuario, el que puede tocar todo. Si un atacante compromete el contenedor, con root adentro ya está a un paso del host. Con `appuser` (un usuario sin privilegios) limitamos ese riesgo: aunque el contenedor caiga, el atacante no tiene poderes de administrador. Es la buena práctica de "nunca root" que vimos en el guion anterior, llevada a la práctica.

```dockerfile
COPY --from=builder --chown=appuser:appuser /opt/venv /opt/venv
```

Esta es **la instrucción clave del multi-stage**. `COPY --from=builder` no copia desde nuestra máquina: copia **desde la imagen de la etapa `builder`**. Estamos trayendo el virtualenv terminado (con todas las dependencias instaladas) hacia la etapa runner.

Y `--chown=appuser:appuser` cambia el **dueño** de todos los archivos copiados al usuario `appuser` (el formato es `usuario:grupo`).

**Nota:** "dueño" de un archivo = qué usuario tiene permiso sobre él. Sin `--chown`, los archivos quedarían siendo de `root` (porque la etapa builder corrió como root), y `appuser` no tendría permiso ni para leerlos ni para ejecutarlos. La app arrancaría y explotaría por permisos. Este detalle parece menor, pero es exactamente el tipo de bug que pierde horas de debugging.

```dockerfile
WORKDIR /app
```

`WORKDIR` establece el directorio de trabajo para todas las instrucciones siguientes: a partir de acá, todo `COPY`, `RUN` y `CMD` relativo se ejecuta desde `/app`. Si la carpeta no existe, Docker la crea automáticamente. (Es el "cd automático" que vimos antes.)

```dockerfile
COPY --chown=appuser:appuser ./app ./app
```

Acá copiamos el código de la aplicación desde nuestra máquina: `./app` (la carpeta del proyecto) va a `./app` dentro del contenedor — que como `WORKDIR` es `/app`, termina siendo `/app/app`. Y otra vez `--chown` para que el código sea de `appuser`, no de root.

```dockerfile
USER appuser
```

`USER` le dice a Docker que todos los procesos siguientes —incluido el `CMD` final— corran como `appuser` y no como root. Desde este momento, la app corre con permisos de un usuario común. Justo lo que queremos.

```dockerfile
ENV PATH="/opt/venv/bin:$PATH"
```

Otra vez el PATH: los binarios del virtualenv que copiamos desde builder quedan accesibles. Sin esto, el `CMD` no encontraría `uvicorn`.

```dockerfile
EXPOSE 8000
```

`EXPOSE` es una instrucción de **documentación**: le dice a cualquiera que mire la imagen "esta app escucha en el puerto 8000".

**Nota:** un "puerto" es el canal numerado por donde una app recibe conexiones (el número de departamento del edificio-IP, como vimos en el guion anterior). El 8000 es el puerto por defecto de **uvicorn** — el servidor web que ejecuta FastAPI.

Un detalle que confunde a todos al principio: `EXPOSE` **no publica** el puerto. No lo hace accesible desde afuera del contenedor. Solo es una anotación en los metadatos de la imagen (visible con `docker inspect`). Para que el puerto sea realmente accesible desde el host hay dos formas:

- `docker run -p 8000:8000` — publica el puerto en tiempo de ejecución.
- `ports: - "8000:8000"` en `docker-compose.yml` — hace lo mismo, declarado en el archivo.

Y si no ponemos `EXPOSE` pero sí `ports:`, el puerto igual se publica. `EXPOSE` no es obligatorio — es buena práctica porque documenta la intención: cualquiera que vea la imagen sabe que debe mapear el 8000.

El `8000` del EXPOSE debe coincidir con el `--port 8000` del CMD y con el puerto del healthcheck. Si se desincronizan, el sistema puede reportar la app como muerta cuando está viva.

#### Resumen de la etapa runner

Arranca desde Python 3.12 slim limpio, crea un usuario no root, copia solo el virtualenv con dependencias (cambiando el dueño), copia el código de la aplicación, configura el usuario no root y el PATH.

La imagen final contiene solo: sistema base + virtualenv con dependencias + código de la app. Sin compiladores, sin caché de pip, sin archivos temporales. Eso es lo que hace una imagen de producción: lo mínimo para correr, nada más.

### HEALTHCHECK

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"
```

**Nota:** un **healthcheck** (chequeo de salud) es el "chequeo médico" del contenedor: un comando que Docker ejecuta periódicamente para saber si la app está viva de verdad. No basta con que el proceso corra: la app puede estar "colgada" sin responder, y un proceso colgado cuenta como vivo para el sistema operativo. El healthcheck distingue "el proceso existe" de "la app responde".

#### Parámetros

Tres parámetros gobiernan el chequeo:

- `--interval=30s` — cada cuánto se ejecuta el chequeo (cada 30 segundos).
- `--timeout=5s` — cuánto esperar por una respuesta antes de darlo por fallido (5 segundos).
- `--retries=3` — después de 3 fallos consecutivos, el contenedor se marca `unhealthy` (enfermo).

#### Comando ejecutado

El comando del chequeo es este:

```python
python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"
```

Vamos por partes: `python -c "..."` ejecuta código Python directamente desde la línea de comandos. Ese código importa `urllib.request` (una librería de Python para hacer pedidos web) y abre la URL `http://localhost:8000/health` — o sea, le hace un **GET** (un pedido de lectura) a nuestro propio endpoint de salud.

Dos detalles finos:

- La URL usa `localhost` porque el chequeo se ejecuta **dentro del mismo contenedor**: no necesita salir a la red, se mira el ombligo.
- El endpoint `/health` es un endpoint especial de la API que no hace nada más que responder "estoy viva". Es el estándar de la industria: toda app seria expone uno.

#### Códigos de salida

¿Cómo sabe Docker si el chequeo pasó? Por el **código de salida** del comando — el número que un programa devuelve al terminar, donde 0 significa éxito:

- `0` → healthy (sano): el pedido respondió.
- `1` → fallo: el pedido falló o devolvió error.
- timeout/error → también cuenta como fallo.

Y cuando un contenedor se marca `unhealthy`, Docker Compose puede usarlo para coordinar arranques:

```yaml
depends_on:
  db:
    condition: service_healthy
```

Traducción: "no arranques este servicio hasta que el otro esté **healthy**". Esto lo vamos a ver en acción en el compose.

### CMD

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

`CMD` define el comando que se ejecuta cuando el contenedor arranca — el proceso que será el PID 1 del contenedor (el "jefe" de todos los procesos internos, que vimos en la Parte 2).

#### Componentes

Desglosemos los cuatro argumentos:

**Servidor**

```text
uvicorn
```

**uvicorn** es el servidor ASGI para FastAPI. **Nota:** ASGI es el estándar que define cómo Python maneja pedidos web asincrónicos (varias peticiones a la vez sin bloquearse). En criollo: uvicorn es el programa que recibe los pedidos HTTP y se los pasa a nuestra app FastAPI.

**Aplicación**

```text
app.main:app
```

Es la dirección interna de nuestra app dentro del paquete, en formato `carpeta.archivo:instancia`:

- `app` → el paquete Python (la carpeta con `__init__.py`).
- `main` → el archivo `main.py`.
- `app` → la instancia de FastAPI creada ahí (la variable que contiene la app).

**Host**

```text
0.0.0.0
```

"Escuchá en todas las interfaces de red". Si usáramos `127.0.0.1` (solo local), el contenedor se escucharía a sí mismo y nadie de afuera podría entrar. Con `0.0.0.0` aceptamos conexiones de cualquier interfaz — necesario para que el mapeo de puertos del host funcione.

**Puerto**

```text
8000
```

El puerto TCP donde uvicorn escucha, que debe coincidir con el EXPOSE y con el healthcheck. **Nota:** TCP es el protocolo de transporte de internet — el "servicio de correo certificado" que garantiza que los datos llegan completos y en orden.

#### ¿Por qué formato JSON?

Dos formas de escribir CMD:

- `CMD ["uvicorn", "app.main:app", ...]` — **exec form** (formato exec): cada argumento separado, entre corchetes, como una lista JSON.
- `CMD uvicorn app.main:app ...` — **shell form** (formato shell): el comando completo como texto.

El problema del shell form: Docker ejecuta el comando a través de un shell intermedio (`/bin/sh -c`), y ese shell se queda como PID 1, tragándose las señales del sistema. **Nota:** una "señal" (signal) es un aviso que el sistema operativo manda a un proceso; la más importante es **SIGTERM**, que significa "apagate con elegancia". Cuando Docker detiene un contenedor, le manda SIGTERM esperando que la app cierre todo limpio.

Con exec form, `uvicorn` es el PID 1 y **recibe las señales directamente**: apagado limpio, cero procesos zombie, cero datos a medio escribir. Por eso el exec form es la práctica correcta.

---

## 📋 Paso 2: Entender docker-compose.yml

Con la imagen entendida, pasamos al archivo que arma el sistema completo. Abrimos `docker-compose.yml` en el editor:

```yaml
services:

  api:
    build: ./api
    ports:
      - "8000:8000"
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    env_file: .env
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres-data:

networks:
  backend:
```

Este archivo es el "plano del sistema": dos servicios (la API y la base), una red privada, un volumen persistente, y reglas de arranque y reinicio. Vamos campo por campo, sin saltarnos ninguno.

### `services:`

`services:` es la clave raíz de cualquier archivo docker-compose.yml — la sección donde se definen los contenedores. Sin `services:`, Compose no sabe qué debe crear. Es la entrada obligatoria de toda la historia.

### Servicio `api`

```yaml
  api:
    build: ./api
    ports:
      - "8000:8000"
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend
    restart: unless-stopped
```

- `api:` — el nombre del servicio. Compose lo usa como identificador del contenedor **y como nombre de host en la red interna**: así como la app pregunta por `db`, el servicio `db` podría hablar con `api` usando solo el nombre `api`. Eso es el DNS interno que vimos en el guion anterior, funcionando.
- `build: ./api` — "construí la imagen usando el Dockerfile que está en la carpeta `./api`". Compose ejecuta `docker build` automáticamente durante `docker compose up`; no hay que correrlo a mano.
- `ports: - "8000:8000"` — el mapeo de puertos que ya conocemos: el 8000 de nuestra máquina (izquierda) redirige al 8000 del contenedor (derecha). Sin esto, aunque uvicorn escuche adentro, el puerto solo sería visible dentro de la red de Docker — ni el navegador ni `curl` podrían entrar.
- `env_file: .env` — carga las variables de entorno desde un archivo `.env` que vive al lado del compose (lo creamos en el Paso 3). **Nota:** `.env` es un archivo de texto con pares `CLAVE=valor` (una variable por línea), típicamente con credenciales. Por eso no está en el repositorio: se crea localmente a partir del template.
- `depends_on:` con `condition: service_healthy` — define el orden de arranque. Sin `condition`, `depends_on` solo espera a que el otro contenedor esté *running* (vivo como proceso). Con `service_healthy`, espera hasta que el healthcheck de `db` devuelva `healthy` — o sea, espera a que PostgreSQL esté **listo para aceptar conexiones**. Es la diferencia entre "la caja está encendida" y "el servicio adentro está operativo". Sin esto, la API intentaría conectarse a la base antes de tiempo, fallaría y entraría en un bucle de reinicios.
- `networks: - backend` — conecta el servicio a la red `backend` (declarada al final del archivo). Todos los servicios en la misma red se ven entre sí por nombre. Adentro del código Python, la conexión a la base usa el nombre del servicio como host: `postgresql://appuser:password@db:5432/appdb` — Docker resuelve `db` a la IP interna automáticamente.
- `restart: unless-stopped` — la política de reinicio: "reiniciá el contenedor automáticamente si se detiene por cualquier causa (crash, error, proceso que termina), excepto si el usuario lo detuvo explícitamente con `docker compose stop` o `docker compose down`". Es la autosanación del sistema: si la app crashea a las 3 AM, Docker la levanta sola.

### Servicio `db`

```yaml
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    env_file: .env
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 5s
      retries: 5
```

- `db:` — el nombre del servicio para PostgreSQL; la API le habla por este nombre.
- `image: postgres:16-alpine` — ojo a la diferencia con `build`: `api` se **construye** desde un Dockerfile; `db` se **descarga** ya construida desde Docker Hub. `postgres:16-alpine` es PostgreSQL versión 16 sobre Alpine Linux, el sistema base ultraligero — resultado: una imagen de ~200MB en vez de ~400MB.
- `restart: unless-stopped` — misma política de autosanación.
- `env_file: .env` — mismo archivo de variables. PostgreSQL usa tres variables específicas: `POSTGRES_USER` (usuario administrador), `POSTGRES_PASSWORD` (su contraseña) y `POSTGRES_DB` (la base de datos que crea en el primer arranque).
- `volumes: - postgres-data:/var/lib/postgresql/data` — el named volume que ya conocemos: los datos de la base viven en el disco del host, no en el contenedor. Si el contenedor muere y se recrea, los datos sobreviven. Sin este volumen, cada `docker compose down && docker compose up` borraría la base entera.
- `healthcheck:` — la verificación de salud. Detalle fino: como `db` usa una imagen ya construida (sin Dockerfile propio), el HEALTHCHECK **no puede** ir en un Dockerfile; por eso va declarado acá, en el compose.
- `test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]` — el comando de verificación. `CMD-SHELL` ejecuta el comando a través de un shell (`/bin/sh -c`). `pg_isready` es una herramienta de PostgreSQL que responde a la pregunta "¿está el servidor listo para recibir conexiones?" — devuelve 0 si sí, 1 si no. `-U appuser` indica el usuario y `-d appdb` la base a verificar.
- `interval: 5s`, `timeout: 5s`, `retries: 5` — chequeo cada 5 segundos, con hasta 5 segundos de espera, y 5 fallos consecutivos para declarar `unhealthy`. Como `db` es la base (la pieza más lenta en arrancar), chequea más seguido que la API.

### `volumes:`

```yaml
volumes:
  postgres-data:
```

Sección raíz donde se **declaran** los volúmenes con nombre. `postgres-data:` declara el volumen: Docker crea un directorio en el host (en `/var/lib/docker/volumes/`) y lo asocia al nombre. Cualquier servicio que monte `postgres-data:/ruta` usa ese mismo directorio. Declararlo aquí es buena práctica (aunque Compose lo crearía igual implícitamente): el archivo queda explícito, y nadie tiene que adivinar qué volúmenes usa el sistema.

### `networks:`

```yaml
networks:
  backend:
```

Otra sección raíz, esta vez para declarar redes. `backend:` declara una red tipo **bridge** (el puente virtual que ya conocemos, el tipo por defecto): una red aislada donde **solo** los servicios conectados a `backend` pueden verse. Los contenedores de otras redes (o sin red) ni siquiera saben que esto existe. Aislamiento por diseño: la API y la base se hablan entre ellas, y nada más.

---

## 🚀 Paso 3: Desplegar

Llegó la hora de la verdad: levantar el stack. **Todos los comandos de este paso se ejecutan en la terminal del Codespace, dentro del directorio `lab-01-api-db/`** — si los corremos desde otra carpeta, Compose no encuentra `docker-compose.yml` y no sabe qué levantar.

**Nota:** "stack" es el nombre informal del sistema completo definido por un compose: todos sus servicios, redes y volúmenes juntos.

### Crea el archivo .env

Antes de levantar nada, creamos el archivo `.env` en la carpeta del lab (en el editor del Codespace), con este contenido:

```bash
DATABASE_URL=postgresql://appuser:apppass@db:5432/appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=apppass
POSTGRES_DB=appdb
```

- `DATABASE_URL` — la URL de conexión que usa la **API** para llegar a la base. Desglose: `postgresql://` es el protocolo, `appuser:apppass` es usuario:contraseña, `db` es el host (el nombre del servicio, que el DNS de Docker resuelve), `5432` es el puerto estándar de PostgreSQL y `appdb` la base.
- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` — las variables que usa **PostgreSQL** en su primer arranque para crear el usuario, la contraseña y la base.

Fíjense en el patrón: un mismo archivo alimenta a ambos servicios — la API lee `DATABASE_URL`, la base lee las `POSTGRES_*`. Y un detalle de seguridad que no admite discusión: **nunca committear `.env`**, porque contiene credenciales (las contraseñas). Por eso no aparece en el árbol de archivos del repositorio: es local, es secreto. (Conviene que `.gitignore` lo tenga en la lista, como vimos en el guion anterior.)

### Levantar el stack

```bash
docker compose up -d
```

Un comando, y se armó todo el sistema. Repasemos el `-d` de *detached*: los servicios corren en segundo plano y la terminal queda libre.

#### Qué ocurre internamente

Detrás de ese único comando hay una secuencia larga, y conviene tenerla en la cabeza:

1. Lee `docker-compose.yml` — el plano del sistema.
2. Lee `.env` — las credenciales.
3. Construye la imagen de `api` — ejecuta el Dockerfile multi-stage (builder → runner).
4. Descarga `postgres:16-alpine` — de Docker Hub.
5. Crea la red `backend` — el puente privado.
6. Crea el volumen `postgres-data` — el disco persistente.
7. Arranca PostgreSQL — y empieza su healthcheck.
8. Espera el healthcheck — `condition: service_healthy` en acción: no pasa hasta que la base esté operativa.
9. Arranca la API — recién cuando la base está lista.

Nueve pasos automáticos por un solo comando. Eso es la orquestación: el orden no lo recordamos nosotros, lo declara el archivo.

### Verificar estado

```bash
docker compose ps
```

`docker compose ps` lista los servicios del stack con su estado. Deben aparecer ambos:

- `running` — el contenedor está corriendo.
- `healthy` — el healthcheck pasó.

Si vemos `starting`, hay que esperar unos segundos: la base tarda en arrancar y pasar su chequeo. Si vemos `unhealthy` o `exited`, algo falló y vamos a usar el siguiente comando para ver qué.

### Ver logs

```bash
docker compose logs
```

Los **logs** (registros de actividad) de todos los servicios del stack, mezclados en orden. Es el primer lugar donde mirar cuando algo falla.

```bash
docker compose logs api
```

Y así vemos solo los logs de un servicio específico. Si la API se quejó de la conexión a la base, acá aparece el error exacto.

> Recordatorio: todos los comandos `docker compose` se ejecutan desde `lab-01-api-db/`. Desde cualquier otra carpeta, Compose no encuentra el archivo.

---

## ❤️ Paso 4: Verificar healthchecks

El stack está arriba. Ahora viene la parte profesional: **verificar que el sistema está sano de verdad**, no solo "encendido".

### Encontrar el nombre del contenedor

Un detalle práctico que confunde siempre: Compose no llama a los contenedores `api` y `db` a secas. Les asigna nombres automáticos con el formato `<directorio>_<servicio>_<número>` — por ejemplo `lab-01-api-db-api-1` y `lab-01-api-db-db-1` (el número es para soportar varias réplicas del mismo servicio).

Para ver los nombres reales:

```bash
docker compose ps --format 'table {{.Name}}\t{{.Status}}'
```

El `--format` personaliza la salida: `{{.Name}}` es el nombre del contenedor y `{{.Status}}` su estado, separados por un tabulador. Es una plantilla de Go, el lenguaje en el que está escrito Docker — no hace falta dominarla, solo copiar el patrón.

De ahora en más, los comandos de `docker inspect` usan esos nombres completos. Si en nuestra máquina difieren, los reemplazamos en los comandos siguientes.

### Estado actual

```bash
docker inspect lab-01-api-db-api-1 --format '{{.State.Health.Status}}'
```

**Nota:** `docker inspect` es el "rayos X" de Docker: devuelve todos los metadatos de un contenedor en formato JSON. El `--format` extrae solo un campo: `{{.State.Health.Status}}` — "el estado de salud, dentro del estado, dentro del contenedor".

Los tres resultados posibles:

- `healthy` — el healthcheck pasó correctamente. Todo bien.
- `starting` — Docker todavía está ejecutando los chequeos iniciales; esperar unos segundos y reintentar.
- `unhealthy` — el healthcheck falló 3 veces seguidas. Algo está mal de verdad.

### Ver historial de healthchecks

```bash
docker inspect lab-01-api-db-api-1 --format '{{json .State.Health}}' | python3 -m json.tool
```

Acá `{{json .State.Health}}` convierte la estructura de salud completa a JSON, y el pipe `|` lo pasa a `python3 -m json.tool`, un formateador que imprime el JSON con sangría para que sea legible.

**Nota:** el pipe `|` conecta la salida de un comando con la entrada de otro — es un conector entre comandos que ya usamos en el Level 0.

La salida muestra un array `Log` con el historial de cada chequeo:

- `Start` — cuándo empezó el chequeo.
- `End` — cuándo terminó.
- `ExitCode` — 0 si fue exitoso, 1 si falló.
- `Output` — la salida del comando de salud.

Si el historial está vacío, el healthcheck todavía no se ejecutó: esperar hasta 30 segundos (el `--interval=30s` del HEALTHCHECK de la API) y reintentar. Ver el historial de chequeos con sus códigos de salida es la forma definitiva de saber si un "unhealthy" fue un tropiezo o una enfermedad real.

---

## 🌐 Paso 5: Probar la API

El sistema está sano. Ahora lo ponemos a trabajar. Todo este paso se hace en la terminal del Codespace con **curl** — el "navegador de terminal": un comando que hace pedidos HTTP y muestra la respuesta en texto.

**Nota:** HTTP es el protocolo de la web: el idioma en que los clientes (navegadores, curl, otras APIs) piden cosas a los servidores. Y un **JSON** es el formato en que las APIs modernas devuelven datos: texto estructurado con claves y valores que tanto humanos como máquinas pueden leer.

### Healthcheck

```bash
curl http://localhost:8000/health
```

Pedido de lectura al endpoint de salud. La respuesta esperada:

```json
{"status":"ok"}
```

"Estoy viva" — el mismo endpoint que usa el healthcheck interno, ahora probado desde afuera.

### Crear un item

```bash
curl -s -X POST http://localhost:8000/items \
  -H "Content-Type: application/json" \
  -d '{"name": "Mi primer item", "description": "Creado desde el lab"}'
```

Desglose de las banderas, porque este comando tiene varias nuevas:

- `-s` (*silent*) — no mostrar barras de progreso ni mensajes extra; solo la respuesta.
- `-X POST` — el método HTTP. **Nota:** POST es el verbo HTTP para "crear algo nuevo" (GET es "leer", DELETE es "borrar"). Una API REST usa estos verbos para expresar las operaciones del CRUD.
- `-H "Content-Type: application/json"` — un **header** (cabecera HTTP): metadatos del pedido. Este le dice a la API "lo que sigue va en formato JSON".
- `-d '{"name": ..., "description": ...}'` (*data*) — el cuerpo del pedido: los datos del item que creamos, en JSON. La comilla simple del inicio protege el JSON de los espacios.

La API crea el item en la base de datos y responde con el objeto creado (con su `id` asignado).

### Listar items

```bash
curl -s http://localhost:8000/items | python3 -m json.tool
```

GET (el pedido por defecto de curl) a `/items`, con el formateador JSON de Python para que la respuesta se vea linda:

```json
[
  {
    "id": 1,
    "name": "Mi primer item",
    "description": "Creado desde el lab"
  }
]
```

Un array (lista) con los items guardados. La API leyó de la base de datos, y la base respondió.

### Obtener por ID

```bash
curl -s http://localhost:8000/items/1 | python3 -m json.tool
```

Ahora la URL lleva un **parámetro de ruta**: `/items/1` significa "el item con id 1". Es el patrón clásico de REST: el recurso (`items`) más el identificador del ejemplar (`1`).

### Eliminar

```bash
curl -s -o /dev/null -w "%{http_code}" -X DELETE http://localhost:8000/items/1
```

Este tiene truco: `-o /dev/null` descarta el cuerpo de la respuesta (no nos interesa), y `-w "%{http_code}"` imprime solo el **código de estado HTTP** — el número con que el servidor responde si todo salió bien o mal.

La respuesta esperada:

```text
204
```

**Nota:** el `204` es "No Content": la operación se ejecutó con éxito pero no hay nada que devolver (el item fue borrado, no hay datos de respuesta). La familia de códigos: 2xx = éxito, 4xx = error del cliente, 5xx = error del servidor.

Si todo esto funciona, nuestra API CRUD está operativa: crear, leer, listar y borrar — el ciclo completo, contra una base de datos real, dentro de contenedores.

---

## 💾 Paso 6: Validar persistencia

Este es el momento de demostrar que la teoría de los volúmenes es verdad. La prueba más contundente de que los datos viven afuera del contenedor.

### Crear datos y detener PostgreSQL

```bash
curl -s -X POST http://localhost:8000/items \
  -H "Content-Type: application/json" \
  -d '{"name": "Persistente", "description": "Debe sobrevivir"}'

docker compose stop db
```

Creamos un item con nombre irónico: "Persistente" — "Debe sobrevivir". Y después apagamos la base de datos con `docker compose stop db`.

**Nota:** `stop` apaga el contenedor sin eliminarlo: el contenedor queda en estado `exited` pero existe, y sus volúmenes montados quedan intactos. Es "apagar la máquina", no "formatear el disco".

### Intentar consultar datos

```bash
curl http://localhost:8000/items
```

La API sigue viva, pero su base está apagada. Respuesta esperada:

```text
Internal Server Error
```

**Nota:** este es el error 500 de HTTP — "el servidor falló". La API no puede hablar con la base porque la base no está, así que el pedido explota. Es el comportamiento correcto: la API nos avisa que algo de abajo está roto, en vez de devolver datos inventados.

### Levantar PostgreSQL nuevamente

```bash
docker compose start db
```

`start` arranca el contenedor que estaba apagado. Ojo: es `start`, no `up` — no recreamos nada, solo encendemos lo que estaba detenido.

### Verificar datos

```bash
curl -s http://localhost:8000/items | python3 -m json.tool
```

Y acá está el momento de la verdad: el item "Persistente" debería seguir ahí. La base se apagó, se prendió, y los datos siguen — gracias al volumen.

### ¿Por qué sobreviven?

La cadena completa del porqué, en orden:

1. PostgreSQL guarda sus datos en `/var/lib/postgresql/data`.
2. Esa ruta está montada en el volumen `postgres-data`.
3. El volumen vive **fuera** del contenedor, en el disco del host.
4. `stop` no elimina volúmenes (solo apaga el contenedor).
5. `start` vuelve a montar el mismo volumen en el mismo contenedor.
6. PostgreSQL arranca y continúa donde quedó, leyendo los datos que nunca se movieron.

> Y la advertencia que cierra el punto: si en lugar de `stop`/`start` ejecutáramos `docker compose down -v`, los datos se perderían — porque `-v` elimina también los volúmenes. Ese comando es el "formateo total"; se usa con consciencia plena de lo que borra.

---

## 🔍 Paso 7: Debugging

Ningún sistema real funciona siempre a la primera. Saber **mirar adentro** del sistema es la habilidad que distingue al que opera de verdad. Este paso es la caja de herramientas del diagnóstico.

Recuerden: algunos comandos necesitan el nombre exacto del contenedor; si en nuestra máquina no es `lab-01-api-db-api-1`, lo reemplazamos por el que vimos con `docker compose ps`.

### Ver logs en tiempo real (API)

```bash
docker logs -f lab-01-api-db-api-1
```

`docker logs` muestra los logs del contenedor y funciona desde cualquier directorio (usa el nombre del contenedor, no la carpeta del proyecto). `-f` (*follow*) mantiene la ventana abierta mostrando los logs nuevos a medida que aparecen — como ver una serie en directo. Ctrl+C para salir.

### Últimas 20 líneas (PostgreSQL)

```bash
docker logs --tail 20 lab-01-api-db-db-1
```

`--tail 20` muestra solo las últimas 20 líneas — el final de la historia, que es lo que suele importar cuando algo falló. Sin `--tail`, `docker logs` devuelve el historial completo desde el arranque (que puede ser largo).

### Obtener IP del contenedor en la red backend

```bash
docker inspect lab-01-api-db-api-1 --format '{{(index .NetworkSettings.Networks "lab-01-api-db_backend").IPAddress}}'
```

Esto es `docker inspect` con una plantilla más elaborada: navega por los metadatos hasta las redes del contenedor, busca la red llamada `lab-01-api-db_backend` (el nombre que Compose le puso a la red `backend` del proyecto) y extrae su `IPAddress`. Útil para ver con nuestros propios ojos la IP interna que el DNS de Docker resuelve cuando decimos `db`.

### Ver variables de entorno del contenedor

```bash
docker inspect lab-01-api-db-api-1 --format '{{json .Config.Env}}' | python3 -m json.tool
```

`{{json .Config.Env}}` convierte la lista de variables de entorno del contenedor a JSON, y el formateador las imprime con sangría. Acá podemos **verificar con nuestros propios ojos** que las variables del `.env` llegaron al contenedor (incluida la `DATABASE_URL`). ¿Sospecha de que falta una variable? Este comando la confirma o la descarta.

### Ver consumo de recursos en vivo

```bash
docker stats lab-01-api-db-api-1 --no-stream
```

`docker stats` muestra CPU, memoria, red y disco de los contenedores — los datos salen directamente de los cgroups que vimos en la Parte 2. `--no-stream` imprime una sola medición y termina (sin el flag, se queda actualizando en vivo como el `top` de Linux).

### Entrar al contenedor de la API

```bash
docker exec -it lab-01-api-db-api-1 sh
```

`docker exec` ejecuta un comando dentro de un contenedor que ya está corriendo. `-i` (*interactive*) mantiene la entrada abierta, `-t` (tty) nos da una terminal, y `sh` es el shell disponible en las imágenes Alpine. Resultado: estamos **adentro** del contenedor, con una consola. Para salir: `exit` o Ctrl+D.

**Nota:** este comando hay que usarlo con criterio: para **mirar y diagnosticar** está perfecto (el guion anterior lo marcó como anti-patrón cuando se usa para "arreglar" a mano). Mirar está bien; arreglar se hace en el Dockerfile.

### Consultar PostgreSQL directamente

```bash
docker exec -it lab-01-api-db-db-1 psql -U appuser -d appdb -c "SELECT * FROM items;"
```

**Nota:** `psql` es el cliente oficial de PostgreSQL: la herramienta para hablarle a la base en su propio idioma. `-U appuser` el usuario, `-d appdb` la base, `-c "SELECT * FROM items;"` ejecuta una consulta **SQL** (el lenguaje de las bases relacionales — esta consulta dice "traeme todas las filas de la tabla items") y termina.

¿Para qué saltarse la API y hablarle directo a la base? Para verificar datos sin la intermediación de la app: si la API responde mal, así distinguimos si el problema es de la API o de los datos. Es el nivel más bajo del diagnóstico.

---

## 🧹 Paso 8: Limpiar

Todo sistema se termina, y en Docker la limpieza es parte de la disciplina. Dos comandos con una diferencia que hay que tener clarísima.

### Detener y eliminar contenedores

```bash
docker compose down
```

`down` detiene los contenedores y los elimina (junto con la red del proyecto, que Compose creó). Lo importante: **los volúmenes permanecen**. Los datos quedan en el disco, esperando el próximo `docker compose up`.

### Eliminar también los volúmenes

```bash
docker compose down -v
```

Con `-v` (volumes) eliminamos también los volúmenes. ⚠️ Esto borra **permanentemente** los datos: no hay papelera de reciclaje, no hay vuelta atrás. `down` para apagar conservando, `down -v` para el formateo total — cuando queremos empezar de cero absoluto.

### Script incluido

```bash
./scripts/cleanup.sh
```

El repositorio ya trae un script que automatiza la limpieza completa:

```bash
docker compose down -v
```

junto con verificaciones adicionales. Los scripts del lab (`setup.sh`, `validate.sh`, `cleanup.sh`) son el embrión de lo que en el Level 5 vamos a automatizar con CI/CD: el ciclo de vida del sistema convertido en comandos repetibles.

---

## ✅ Criterios de éxito

¿Cómo sabemos que el lab salió bien? Con una tabla de criterios verificables — cada fila es una prueba concreta:

| Criterio | Validación |
|---|---|
| API responde | `curl /health` → `{"status":"ok"}` |
| CRUD funcional | POST, GET, DELETE items funcionan |
| Persistencia | Datos sobreviven a `docker compose stop/start db` |
| Usuario no root | `docker exec api whoami` → `appuser` |
| Healthchecks verdes | `docker inspect api` → `"healthy"` |
| Red aislada | Contenedores solo se ven en red `backend` |

Repasemos los dos que tienen comandos nuevos:

- `docker exec api whoami` → `whoami` responde "¿quién soy?" — imprime el usuario actual del proceso. Si devuelve `appuser`, la buena práctica del Dockerfile se está cumpliendo en producción.
- "Red aislada" — los contenedores se ven entre sí por nombre en `backend`, y nada de afuera los alcanza.

Los seis criterios son el checklist del profesional: no "parece que funciona", sino "está demostrado que funciona".

---

## 💡 Preguntas de reflexión

Cinco preguntas para pensar — y aquí van las respuestas en palabras propias, porque entenderlas es el objetivo real del lab.

### 1.

**¿Por qué separar las dependencias del build (`builder`) del runtime (`runner`)? ¿Qué pasa si usas una sola etapa?**

Porque construir software deja basura: compiladores, cachés de pip, archivos temporales — cientos de megabytes que la app no usa en producción. Con una sola etapa, toda esa basura quedaría dentro de la imagen final: deploys más lentos, más superficie de ataque, más disco ocupado. Con multi-stage, el builder fabrica y se descarta; el runner copia solo el resultado limpio (el virtualenv). Imagen final chica, segura y veloz.

### 2.

**¿Qué ocurriría si `depends_on` no tuviera `condition: service_healthy`? ¿Arrancaría la API antes que la base de datos?**

Sí, casi seguro. `depends_on` simple solo espera a que el contenedor de `db` esté *running* — "el proceso existe". Pero un PostgreSQL recién arrancado tarda en estar listo para aceptar conexiones. La API arrancaría, intentaría conectarse, fallaría, y entraría en bucle de reinicios (agravado por `restart: unless-stopped`). Con `service_healthy`, la API espera a que la base pase su healthcheck — listo para recibir — y arranca limpio.

### 3.

**Sin el flag `--chown` en `COPY --from=builder`, el contenedor falla al arrancar. ¿Por qué?**

Porque la etapa builder corrió como root, así que los archivos del virtualenv copiados quedan siendo de root. Pero la etapa runner declara `USER appuser`, y `appuser` no tiene permiso sobre archivos ajenos: ni para leer las librerías ni para ejecutar los binarios. La app arranca y explota por permisos. `--chown=appuser:appuser` transfiere la propiedad de los archivos al usuario que va a ejecutarlos.

### 4.

**¿Cuál es la diferencia práctica entre `docker compose down` y `docker compose down -v`? ¿Cuándo usarías cada uno?**

`down` detiene y elimina contenedores y red, conservando los datos. `down -v` hace lo mismo y además borra los volúmenes — los datos desaparecen para siempre. `down` es el apagado normal: desarrollo diario, conservar la base. `down -v` es el formateo: ambiente de pruebas que queremos reiniciar desde cero, o limpieza final del lab.

### 5.

**En el Dockerfile se usa `CMD` en formato JSON (`["uvicorn", ...]`). ¿Qué problema evitarías respecto al formato shell (`CMD uvicorn ...`)?**

El formato shell mete un intérprete de shell en el medio, y ese shell queda como PID 1 tragándose las señales. Cuando Docker manda SIGTERM para apagar el contenedor, la app nunca lo recibe: apagados que se cuelgan, procesos zombie, datos a medio escribir. El formato exec (JSON) hace que `uvicorn` sea el PID 1 y reciba las señales directamente: apagado limpio y predecible.

---

## 🏁 Cierre

Este no es un lab más. Hagamos el inventario de lo que acabamos de hacer:

- **Construir una imagen Docker profesional con multi-stage** — dos etapas, usuario no root, healthcheck, exec form, imagen final mínima.
- **Orquestar dos servicios con redes, volúmenes y healthchecks** — API + base de datos, en su red privada `backend`, con datos persistentes, arranque coordinado y autosanación.
- **Validar persistencia de datos real** — apagamos la base, la encendimos, y los datos seguían ahí.
- **Debuggear como se hace en producción** — logs, inspect, exec, stats: mirar adentro del sistema cuando algo falla.

Esto es exactamente lo que separa a alguien que "sabe Docker" de alguien que puede operar plataformas cloud-native: no es conocer comandos, es haber construido un sistema completo y haberlo visto funcionar, fallar y persistir.

El README dentro del lab tiene más detalles y ejercicios adicionales si queremos profundizar.

Y el siguiente paso del camino está marcado: **orquestación con Kubernetes** — donde todo lo que aprendimos a coordinar a mano con Compose pasa a coordinarlo una plataforma entera.

Fin del guion.
