# Laboratorio 3 — Instalación del Entorno Oracle: preguntas de comprobación

**Alumna:** Andrea Guillén Fernández-Santamaría
**Repositorio:** https://github.com/AndysCorner12/oracle-database-lab
**Asignatura:** Administración de Bases de Datos — Prof. Dr. Richard Avilés López

> **Modalidad:** igual que en el Lab 2, laboratorio realizado de forma individual (compagino estudios y trabajo y la entrega es fuera de plazo). La revisión del Pull Request la hago con una segunda cuenta: **aguillen31ucam** como Autora y **AndysCorner12** (propietaria del repositorio) como Revisora.

## Notas sobre mi entorno

Diferencias respecto al enunciado, todas justificadas en los scripts o en los commits:

- **Ubuntu 26.04.1 LTS** en WSL 2 en lugar de 24.04: es la versión que instaló `wsl --install` en mi equipo. Todo el laboratorio funcionó igual (evidencia 01).
- **Puertos 8080 y 8081 ocupados** por otros contenedores (dos WordPress con los que trabajo). Por eso en el ejercicio G7 publiqué nginx en **8082** (`-p 8082:80`) y ORDS escucha en **8090** en lugar de 8080. El cambio está comentado en `00-config.sh` (constante `ORDS_PORT`) y explicado en el commit de la Parte M.
- **Ejercicios G5 y G6:** en la primera grabación de la práctica de Docker me salté esos dos ejercicios. Los repetí añadiendo a la misma grabación (`script -q -a`) y corregí el commit, que aún no estaba publicado, con `git commit --amend`.
- **Usuario ALUMNO:** la imagen oficial `database/free` ignora `APP_USER`/`APP_USER_PASSWORD`. Por eso el bloque 4 de la evidencia 07 devolvió `no rows selected`. El usuario se creó en la Parte M con `12-ords-habilitar-alumno.sql`, que lo crea si no existe.
- **shellcheck** detectó dos avisos de tipo *warning*: SC2155 en `00-config.sh` y SC2034 en `12-arrancar-ords.sh`. Los corregí en un commit propio (`fix(env): ...`). Tras la corrección, `shellcheck -x` no muestra ningún aviso.
- **Versiones instaladas:**
  - Docker 29.9.3 (Docker Desktop 4.99.0).
  - Oracle AI Database 26ai Free 23.26.3.0.0, imagen `sha256:f988b0c0…8fba`.
  - OpenJDK 21.0.13 y SQLcl 26.3.1.
  - SQL Developer 26.2.0.186.2220 y ORDS 26.3.1.

---

## Docker

### 1. ¿Qué diferencia hay entre una imagen y un contenedor? Usa como ejemplo lo que hiciste en los ejercicios G2 y G4.

Una **imagen** es una plantilla de solo lectura: el sistema de archivos y la configuración necesarios para ejecutar algo. Un **contenedor** es una instancia en ejecución, o ya terminada, creada a partir de una imagen, con su propia capa escribible encima.

- En **G2** descargué la imagen `alpine:3.20` con `docker pull`. Aparece en `docker images` (12,2 MB), pero no hay nada ejecutándose.
- En **G4** creé a partir de ella el contenedor `prueba` con `docker run -it --name prueba alpine:3.20 sh`. Tenía su propio hostname (`ed0406abe13c`) y, al salir, quedó en `docker ps -a` como `Exited (0)`.

De una misma imagen se pueden crear muchos contenedores: después creé `prueba2` y otros dos temporales con `--rm`, todos desde la misma `alpine:3.20`.

### 2. En el Ejercicio G5 el archivo nota.txt desapareció y en el G6 no. Explica por qué.

- **En G5**, escribí `/tmp/nota.txt` dentro del contenedor `prueba`. Ese archivo vivía en la capa escribible de *ese* contenedor. Al hacer `docker rm prueba`, la capa se borró con él. El contenedor nuevo `prueba2` partía otra vez de la imagen limpia, así que obtuve `cat: can't open '/tmp/nota.txt': No such file or directory`.
- **En G6**, escribí el archivo en `/datos`, que era un **volumen con nombre** (`datos-prueba`) montado con `-v datos-prueba:/datos`. El volumen vive fuera del contenedor, gestionado por Docker. El primer contenedor se borró solo al salir (`--rm`), pero un segundo contenedor que montó el mismo volumen leyó `dato importante`.

Es exactamente lo que hacemos con Oracle: los datafiles van en el volumen `oralab-26ai-data`, no dentro del contenedor.

