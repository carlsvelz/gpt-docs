# Guion de texto — Level 1: Docker, el sistema de producción en miniatura

> **Cómo usar este guion:** cada título de este archivo es idéntico a un título de `level-1/docs/section-2.md`. Se leen lado a lado: la section muestra los comandos y la teoría oficial; este guion explica lo mismo con nuestras propias palabras, en estilo for dummies, definiendo cada término técnico la primera vez que aparece. Si en algún momento algo suena a chino, la definición está un poco más arriba, en el bloque anterior.

---

# 🐳 Docker: La Abstracción del Sistema de Producción

Bienvenidos al cierre del primer nivel del roadmap. En el Level 0 aprendimos a movernos en Linux, a escribir scripts en Bash y a versionar con Git y GitHub: las bases operativas. Ahora cerramos el Level 1, el nivel de los **contenedores** con Docker.

Una idea para llevarse desde el arranque, porque todo lo demás cuelga de acá:

> Un contenedor es un proceso de Linux aislado, con su propio filesystem, su propia red y sus propios límites de recursos. No es una máquina virtual, no es un mini servidor, no es magia: es un proceso normal de Linux al que le damos superpoderes de aislamiento.

**Nota:** un "proceso" es un programa en ejecución. Cuando abrimos una app, el sistema operativo crea un proceso: un bloque de memoria, CPU y archivos asociado a esa app. Un contenedor es exactamente eso, pero encerrado en paredes virtuales.

## Parte 3 · Final

Esta es la parte final de nuestro mini-curso de Docker, y antes de meternos de lleno, repasemos de dónde venimos:

- En la **Parte 1** vimos el problema de fondo: en producción no se despliega código, se despliega un sistema operativo en ejecución. "Funciona en mi máquina" pasa porque en cada máquina hay un sistema diferente (otro runtime, otras librerías, otra configuración). Docker resuelve eso empaquetando aplicación, runtime, librerías y configuración en una sola unidad portable: la **imagen**. Y vimos la arquitectura: el cliente `docker` manda pedidos al **daemon** (`dockerd`), que es el cerebro que realmente crea contenedores, imágenes, redes y volúmenes, hablando con piezas de bajo nivel como `containerd` y `runc`. También vimos que un contenedor **no es una VM**: comparte el kernel del host (el corazón del sistema operativo), y usa tres mecanismos del kernel de Linux: **namespaces** (paredes que aíslan procesos, red y filesystem), **cgroups** (techos que limitan CPU y memoria) y un **filesystem por capas** (imagen de solo lectura + capa de escritura temporal).
- En la **Parte 2** abrimos el motor: las imágenes son plantillas inmutables construidas en capas que se cachean y se reutilizan; los namespaces engañan a cada contenedor para que crea que está solo; los cgroups ponen límites reales de recursos (y si la memoria se acaba, el kernel mata el contenedor con un OOM Kill); y el proceso principal del contenedor es el PID 1 de su mundo, con responsabilidades especiales de apagado limpio.

Hoy cerramos el sistema completo con los temas que separan a quien ejecuta `docker run` de quien construye plataformas: redes y comunicación entre contenedores, persistencia de datos, la receta de imágenes (Dockerfile), la orquestación de sistemas completos (Docker Compose) y las reglas de producción.

---

## 🌐 4.3 DNS interno y comunicación contenedor-contenedor

Pregunta práctica: tenemos una aplicación web y una base de datos, cada una en su contenedor. ¿Cómo se hablan?

Respuesta corta: por una **red** que Docker crea entre ellos, usando algo que suena sofisticado pero es simple: un **DNS interno**.

**Nota:** DNS (Domain Name System) es el sistema de nombres de dominios. En internet funciona como una agenda telefónica: el navegador pregunta "¿dónde vive google.com?" y el DNS responde con una dirección IP (la dirección numérica de una máquina en una red, algo así como 142.250.64.14). Docker hace lo mismo, pero a escala micro: el contenedor pregunta "¿dónde vive `db`?" y el DNS interno responde con la IP interna de ese contenedor.

¿Por qué importa? Porque las **IP cambian** cada vez que un contenedor nace o muere. Si la app guardara la IP de la base de datos como un número fijo, se rompería en cada reinicio. En cambio, los **nombres no cambian**: el contenedor se llama `db`, punto. La app siempre pregunta por `db` y el DNS resuelve la IP actual. Esto se llama **resolución por nombre**, y es la base de una arquitectura de **microservicios** (muchos servicios pequeños e independientes que se comunican entre sí por la red).

El truco técnico: cada red **bridge personalizada** trae su propio DNS embebido, que vive en la IP interna `127.0.0.11` y escucha en el puerto 53 (el puerto estándar de DNS).

**Nota:** un "puerto" es un canal numerado dentro de una IP. Piensen en la IP como la dirección de un edificio y el puerto como el número de departamento. El puerto 53 es de DNS, el 80 de HTTP (páginas web), el 5432 de PostgreSQL (base de datos). Y un "bridge" es un puente virtual: una red interna privada donde solo entran los contenedores que se conectan a ella.

### 🧠 Cómo funciona

El flujo completo, paso a paso, con la aplicación y la base de datos:

1. El contenedor `app` quiere hablar con la base de datos y le pregunta al DNS de Docker: "¿dónde vive `db`?"
2. El DNS interno responde: "en la IP interna 10.10.0.3".
3. `app` manda su tráfico a esa IP, en el puerto 5432 (el puerto donde la base de datos escucha).
4. La base de datos recibe, procesa la consulta y responde.

Noten el detalle importante: `app` nunca le habla a `db` por IP fija — pregunta por el **nombre** y el DNS hace el trabajo de traducirlo.

Todo esto ocurre **dentro de la red bridge personalizada**: los contenedores que están en esa red se ven entre sí por nombre y pueden comunicarse automáticamente. Los que están afuera, no los ven. Eso es aislamiento y orden al mismo tiempo: cada sistema vive en su propia red privada, sin chocar con los demás.

### 🧪 Ejemplo práctico

En la terminal del Codespace, probamos esto en vivo:

```bash
# Crear red personalizada
docker network create app-net

# Ejecutar contenedores
docker run -d --name web --network app-net nginx:alpine
docker run -d --name api --network app-net nginx:alpine

# Entrar a un contenedor temporal
docker run -it --rm --network app-net alpine:3.19 sh

# ping web
# ping api
# nslookup web
```

Vamos línea por línea, porque hay varios términos nuevos:

- `docker network create app-net` → crea una red personalizada llamada `app-net`. Es nuestra sala privada: solo los contenedores que se conecten a `app-net` podrán verse entre sí.
- `docker run -d --name web --network app-net nginx:alpine` → corre un contenedor llamado `web`, en **segundo plano** (`-d` viene de *detached*: la terminal no se queda pegada al contenedor, recuperamos el prompt), conectado a `app-net`. La imagen es `nginx:alpine`. **nginx** es un servidor web famoso (de los más usados de internet), y **alpine** es una distribución de Linux ultraligera, pensada para ocupar pocos megabytes. El formato `imagen:etiqueta` indica la versión: acá la etiqueta es `alpine`. Lo mismo para `api`.
- `docker run -it --rm --network app-net alpine:3.19 sh` → corre un contenedor temporal e **interactivo**: `-i` (interactivo) mantiene la entrada abierta, `-t` (terminal falsa) nos da una consola dentro del contenedor, `--rm` (remove) lo elimina automáticamente al salir. Adentro ejecutamos `sh`, el intérprete de comandos básico de Linux (una terminal chiquita). Este contenedor es nuestro "observador": se conecta a la red solo para mirar cómo se comunican los demás.
- `ping web` → **ping** es un comando que manda "¿estás ahí?" a una máquina y espera respuesta. Si responde, la red funciona y la resolución por nombre también: el contenedor pudo encontrar a `web` por su nombre.
- `ping api` → lo mismo contra el segundo contenedor.
- `nslookup web` → **nslookup** significa "consultar el DNS por este nombre": nos muestra la IP interna que el DNS le asignó a `web`. Es la prueba directa de que el DNS interno está resolviendo.

Si los pings responden, acabamos de ver microservicios en acción: contenedores aislados que se encuentran entre sí por nombre, sin IPs fijas, sobre una red privada. Con Ctrl+D salimos del contenedor temporal, y con `--rm` se borra solo.

---

## 💾Persistencia de Datos

### 🧨 El problema

Ahora una de las preguntas más dolorosas de Docker: ¿cuántas veces levantamos una base de datos en un contenedor, trabajamos semanas… y un `docker rm` borró todo?

**Nota:** `docker rm` elimina un contenedor. Parece un comando más, pero en el modelo de Docker es un gesto cotidiano: los contenedores se tiran y se recrean todo el tiempo.