### 3. ¿Qué diferencia hay entre docker ps y docker ps -a, y qué significa STATUS = Exited (0)?

- `docker ps` muestra solo los contenedores **en ejecución**.
- `docker ps -a` muestra **todos**, también los detenidos.

En mi caso, `docker ps` solo listaba los contenedores de WordPress/MariaDB. `docker ps -a` mostraba además el `hello-world` y el `prueba` ya terminados.

`Exited (0)` significa que el proceso principal del contenedor terminó y que su **código de salida fue 0**, es decir, que acabó correctamente y sin error. Un código distinto de 0 indicaría un fallo; por ejemplo, mis contenedores `wpcli` aparecían como `Exited (255)`.

### 4. En -p 8181:8181, ¿qué número corresponde a tu equipo y cuál al contenedor? ¿Qué pasaría con -p 80:8080 en el ejercicio de nginx?

El formato es `-p <puerto del equipo>:<puerto del contenedor>`. En `-p 8181:8181` el **primero** es el puerto de mi equipo (host) y el **segundo**, el del contenedor. Aquí coinciden.

Con `-p 80:8080`, Docker escucharía en el puerto 80 de mi equipo y reenviaría al **8080 del contenedor**. Pero nginx escucha dentro del contenedor en el **80**, no en el 8080, así que la petición llegaría a un puerto donde no hay nadie y el navegador no obtendría respuesta (conexión cerrada o vacía).

Lo correcto es `-p <puerto libre>:80`. En mi caso usé `-p 8082:80`, porque 8080 y 8081 ya estaban ocupados por otros contenedores. `docker port web` lo confirmó: `80/tcp -> 0.0.0.0:8082`.

### 5. ¿Por qué un contenedor de Oracle se queda en marcha y el de hello-world termina solo?

Un contenedor vive mientras vive su **proceso principal**:

- El de `hello-world` es un programa que imprime un mensaje y termina. Al acabar el proceso, el contenedor pasa a `Exited (0)`.
- El de Oracle arranca el listener y la instancia y se queda en primer plano mostrando el `alert.log` (en mis logs: `The following output is now a tail of the alert.log`). Es un proceso de larga duración que no termina, así que el contenedor sigue `Up`.

### 6. ¿Qué es el digest de una imagen y por qué lo registramos si ya sabemos que usamos :latest?

El **digest** es el hash SHA-256 del contenido de la imagen. Identifica de forma única e inmutable exactamente esos bytes. El mío es `sha256:f988b0c04c4c386cd306a2a914c0d7a9702d83acc31b064a28ad8eb6278a8fba` (evidencia 04).

`:latest` es solo una **etiqueta móvil**: Oracle la mueve a una versión nueva cada vez que publica una actualización. Saber que usé `latest` no dice qué versión tenía ese día. Con el digest se puede saber exactamente qué imagen se instaló, reproducir el entorno (`docker pull imagen@sha256:...`) y explicar diferencias si dentro de unos meses otra persona obtiene una versión distinta.

### 7. ¿Qué comando borraría realmente los datos de Oracle? ¿Por qué docker rm oralab-26ai no lo hace?

`docker rm oralab-26ai` solo elimina el **contenedor**, es decir, el proceso y su capa escribible. Los datafiles están en el volumen con nombre `oralab-26ai-data`, montado en `/opt/oracle/oradata`, y los volúmenes tienen un ciclo de vida independiente. Si vuelvo a crear el contenedor con el mismo `-v`, la base de datos reaparece intacta.

Lo que borra realmente los datos es eliminar el **volumen**: `docker volume rm oralab-26ai-data`, después de borrar el contenedor. También `docker rm -v`, o `docker volume prune` si el volumen quedara huérfano. Es lo que comprobé a pequeña escala en G6 con `docker volume rm datos-prueba`.

---

## Git, organización y evidencia

### 8. ¿Por qué este laboratorio se hace dentro del repositorio oracle-database-lab, con Issue, branch y Pull Request, en vez de en una carpeta aparte?

Porque la instalación del entorno es un **cambio de infraestructura** del proyecto y debe tratarse como cualquier otro cambio: trazable, revisado y reproducible.

- El **Issue #5** define qué hay que hacer y con qué criterios.
- La **branch** `chore/5-install-oracle-environment` aísla el trabajo sin tocar `main`, que además está protegida.
- Cada **commit** registra una Parte con su script y su evidencia.
- El **Pull Request** permite que otra persona lo revise antes de integrarlo.

Así cualquiera del equipo puede reconstruir el mismo entorno ejecutando los scripts versionados. En una carpeta aparte, la instalación sería algo que "hice en mi portátil" sin historial ni revisión, y es lo que se pierde en cuanto falla el equipo.

### 9. ¿Qué diferencia hay entre source 00-config.sh y bash 00-config.sh? ¿Por qué usamos source?

- `bash 00-config.sh` lanza **otro proceso bash**, un subshell. Las variables y la función `ts` se definen en ese proceso y desaparecen al terminar, así que en mi terminal no quedaría nada.
- `source 00-config.sh` ejecuta el archivo **en la shell actual**, así que `$CONT_NAME`, `$EVID`, `$ORDS_PORT` o `ts` quedan disponibles para los comandos siguientes.

Por eso cada script lo carga con `source` al principio y yo lo volvía a cargar en cada terminal nueva. Si no se hace, los nombres de evidencia salen sin marca (`/script/_01-...`).

### 10. Explica cada parte del nombre 20260915T091230Z_02-docker.script.log.

- **`20260915`**: fecha en formato ISO 8601 compacto (AAAAMMDD), 15 de septiembre de 2026.
- **`T`**: separador entre fecha y hora del estándar ISO 8601.
- **`091230`**: hora HHMMSS, 09:12:30.
- **`Z`**: la hora está en **UTC** (*Zulu*), no en hora local. Así no hay ambigüedad con husos horarios ni cambios de hora. Lo genera `ts()` con `date -u`.
- **`_`**: separa la marca de tiempo de la descripción.
- **`02`**: número de evidencia según el mapa del laboratorio, que permite ordenarlas y comprobar que no falta ninguna.
- **`-docker`**: descripción breve en kebab-case.
- **`.script.log`**: tipo de evidencia, aquí la salida de un script de terminal. Las otras son `.spool.log` (sesión SQL) y `.png` (captura).

Mi equivalente real es `20261110T100720Z_02-docker.script.log`.

### 11. ¿Para qué sirve .gitattributes y qué error evita?

Le dice a Git cómo tratar ciertos archivos. En nuestro caso, `*.sh`, `*.sql` y `*.md` con `text eol=lf` obligan a guardarlos con finales de línea **LF** (Linux) aunque se editen en Windows, que usa CRLF.

Evita el error `$'\r': command not found` (o `bad interpreter`) al ejecutar un script `.sh` con finales de línea de Windows en Linux. Bash interpreta el `\r` como parte del comando. Por eso fue el primer commit de la branch: había que normalizar antes de añadir ningún script.

### 12. ¿Por qué en este Pull Request elegimos Create a merge commit en lugar de Squash and merge?

Porque aquí **cada commit tiene valor propio**: corresponde a una Parte del laboratorio, con su script y su evidencia, y documenta cuándo y cómo se verificó cada herramienta. *Squash and merge* juntaría los 16 commits de la branch en uno solo y se perdería ese historial paso a paso; ya no se podría ver, por ejemplo, el commit que instaló ORDS o el que corrigió los avisos de shellcheck.

*Create a merge commit* conserva todos los commits en `main` y añade un commit de merge que marca el punto de integración. En el Lab 2 era al revés: los commits intermedios ("arreglo de la revisión") no aportaban nada por separado y *squash* dejaba un historial más limpio.

---

## Seguridad

### 13. Describe las cuatro capas de la estrategia de contraseñas (Parte D) y qué pasaría si te saltas la primera.

1. **Decirle a Git qué ignorar:** añadir `config/.env` (y `backups/`) a `.gitignore` *antes* de crear el archivo con contraseñas.
2. **Plantilla versionada sin secretos:** `config/.env.example`, con los nombres de las variables (`ORACLE_PWD`, `APP_USER_PWD` y, desde la Parte M, `ORDS_PUBLIC_PWD`) y valores de ejemplo como `change_me`.
3. **Archivo real local:** copiar la plantilla a `config/.env`, poner las contraseñas reales y verificar con `git check-ignore -v config/.env` que Git lo ignora. A mí me respondió `.gitignore:1:config/.env`.
4. **Leer sin escribir nunca a mano:** cargar con `set -a; source config/.env; set +a` y usar siempre `"$ORACLE_PWD"`, de modo que la contraseña no se teclea ni queda en el historial.

Si te saltas la **primera**, en cuanto creas `config/.env` Git lo ve como archivo nuevo. Un `git add .` o un `git add config/` lo metería en un commit, y tras el push la contraseña quedaría publicada en GitHub y en el historial, aunque se borre después.