¿Por qué se pierde todo? Por una consecuencia directa de lo que vimos en la Parte 2: los contenedores son **efímeros** (temporales, desechables). Cuando el contenedor desaparece, también desaparece su **capa de escritura** — la única parte del filesystem del contenedor donde se puede escribir. Y ahí vive todo lo que se genera en tiempo de ejecución:

- bases de datos
- logs (los registros de actividad de la aplicación)
- uploads (archivos que los usuarios suben)
- configuraciones dinámicas (ajustes que la app cambia mientras corre)

Todo eso nace en la capa de escritura, y con la capa de escritura muere. El contenedor es un plato de papel: cómodo, reemplazable… y no aguanta lo que le ponemos encima.

### 🗂️ Volúmenes vs Bind Mounts

La solución es sacar los datos del contenedor: que vivan en el disco del host (la máquina donde corre Docker), no adentro. Docker ofrece tres mecanismos, y la tabla de la section es la regla de oro:

| Tipo | ¿Qué es? | ¿Cuándo se usa? |
|---|---|---|
| **Named Volumes** (volúmenes con nombre) | Almacenamiento administrado por Docker, con nombre propio, desacoplado del contenedor | Producción |
| **Bind Mounts** (montajes enlazados) | Una carpeta del host que se conecta dentro del contenedor | Desarrollo |
| **tmpfs** | Almacenamiento en memoria RAM, no en disco | Datos temporales |

**Nota:** "montar" viene del inglés *mount*: conectar un sistema de archivos a un punto del árbol de directorios. Cuando decimos "montar un volumen en `/var/lib/postgresql/data`", queremos decir que esa carpeta, que parece estar dentro del contenedor, en realidad apunta a almacenamiento externo. El contenedor no sabe la diferencia: para él, es su carpeta normal. Y tmpfs (temporary filesystem) guarda datos en RAM, que se borran solos al apagar; sirve para cachés o datos que no queremos en disco.

### ✅ Named Volumes

El ejemplo de producción, en la terminal del Codespace:

```bash
docker volume create mi-datos

docker run -d \
  --name postgres-db \
  -v mi-datos:/var/lib/postgresql/data \
  postgres:15
```

Desglose, parte por parte:

- `docker volume create mi-datos` → crea un volumen con nombre `mi-datos`. Es como crear un disco externo etiquetado, que Docker administra.
- `docker run -d --name postgres-db` → corre en segundo plano un contenedor llamado `postgres-db`. **PostgreSQL** (o "postgres") es un motor de bases de datos relacional (guarda datos en tablas con filas y columnas, como una planilla gigante); la versión es la 15.
- `-v mi-datos:/var/lib/postgresql/data` → la bandera `-v` (de volume) conecta el volumen `mi-datos` con la carpeta `/var/lib/postgresql/data` dentro del contenedor, que es exactamente donde PostgreSQL guarda sus datos. Resultado: el contenedor puede morir mil veces; los datos sobreviven en el volumen, listos para conectar el próximo contenedor al mismo nombre.

#### Ventajas

¿Por qué son la opción de producción?

- **Persistencia real** — los datos sobreviven a los contenedores.
- **Desacoplados del contenedor** — se crean, se listan y se borran por separado; el contenedor es solo un inquilino temporal.
- **Fáciles de migrar** — el volumen se puede copiar o respaldar como una unidad.
- **Docker administra permisos** — no tenemos que pelearnos con dueños de archivos; Docker se encarga.

### 🛠️ Bind Mounts

La contraparte para desarrollo:

```bash
docker run -d \
  --name nginx-dev \
  -v /home/usuario/site:/usr/share/nginx/html:ro \
  -p 8080:80 \
  nginx:alpine
```

Acá la bandera `-v` no apunta a un volumen con nombre, sino a una **carpeta del host**: `/home/usuario/site` se monta en `/usr/share/nginx/html` (la carpeta donde nginx sirve las páginas web). El sufijo `:ro` significa *read only*: el contenedor solo puede leer esa carpeta, no escribir en ella. Y `-p 8080:80` es el **mapeo de puertos**: el puerto 8080 del host redirige al puerto 80 del contenedor. Así abrimos `http://localhost:8080` en el navegador y vemos el sitio.

La magia del bind mount es el **hot reload** (recarga en caliente): editamos el archivo en el host y el cambio aparece al instante en el sitio, sin reconstruir nada, porque el contenedor ve la misma carpeta en vivo. Perfecto para desarrollo.

Pero en producción se vuelve un problema:

- **Problemas de permisos** — los archivos del host pueden pertenecer a un usuario distinto al del contenedor, y las apps no pueden escribir donde deberían.
- **Dependencia del host** — el contenedor deja de ser autónomo: necesita que el host tenga esa carpeta exacta en ese lugar exacto.
- **Baja portabilidad** — lo que funciona en Linux no se comporta igual en Mac o Windows; el sistema pierde su cualidad de "corre igual en todos lados".

Por eso: bind mounts para desarrollar, named volumes para producir.

### 🧠 Principio clave

Que quede grabado, porque es el mismo principio que va a gobernar Kubernetes en el próximo nivel:

> Los contenedores deben ser **inmutables**, **efímeros** y **reemplazables**. Los datos viven afuera.

- **Inmutables** — no se "arreglan" a mano: si cambia algo, se construye una imagen nueva.
- **Efímeros** — pueden morir en cualquier momento sin drama.
- **Reemplazables** — tirar uno y levantar otro es una operación normal.

Y como los datos viven afuera (en volúmenes), reemplazar el contenedor no cuesta nada. Esa es la mentalidad de producción: el contenedor es ganado, no mascota (este concepto vuelve más adelante, en anti-patrones).

---

## 🏗️ Dockerfile

Ya sabemos correr imágenes hechas (nginx, postgres). La pregunta del millón: ¿cómo fabricamos la imagen de **nuestra propia aplicación**?

Con un **Dockerfile**: un archivo de texto con instrucciones que funciona como una **receta reproducible del sistema**. "Reproducible" es la palabra clave: dos personas que sigan la misma receta obtienen exactamente la misma imagen. Se acabó el "en mi máquina funcionaba".

**Nota:** el Dockerfile es un archivo de texto plano sin extensión (se llama `Dockerfile`, así, sin más). Se crea en el editor del Codespace y se usa con el comando `docker build`.

### 🧬 Anatomía básica

Un Dockerfile típico para una aplicación Node.js:

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

Instrucción por instrucción, en lenguaje humano:

| Instrucción | Qué hace | Traducción |
|---|---|---|
| `FROM node:18-alpine` | Elige la imagen base | "Arrancamos desde una imagen que ya trae Node 18 sobre Alpine" — siempre es la primera línea; sin ella no hay nada |
| `WORKDIR /app` | Fija el directorio de trabajo | "Dentro del contenedor, todo lo que sigue ocurre en la carpeta `/app`" — como un `cd` automático y persistente |
| `COPY package*.json ./` | Copia archivos del host al contenedor | "Traé los archivos de dependencias a la carpeta actual". El `*` es un comodín: matchea `package.json` y `package-lock.json` |
| `RUN npm ci --only=production` | Ejecuta un comando durante la construcción | "Instalá las dependencias de la app". `npm` es el gestor de paquetes de Node (el programa que instala librerías de JavaScript); `ci` instala las versiones exactas que declara el archivo de bloqueo; `--only=production` deja afuera las herramientas de desarrollo |
| `COPY . .` | Copia todo el proyecto | "Copiá todo lo que está en la carpeta actual del host hacia `/app`" — el código de la app entra acá |
| `EXPOSE 3000` | Declara el puerto | "Esta app escucha en el puerto 3000" — es una documentación formal del puerto, no publica nada por sí solo |
| `CMD ["node", "server.js"]` | Define el comando de arranque | "Cuando el contenedor inicie, ejecutá `node server.js`" — el proceso que será el PID 1 del contenedor. Solo puede haber un CMD efectivo por imagen |

**Nota:** `package.json` es el archivo que lista las librerías del proyecto y sus versiones — la lista de compras de la app. `package-lock.json` es la versión congelada, con las versiones exactas de todo, para que la instalación sea idéntica en cualquier máquina. Por eso el orden del Dockerfile importa: primero copiamos la lista de compras e instalamos, y recién después copiamos el código. Así, cuando cambiamos solo el código, Docker reutiliza la capa de instalación cacheada (vimos el caché de capas en la Parte 2).

### 🔥 Buenas prácticas

Tres reglas que nos van a salvar en producción.

#### 👤 No usar root

**root** es el superusuario de Linux: puede tocar absolutamente todo en el sistema. Si nuestra app corre como root y el contenedor es comprometido, el atacante tiene poderes de root dentro de él, y de ahí al host hay un paso corto. La práctica correcta: crear un usuario sin privilegios y correr como él:

```dockerfile
RUN adduser -D appuser
USER appuser
```

`adduser -D appuser` crea un usuario nuevo llamado `appuser` (la `-D` lo crea sin contraseña, convención de Alpine para imágenes). `USER appuser` ordena que todos los procesos posteriores corran como ese usuario limitado. Así, aunque alguien entre al contenedor, no tiene poderes de administrador.