### 14. ¿Por qué no escribimos la contraseña directamente en el comando docker run, aunque el script no se suba a Git?

Porque Git no es el único sitio donde queda. Todo lo que se teclea en la terminal se guarda en texto plano en `~/.bash_history`. Si se escribe `-e ORACLE_PWD=MiClave123`, la contraseña queda en ese archivo indefinidamente, y además puede aparecer en capturas de pantalla, en la evidencia grabada con `tee`/`script` o en el portapapeles.

Con `-e ORACLE_PWD="$ORACLE_PWD"` en el historial y en el script solo queda el **nombre** de la variable. Por eso, antes de cada commit con evidencias, comprobé con `grep -rlF` que ninguna de mis tres contraseñas aparecía en `docs/` ni en `scripts/`.

### 15. Si descubres tu contraseña en un commit ya publicado, ¿basta con borrarla en un commit nuevo? ¿Qué debes hacer?

No basta. Un commit nuevo solo cambia la versión actual del archivo, pero el commit anterior sigue en el historial y cualquiera puede verlo con `git log -p` o en GitHub. Además, si alguien ya clonó el repositorio, tiene su copia.

Lo correcto es:
1. Dar el secreto por **comprometido** y **rotarlo** de inmediato: cambiar la contraseña en `config/.env` y recrear el contenedor con la nueva.
2. Avisar al docente o al responsable para limpiar el historial de la branch (reescritura con herramientas como `git filter-repo` y push forzado coordinado).

En una empresa se trata como un incidente de seguridad. En mi caso, el control de la Parte O (`git grep` y `git log -p | grep -c`) dio 0 apariciones antes del push.

---

## Oracle y herramientas

### 16. ¿Por qué no usamos SPOOL ni @archivo.sql con sqlplus dentro del contenedor, y qué hicimos en su lugar?

Porque `sqlplus` se ejecuta **dentro del contenedor**:

- Un `SPOOL` escribiría el archivo en el sistema de archivos del contenedor, no en mi repositorio.
- `@archivo.sql` buscaría el script dentro del contenedor, donde no existe (error SP2-0310).

En su lugar, el SQL se envía desde mi equipo por la **entrada estándar** con `docker exec -i ... sqlplus -s ... < archivo.sql`, y la salida se guarda en el repositorio con `| tee "$EVID/spool/..."`. Así se generaron las evidencias 07 y 08. En la Parte L, con SQLcl instalado en Ubuntu, sí usé `SPOOL` con normalidad (evidencia 10).

### 17. ¿Qué hace WHENEVER SQLERROR EXIT SQL.SQLCODE al inicio de V000 y V001, y qué pasaría sin esa línea?

Hace que, si cualquier sentencia SQL falla, SQL*Plus **termine inmediatamente** y devuelva el código del error de Oracle como código de salida del proceso. Gracias a eso, `08-aplicar-migraciones.sh` detecta el fallo (`if ! docker exec ... ; then`), muestra `ERROR en ...` y no intenta aplicar la siguiente migración.

Sin esa línea, SQL*Plus mostraría el `ORA-xxxxx`, seguiría ejecutando el resto del script y terminaría con código 0, como si todo hubiera ido bien. Se podría aplicar V001 sobre un V000 a medias (por ejemplo, crear tablas para un usuario que no existe) sin que nada avisara.

### 18. ¿Qué es una migración y por qué V000 y V001 no se deben editar una vez aplicadas?

Una **migración** es un script versionado que lleva la base de datos de un estado al siguiente. Se aplican en orden según su prefijo: V000 crea tablespaces y usuarios, y V001 crea las tablas de los 5 esquemas. Herramientas como Flyway o Liquibase (integrada en SQLcl) registran qué migraciones se aplicaron.

Una vez aplicadas no se editan porque **ya forman parte del historial de la base de datos**. Si cambio V001 después de aplicarla, mi base ya no se corresponde con el archivo, y otra persona que la ejecute desde cero obtendría una base distinta de la mía. Además, las herramientas de migración detectan el cambio de checksum y fallan. Si hay que corregir algo, se crea una migración nueva (V002...) que haga el cambio sobre el estado actual.

### 19. ¿Por qué en SQL Developer se usa el servicio FREEPDB1 y no FREE ni un SID?

Oracle 26ai usa arquitectura **multitenant**:

- **FREE** es el contenedor raíz (CDB$ROOT), que contiene el diccionario común y no es el lugar para trabajar.
- **FREEPDB1** es la base de datos conectable (PDB) donde están nuestros usuarios `ADMIN_*` y `ALUMNO` y sus tablas.

Si me conecto a FREE, o usando el SID (que identifica la instancia, no una PDB), llego a la raíz y no veo los esquemas del curso. Por eso en SQL Developer elegí *Nombre del Servicio* = `FREEPDB1` (captura 11), y la consulta en Database Actions devolvió `ALUMNO | FREEPDB1` (captura 12).

### 20. ¿Qué aporta SQLcl frente a SQL*Plus, y por qué un DBA debe dominar ambas?

**SQLcl** es la herramienta moderna del día a día. Aporta:
- autocompletado e historial;
- formato automático de resultados (`SET SQLFORMAT ansiconsole`, que usé en la evidencia 10);
- conexiones guardadas sin contraseña (`CONNECT -save oralab26-system`, que comprobé con `CONNMGR list`);
- integración con Liquibase para migraciones.

Además, se instala en mi equipo, así que `SPOOL` escribe directamente en el repositorio.

**SQL*Plus** está en cualquier servidor Oracle del mundo desde hace décadas. Cuando solo hay una terminal en un servidor de producción, como en nuestro caso dentro del contenedor, es lo que hay.

Un DBA debe dominar ambas: SQLcl para trabajar con productividad y SQL*Plus para poder operar en cualquier servidor sin instalar nada.

---

## Entorno de trabajo

### 21. ¿Por qué el curso pasa de Git Bash a Ubuntu en WSL 2? Da al menos dos problemas concretos de Git Bash que desaparecen en Ubuntu.

Porque Oracle, Docker y prácticamente toda la infraestructura de empresa corren sobre Linux. En Ubuntu se usan los mismos comandos, rutas (`/opt`, `/etc`, `/var/log`), gestor de paquetes (`apt`) y permisos que en un servidor real. Git Bash es solo una emulación de bash sobre Windows.

Problemas concretos de Git Bash que desaparecen:
- **Conversión de rutas:** Git Bash convierte rutas como `/opt/...` en rutas de Windows y rompe argumentos de `docker`.
- **Terminales interactivas:** `docker run -it` falla con "not a TTY" y necesita trucos como `winpty`. En Ubuntu, `docker run -it ... alpine sh` (ejercicio G4) funcionó directamente.
- **Utilidades de administración que faltan:** no trae `free`, `ss` ni `htop`. Yo usé `free` en el script 01 y `ss -tlnp` para comprobar que ORDS escuchaba en el 8090.
- **Pegado:** en Git Bash tuve problemas al pegar comandos (aparecía `^[[200~`, el pegado entre corchetes).
- **Herramientas Java:** tienen problemas para pedir contraseñas de forma oculta en Git Bash. En Ubuntu, SQLcl pidió la contraseña sin mostrarla.

### 22. ¿Por qué clonamos el repositorio en ~/oracle-database-lab y no trabajamos sobre la carpeta de Windows (/mnt/c/...)? ¿Y por qué recomendamos bash frente a zsh para los scripts del curso?

`/mnt/c/...` es el disco de Windows visto desde Linux. Trabajar ahí obliga a que cada operación cruce entre dos sistemas de archivos distintos, lo que tiene tres consecuencias:
- Git y Docker se vuelven varias veces más lentos.
- No se conservan los permisos de Linux, como el de ejecución. Lo vi al copiar las capturas desde `/mnt/c`: llegaron con permiso de ejecución (`100755`) y tuve que corregirlas con `chmod 644`.
- Vuelven los problemas de finales de línea.

En `~/oracle-database-lab`, dentro del sistema de archivos de Linux, todo funciona de forma nativa. La copia de Windows de los Labs 1 y 2 se queda sin usar, para no trabajar en dos copias a la vez. Al principio ejecuté algunos comandos en PowerShell sobre esa copia por error, y por eso tuve claro que había que trabajar solo en la de Ubuntu.

Sobre **bash frente a zsh**: bash es la shell por defecto en Ubuntu, Debian, Red Hat y Oracle Linux, y está garantizada en cualquier servidor recién instalado. zsh es más cómoda para el uso interactivo, pero tiene diferencias sutiles (arrays, *globbing*, expansión de variables) y normalmente no está instalada en servidores. Por eso los scripts compartidos se escriben en bash (`#!/usr/bin/env bash`) y se ejecutan con `bash script.sh`: así funcionan igual en cualquier máquina.