#### 🪶 Imágenes pequeñas

```dockerfile
FROM node:18-alpine   # ✅ Mejor
FROM ubuntu:24.04     # ❌ Evita
```

**Alpine** pesa pocos megabytes y trae lo mínimo. **Ubuntu** es un sistema completo: cientos de megabytes, con herramientas que nuestra app nunca usa. Cada MB extra de imagen se traduce en deploys más lentos y en más **superficie de ataque** (más programas instalados = más lugares donde puede haber una vulnerabilidad explotable). Regla: imagen chica, dependencias justas.

#### 🚫 `.dockerignore`

Un archivo que le dice a Docker qué **NO** copiar al contenedor:

```bash
node_modules
.git
.env
Dockerfile
```

- `node_modules` → las dependencias ya se instalan adentro con `RUN npm ci`; copiar las del host sería duplicar cientos de megabytes de basura.
- `.git` → el historial de Git no sirve en tiempo de ejecución y puede filtrar información interna.
- `.env` → las variables de entorno con secretos (contraseñas, claves de API) jamás viajan dentro de una imagen; más adelante vemos la forma correcta.
- `Dockerfile` → la receta no va dentro del producto final.

**Nota:** es el mismo concepto que `.gitignore` del nivel anterior: una lista de exclusiones. Acá le decimos a Docker "al empacar, dejá esto afuera". Es la caja de embarque de la imagen: solo lo que la app necesita para correr, no la mudanza completa.

---

## ⚡ Docker Compose

Hagamos un inventario mental de un sistema real: aplicación web (`app`), base de datos (`postgres`), caché (`redis`), trabajadores en segundo plano (`workers`). Con `docker run` tendríamos que escribir cuatro comandos largos, recordar el orden de arranque, las redes, los volúmenes, los puertos… y repetirlo en cada máquina. Eso deja de escalar mentalmente: se vuelve imposible de mantener.

Ahí entra **Docker Compose**: una herramienta que describe el **sistema completo** — todos los servicios y sus relaciones — en un solo archivo de configuración. Con un comando, levanta todo.

**Nota:** "servicios" es el término de Compose para cada pieza del sistema: web, base de datos, worker, caché. Cada servicio es, en el fondo, uno o más contenedores. Y el archivo de configuración se llama `docker-compose.yml` (o `compose.yml`): "yml" es YAML, un formato de texto para escribir configuraciones que se lee por indentación (espacios, no tabulaciones).

### 🧩 Ejemplo

El ejemplo clásico de la section, en el archivo `docker-compose.yml` que creamos en el editor del Codespace:

```yaml
version: '3.8'

services:

  web:
    build: .

    ports:
      - "3000:3000"

    depends_on:
      - db

  db:
    image: postgres:15

    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

Leamos la estructura de arriba a abajo:

- `version: '3.8'` → la versión del formato de Compose que usamos. No es indispensable hoy, pero aparece en miles de proyectos; conviene saber qué es.
- `services:` → abre la lista de servicios. Debajo de esta clave declaramos cada pieza del sistema.
- `web:` → un servicio llamado `web`, con su configuración:
  - `build: .` → "construí la imagen de este servicio a partir del Dockerfile que está en esta carpeta" (el punto es la carpeta actual del proyecto).
  - `ports: - "3000:3000"` → mapeo de puertos, el mismo concepto que `-p`: el puerto 3000 del host apunta al 3000 del contenedor.
  - `depends_on: - db` → "no arranques `web` hasta que el servicio `db` esté disponible". Declara el orden de arranque y la dependencia entre servicios. **Nota:** un "depende de" no espera a que la base esté lista del todo — solo ordena el arranque; la app igual necesita su propia lógica de espera.
- `db:` → el segundo servicio:
  - `image: postgres:15` → este no se construye: usa directamente la imagen oficial `postgres:15` del **registry** (el almacén público de imágenes; el más conocido es Docker Hub).
  - `volumes: - postgres-data:/var/lib/postgresql/data` → conecta el volumen con nombre `postgres-data` a la carpeta de datos de PostgreSQL, exactamente el mecanismo de persistencia que vimos antes.
- `volumes: postgres-data:` → declara el volumen con nombre a nivel raíz, para que Compose lo cree y lo administre como parte del proyecto.

Con Compose, "la app + la base" dejan de ser comandos sueltos y pasan a ser **un sistema describible**: si lo borramos, podemos reconstruirlo idéntico desde el archivo. Esa declaratividad es la misma idea que después vamos a ver multiplicada en Kubernetes.

### 🛠️ Comandos útiles

En la terminal del Codespace, dentro de la carpeta donde está el `docker-compose.yml`:

```bash
docker compose up -d
```

→ "Levantá todos los servicios en segundo plano". `up` construye lo que haya que construir, descarga las imágenes que falten, crea la red interna del proyecto y arranca todo el sistema. Con `-d` (detached) la terminal queda libre.

```bash
docker compose logs -f
```

→ "Mostrá los logs de todos los servicios en vivo". `logs` muestra los registros de actividad; `-f` (follow) mantiene la terminal siguiendo el flujo, como ver una serie en directo. Para salir: `Ctrl+C`.

```bash
docker compose down -v
```

→ "Bajá y destruí todo lo que creó el compose". ⚠️ Atención: la bandera `-v` (de volumes) **elimina también los volúmenes**, es decir, borra los datos. Si solo queremos apagar el sistema conservando los datos, usamos `docker compose down` sin `-v`.

---

## 🏭 Docker en Producción

Hasta acá, laboratorio. Ahora el salto de mentalidad: ¿qué cambia cuando esto se va a un entorno real, con usuarios, tráfico y adversarios? Acá se separa el laboratorio de los sistemas reales.

### 📜 Logging

En un sistema real necesitamos saber qué está pasando. La buena noticia: Docker captura automáticamente lo que la aplicación escribe en su **stdout** (la salida estándar: el canal por donde un programa imprime resultados normales) y su **stderr** (la salida de errores). Eso significa que no necesitamos archivos de log ni configuración especial: basta con imprimir a consola y Docker ya lo recoge.

La buena práctica es imprimir en formato **JSON** (un formato de texto estructurado, con claves y valores, que las máquinas pueden procesar sin ambigüedad):

```javascript
console.log(JSON.stringify({
  level: "INFO",
  message: "Usuario autenticado"
}));
```

`level` (INFO, ERROR, WARN — niveles de severidad) y `message` son los campos estándar que después cualquier herramienta de **observabilidad** (el tema del Level 6 del roadmap) puede leer, filtrar y graficar. Logs estructurados = logs útiles.

### 🔐 Seguridad

#### ❌ Nunca uses `latest`

```bash
docker run nginx:latest   # ❌
docker run nginx:1.24-alpine   # ✅
```

`latest` parece cómodo, pero es una trampa: es una etiqueta **flotante**. Cada vez que el equipo de nginx publica algo nuevo, la etiqueta `latest` cambia de contenido. Mañana la imagen puede ser otra, sin que nadie lo haya decidido — la receta perfecta para los deploys que "se rompen solos". La práctica correcta es **fijar una versión exacta**: `1.24-alpine` siempre es la misma imagen, en cualquier máquina, en cualquier mes. Reproducibilidad ante todo.

### 🚨 Anti-patrones comunes en producción

Un **anti-patrón** es una práctica común en la industria que, aunque se vea por todos lados, es un error conocido. Conviene aprenderlos de memoria para no repetirlos.

#### ❌ Secretos dentro de imágenes

```dockerfile
ENV DB_PASSWORD=supersecret   # MAL
```

**Nota:** `ENV` define variables de entorno (datos que el proceso puede leer con nombre, como `DB_PASSWORD`) dentro de la imagen. **Variables de entorno** son pares nombre-valor que el sistema operativo les pasa a los programas; casi todas las apps los usan para configuración.

¿Por qué está mal? Porque las imágenes son inmutables, se comparten, se suben a registries y se copian a otras máquinas. Un secreto incrustado queda escrito para siempre, a la vista de cualquiera que tenga la imagen. Los **secretos** (contraseñas, tokens, claves de API) jamás van en la imagen.

La forma correcta: pasarlos en tiempo de ejecución, cuando se lanza el contenedor:

```bash
docker run -e DB_PASSWORD=...
```

`-e` define la variable en el contenedor, no en la imagen. La imagen queda limpia; el secreto se inyecta desde afuera. (En niveles más avanzados, eso se hace con gestores de secretos y variables de entorno del entorno de despliegue, con el mismo principio.)

#### ❌ Contenedores como mascotas

"**Mascota**" es una metáfora clásica de infraestructura: un servidor con nombre, personalidad, que nadie se anima a reiniciar porque "si se apaga, se pierde todo". El mundo de los contenedores funciona al revés: los contenedores son **ganado** — intercambiables, reemplazables.

Lo que no hay que hacer:

```bash
docker exec -it app bash
```

para "arreglar producción". `docker exec` ejecuta un comando dentro de un contenedor ya corriendo (`-it` lo hace interactivo). Parece inofensivo, pero es el camino al caos: arreglamos a mano, la imagen sigue rota, y el arreglo muere con el contenedor. El flujo correcto es siempre el mismo:

1. Modificar el **Dockerfile** (la receta).
2. **Rebuild** (reconstruir la imagen).
3. **Redeploy** (tirar el contenedor viejo, levantar el nuevo).

Infraestructura reproducible: el arreglo queda documentado en la receta, no en la memoria de nadie. Si el arreglo vale la pena, vale la pena en la imagen.

#### ❌ Imágenes gigantes

Una imagen enorme significa:

- **Deploys lentos** — cada despliegue transfiere todos esos megabytes.
- **Más superficie de ataque** — más software instalado = más vulnerabilidades posibles.
- **Más consumo de red** — descargar gigas cada vez tiene costo.
- **Peor caching** — menos capas reutilizables entre builds; cada cambio obliga a transferir más.

La regla ya la vimos: imagen chica, dependencias justas, `.dockerignore` bien puesto, bases alpine.

#### ❌ Usar root

Si el contenedor corre como root y es comprometido: **comprometiste el host**. No hay "pero el contenedor aísla" — el aislamiento no es seguridad absoluta, y root dentro del contenedor es la puerta más corta hacia la máquina entera. Ya lo resolvimos en el Dockerfile con `adduser` + `USER`; acá solo queda la advertencia en mayúsculas: nunca, jamás, root en producción.

---

## 🚀 Roadmap DevOps

¿Y ahora qué sigue? Este nivel es el cimiento; miremos la escalera completa del roadmap:

1. **Docker Compose** — ya lo vimos hoy: sistemas completos en un archivo.
2. **CI/CD** — automatizar el build, el test y el deploy (Level 5, con GitHub Actions).
3. **Kubernetes** — el próximo nivel: orquesta contenedores, decide dónde viven, cuántas copias hay y qué pasa si uno muere.
4. **Terraform** — infraestructura como código: declarar servidores, redes y servicios en la nube con archivos.
5. **Cloud** — AWS: llevar todo esto a infraestructura real, pagada por uso.
6. **Observabilidad** — monitoreo y alertas: Prometheus y Grafana para ver cómo se comporta el sistema en vivo.

Cada peldaño usa al anterior. Sin entender hoy la persistencia y la reproducibilidad de Docker, Kubernetes mañana sería pura magia negra.

---

## 🧠 Principio final

Cerramos con la idea que da título al nivel: **Docker no es "correr contenedores"**. Docker es:

- **Reproducibilidad** — la misma receta, el mismo resultado, siempre.
- **Aislamiento** — cada proceso en sus paredes, con sus límites.
- **Despliegue** — una unidad portable lista para cualquier entorno.
- **Networking** — servicios que se encuentran por nombre, en redes privadas.
- **Persistencia** — datos que sobreviven a los contenedores.
- **Automatización** — de la receta al sistema completo con un comando.

Y su verdadero poder aparece cuando se conecta con lo que viene: Kubernetes, CI/CD, IaC (infraestructura como código), observabilidad y cloud. Docker es el ladrillo; los niveles siguientes construyen el edificio.

---

## 5. Consolidar en GitHub

Último paso del nivel, y es cultural: **todo código se versiona**. Lo que construimos hoy va a ser reutilizado en el Level 5, cuando hagamos CI/CD con GitHub Actions (automatización de builds y despliegues). Si no está en Git, no existe. Guardemos el nivel completo en el repositorio de proyecto.

**Nota:** recordemos la estructura: en el Level 0 creamos `~/devops-engineering-student/`, el repositorio de proyecto donde acumulamos el progreso de todos los niveles. Todo lo que sigue en esta sección se ejecuta en la terminal del Codespace, dentro de ese repositorio.

### 5.1 — Crear la estructura del nivel

```bash
cd ~/devops-engineering-student
mkdir -p level-1/app level-1/labs
```

`mkdir -p` crea directorios, y con `-p` (parents) crea también los intermedios si faltan. Estamos armando la estructura del nivel: `level-1/app` para la aplicación y `level-1/labs` para los laboratorios.

### 5.2 — Copiar los archivos de la aplicación

La aplicación Flask que usamos en los laboratorios es parte fundamental del nivel, y entra versionada:

```bash
cp <ruta-al-bootcamp>/level-1/app/app.py level-1/app/
cp <ruta-al-bootcamp>/level-1/app/requirements.txt level-1/app/
cp <ruta-al-bootcamp>/level-1/app/Dockerfile level-1/app/
cp <ruta-al-bootcamp>/level-1/app/templates/index.html level-1/app/templates/
```

**Nota:** `cp` copia archivos; `<ruta-al-bootcamp>` es un marcador — hay que reemplazarlo por la ruta real donde esté el material del bootcamp en la máquina. **Flask** es un framework web de Python (un conjunto de herramientas para construir aplicaciones web); "framework" es una base de código que ya resuelve lo genérico (rutas, respuestas HTTP) para que solo escribamos lo específico de nuestra app.

| Archivo | Propósito |
|---|---|
| `app.py` | La aplicación Flask: responde en el puerto 5000 |
| `requirements.txt` | La lista de dependencias de Python (Flask, gunicorn) — el `package.json` de Python, por así decirlo |
| `Dockerfile` | La receta para construir la imagen de la app |
| `templates/index.html` | El template HTML que renderiza la app |

**Nota:** "renderizar" significa generar la página final que ve el usuario: el template es la plantilla, la app la llena con datos y entrega el HTML completo. Y `gunicorn` es un servidor web para Python, el que sirve la app en producción.

### 5.3 — Copiar los archivos de infraestructura

Además del código de la app, guardamos la infraestructura del nivel — los archivos que describen cómo corre el sistema:

```bash
cp <ruta-al-bootcamp>/level-1/labs/03-dockerfile/Dockerfile level-1/
cp <ruta-al-bootcamp>/level-1/labs/04-docker-compose/docker-compose.yml level-1/
```

**Nota:** "infraestructura" acá son los archivos de definición del sistema (el Dockerfile y el docker-compose.yml), por oposición al código de la aplicación. Van al nivel porque sin ellos la app no tiene receta ni sistema.

### 5.4 — Committear y pushear

```bash
git add level-1/
git commit -m "feat: completar Level 1 — Docker"
git push origin main
```

Tres comandos de Git que ya conocemos del nivel anterior; repasemos la semántica:

| Comando | Explicación |
|---|---|
| `git add level-1/` | Agrega todo el directorio del nivel al **staging** — la zona de preparación, los cambios listos para el commit (como las compras en el carrito) |
| `git commit -m "..."` | Crea un **commit** — un punto de guardado en la historia del repositorio — con mensaje siguiendo *Conventional Commits*: `feat:` indica que agregamos funcionalidad, y el texto describe qué se completó |
| `git push origin main` | Sube los commits locales al repositorio remoto en GitHub. `origin` es el nombre estándar del remoto; `main` es la rama principal (la línea principal de desarrollo) |

**Salida esperada** — si todo salió bien, la terminal muestra el progreso del envío:

```text
Enumerating objects: ...
Writing objects: ...
remote: Resolving deltas: ...
 * [new branch]      main -> main
```

Con eso, el nivel queda respaldado en GitHub: código, recetas e infraestructura, listos para el Level 5.

---

## ✅ Fin del Mini-Curso Docker

Resumen final, el inventario de lo que entendemos ahora:

- **cómo viven los contenedores** — procesos de Linux aislados con namespaces, cgroups y filesystem por capas; efímeros e inmutables.
- **cómo se comunican** — por nombre, gracias al DNS interno, sobre redes bridge personalizadas.
- **cómo persisten datos** — los datos viven afuera: named volumes en producción, bind mounts en desarrollo, tmpfs para lo temporal.
- **cómo se construyen imágenes** — con un Dockerfile como receta reproducible, siguiendo buenas prácticas: sin root, chicas, con `.dockerignore` y versiones fijas.
- **cómo se operan en producción** — logs estructurados por stdout, nunca `latest`, secretos fuera de las imágenes, contenedores como ganado, nunca como mascotas, nunca root.

Docker, bien entendido, no es "correr contenedores": es un **sistema de producción en miniatura**, y ahora sabemos cómo funciona cada pieza. En el próximo nivel aplicaremos todo esto sobre la orquestación con **Kubernetes** — que no reemplaza ninguno de estos conceptos: los orquesta.

Si se llevan una sola frase del nivel, que sea esta: **Docker no es magia. Es reproducibilidad, aislamiento, despliegue, networking, persistencia y automatización — y el superpoder de todo eso aparece cuando se conecta con CI/CD, Kubernetes, IaC, observabilidad y cloud.**

Fin del guion.
