Administración de Bases de Datos

**PRÁCTICA 1 — LABORATORIO 1**

**Git Fundamentals: respuestas y lecciones aprendidas**

Alumna: Andrea Guillén Fernández-Santamaría

Repositorio: https://github.com/AndysCorner12/oracle-database-lab

Profesor: Dr. Richard Avilés López

*Grado en Ingeniería Informática — UCAM*

# 1. Respuestas a las observaciones del laboratorio

## Parte G — ¿A dónde apunta HEAD? ¿En qué orden aparecen los commits?

Tras el commit en la rama feature/customer-search, git log --oneline
--graph --all mostró en mi repositorio:

> \* c558c8d (HEAD -\> feature/customer-search) docs: draft customer
> search notes
>
> \* da30a0d (main) docs: fix typo in schema notes
>
> \* 3ef863f docs: add customer schema notes
>
> \* bc070d2 chore: add folder structure with .gitkeep placeholders
>
> \* a2d53b8 docs: add initial project documentation

**HEAD apunta a la rama** feature/customer-search, y esa rama apunta a
su vez al commit más reciente (docs: draft customer search notes). La
flecha HEAD -\> feature/customer-search indica justamente eso: HEAD no
apunta a un commit directamente, sino a la rama activa.

**Los commits aparecen en orden cronológico inverso**: el más reciente
arriba y el commit inicial abajo. Se ve también que main se ha quedado
un commit por detrás: el commit del borrador solo existe en la rama
feature.

Al ejecutar git switch main y ls docs/, el archivo customer-search.md
desaparece del disco; al volver a la rama, reaparece. No se creó ninguna
carpeta feature/: Git simplemente reescribe el Working Directory para
que coincida con el commit al que apunta la rama activa.

## Parte H — Comentario sobre git status

Secuencia observada en la rama fix/readme-title:

- Primer git status: README.md aparece en rojo como **modified**
  (Changes not staged for commit). Git detecta que el archivo cambió
  respecto al último commit, pero el cambio sigue solo en el Working
  Directory.

- Tras git add README.md, el segundo git status lo muestra en verde bajo
  **Changes to be committed**: el cambio ya está en la Staging Area.

- Tras el commit, git status indica **nothing to commit, working tree
  clean**: Working Directory, Staging Area y último commit coinciden.

Nota: el guion del laboratorio indica git status -c, pero -c no es una
opción válida de git status (Git devuelve un error de opción
desconocida). Lo que se quería comprobar se ve con git status normal, o
con git status -s para el formato corto.

## Parte H — Resultado de los merges

El primer git merge fix/readme-title fue un **Fast-forward** (Updating
da30a0d..f45c4ae): main no tenía commits propios desde que se creó la
rama, así que Git solo adelantó el puntero, sin crear commit de fusión.

El segundo git merge fix/readme-subtitle produjo CONFLICT (content):
Merge conflict in README.md, el prompt pasó a (main\|MERGING) y git
status mostró README.md como both modified. Tras resolverlo, el
historial quedó así:

> \* 39da5a1 (HEAD -\> main) merge: resolve README title conflict
>
> \|\\
>
> \| \* 557ddd7 (fix/readme-subtitle) docs: rename project title
> (academic version)
>
> \* \| f45c4ae (fix/readme-title) docs: rename project title (training
> edition)
>
> \|/
>
> \| \* c558c8d (feature/customer-search) docs: draft customer search
> notes
>
> \|/
>
> \* da30a0d docs: fix typo in schema notes
>
> \* 3ef863f docs: add customer schema notes
>
> \* bc070d2 chore: add folder structure with .gitkeep placeholders
>
> \* a2d53b8 docs: add initial project documentation

## Parte I — Push y pull

Vinculé el remoto con git remote add origin
https://github.com/AndysCorner12/oracle-database-lab.git, subí main con
git push -u origin main y después las tres ramas. Edité README.md desde
la web de GitHub en la rama fix/readme-subtitle y lo traje con git pull,
que hizo un Fast-forward 557ddd7..b743257: antes del pull la línea
"Edited from GitHub web interface." no estaba en mi disco y después sí.

Para practicar el error de sincronización, edité README.md desde la web
directamente en main (commit 1e04b1c) y, sin hacer pull, intenté git
push desde mi ordenador. GitHub lo rechazó con ! \[rejected\] main -\>
main (fetch first): el remoto tenía un commit que mi main local no
tenía, y Git no permite subir algo que borraría ese trabajo. Lo resolví
con git pull, que trajo el commit (fue un Fast-forward 39da5a1..1e04b1c
porque yo no tenía commits locales nuevos; si los hubiera tenido, habría
creado un commit de fusión). Después git push respondió Everything
up-to-date.

# 2. Preguntas de comprobación

## 1. Working Directory, Staging Area y Local Repository

El **Working Directory** es la carpeta tal como la veo y la edito en el
disco. La **Staging Area** es una zona intermedia donde preparo
exactamente qué cambios entrarán en el siguiente commit. El **Local
Repository** es el historial de commits guardado dentro de .git.

Ejemplo con docs/customer-schema.md de la Parte E:

- Lo creo con echo ... \> docs/customer-schema.md → existe solo en el
  Working Directory (untracked, en rojo).

- git add docs/customer-schema.md → pasa a la Staging Area (en verde).

- git commit -m "docs: add customer schema notes" → queda grabado en el
  Local Repository con su hash, y ya aparece en git log.

Una analogía que me sirve: el Working Directory es el escritorio, la
Staging Area es la caja de envío donde meto solo lo que quiero mandar, y
el commit es cerrar y etiquetar la caja.

## 2. Si modifico un archivo pero no hago git add, ¿entra en el próximo commit?

**No.** git commit solo graba lo que está en la Staging Area. Si
modifico un archivo y no hago git add, el cambio se queda en el Working
Directory y el commit no lo incluye. Es exactamente lo que provocamos en
el Ejercicio 1 de la Parte E: el commit no llegó a realizarse porque el
archivo nunca pasó por la Staging Area.

Esto es una ventaja, no un defecto: me permite tener varios cambios a
medias y confirmar solo los que forman un cambio lógico completo. (La
excepción es git commit -a, que añade automáticamente los archivos ya
seguidos que estén modificados, pero nunca los archivos nuevos.)

## 3. ¿Por qué git status no mostraba las carpetas vacías?

Porque **Git versiona archivos, no carpetas**. Internamente guarda
contenidos de archivos y las rutas en las que están; una carpeta vacía
no tiene ningún contenido que registrar, así que para Git simplemente no
existe.

El truco fue crear un archivo de relleno .gitkeep dentro de cada carpeta
(runbooks/.gitkeep, database/schema/.gitkeep, etc.). .gitkeep no es un
nombre especial para Git: es una convención. Al haber un archivo dentro,
la carpeta pasa a formar parte del repositorio. Una alternativa más útil
a largo plazo es poner un README.md que explique para qué sirve la
carpeta.

## 4. ¿Qué es HEAD?

HEAD es el puntero que indica **dónde estoy ahora mismo** en el
historial. Normalmente apunta a una rama (por ejemplo HEAD -\> main), y
la rama apunta al último commit. Cuando hago un commit nuevo, la rama
avanza y HEAD avanza con ella porque va "enganchado" a la rama.

Es como el marcador de "usted está aquí" de un mapa: al hacer git
switch, muevo el marcador a otra rama y Git actualiza los archivos del
disco para que reflejen ese punto del historial. Si HEAD apunta
directamente a un commit en vez de a una rama, se dice que está en
estado "detached HEAD".

## 5. Diferencia entre git switch -c y mkdir

mkdir crea una **carpeta física** en el disco: ocupa espacio y es
visible con ls. git switch -c crea una **rama**, que es solo un puntero
(un pequeño archivo dentro de .git/refs/heads/ con el hash de un commit)
y además mueve HEAD a ella. No se copia ni duplica ningún archivo.

Lo comprobamos en la Parte G de dos formas:

- Tras git switch -c feature/customer-search, ls -la mostró exactamente
  las mismas carpetas que antes: no apareció ninguna carpeta feature/.

- Al alternar entre main y la rama feature, customer-search.md
  desaparecía y reaparecía en docs/. La carpeta es la misma; lo que
  cambia es el contenido que Git coloca en ella según el commit activo.

## 6. Marcadores del conflicto de la Parte H

> \<\<\<\<\<\<\< HEAD
>
> \# Oracle Database Lab (Training Edition)
>
> =======
>
> \# Oracle Database Lab — Academic Version
>
> \>\>\>\>\>\>\> fix/readme-subtitle

- Entre \<\<\<\<\<\<\< HEAD y ======= está **mi versión actual**: lo que
  tenía main en ese momento, que ya incluía el título "Training Edition"
  porque antes había fusionado fix/readme-title.

- Entre ======= y \>\>\>\>\>\>\> fix/readme-subtitle está **la versión
  entrante**: la de la rama que estaba fusionando, con "Academic
  Version".

Git no puede decidir solo porque ambas ramas modificaron la misma línea
partiendo del mismo origen. Lo resolví combinando las dos ideas en \#
Oracle Database Lab (Training Edition — Academic Version), borrando los
tres marcadores, revisando el archivo completo y cerrando con git add
README.md y git commit -m "merge: resolve README title conflict".

## 7. ¿Por qué no hacer --amend sobre un commit ya subido?

--amend no edita el commit: lo **sustituye por uno nuevo con distinto
hash**. Si el commit original ya está en GitHub, mi historial local y el
remoto dejan de coincidir. Al hacer push, GitHub lo rechazará
(non-fast-forward) y la única forma de forzarlo sería git push --force,
que sobrescribe el remoto. Cualquier compañero que ya hubiera descargado
el commit original tendría un historial incompatible con el nuevo.

Regla práctica: mientras el commit sea solo local, --amend es seguro. En
mi repositorio lo comprobé en la Parte E: el commit b1cb136 dcos: fxi
typo desapareció y en su lugar quedó da30a0d docs: fix typo in schema
notes, con otro hash. Una vez publicado, el error se corrige con un
commit nuevo.

## 8. Si borro la carpeta .git, ¿qué se pierde?

Se pierde **todo lo que es Git**: el historial completo de commits,
todas las ramas (incluidas las que no están en el disco en ese momento,
como fix/readme-title), la Staging Area, la configuración local y el
vínculo con el remoto origin.

**No se pierden los archivos del Working Directory**: el disco queda tal
y como estaba en ese instante, en la rama que tuviera activa. Pero sería
una carpeta normal sin historial. Además, los archivos que solo existían
en otras ramas (por ejemplo customer-search.md si estuviera en main) se
perderían.

Si el repositorio ya estaba subido a GitHub, podría recuperar el
historial con git clone, salvo los commits que no hubiera hecho push.

## 9. Diferencia entre Git y GitHub (sin usar "nube")

**Git** es un programa que se instala en mi ordenador y lleva el control
de versiones: guarda instantáneas del proyecto, permite crear ramas,
fusionarlas y volver atrás. Funciona sin conexión; todo lo de las partes
A a H lo hice sin internet.

**GitHub** es un servicio web, de una empresa, que aloja repositorios
Git en sus servidores para que otras personas puedan acceder a ellos, y
añade herramientas de colaboración: Pull Requests, revisión de código,
Issues, automatizaciones y protección de ramas.

Una comparación: Git es como un procesador de texto con historial de
versiones; GitHub es como un sitio de internet donde publicas ese
documento para trabajar sobre él con otras personas. Existen
alternativas a GitHub (GitLab, Bitbucket) que usan el mismo Git por
debajo.

## 10. ¿Por qué no subir un .env con contraseñas aunque el repositorio sea privado?

- **El historial es permanente.** Aunque luego borre el archivo, la
  contraseña sigue en los commits anteriores y cualquiera con acceso
  puede recuperarla con git log o git show.

- **"Privado" no es para siempre.** Un repositorio privado puede hacerse
  público por error, se puede añadir a colaboradores nuevos, alguien
  puede clonarlo en un portátil que luego se pierde, o una cuenta con
  acceso puede ser comprometida.

- **Las credenciales no deben estar en el código.** Cada entorno
  (desarrollo, pruebas, producción) tiene las suyas, y los repositorios
  se copian y comparten mucho más que una base de datos de producción.

Lo correcto es añadir .env al .gitignore antes del primer commit y subir
un .env.example con los nombres de las variables pero sin valores
reales. Si se sube una clave por error, hay que **cambiarla
inmediatamente**, porque limpiar el historial no garantiza que nadie la
haya copiado.

## 11. Push rechazado con "non-fast-forward"

Probablemente **el remoto tiene commits que mi compañero no tiene en
local**: otra persona subió cambios a esa rama, o él mismo editó algo
desde la web de GitHub (como hicimos en la Parte I, paso 4). Git rechaza
el push para no borrar esos commits.

Me pasó a mí en la Parte I al editar main desde la web y luego intentar
subir un commit local. Lo primero que ejecutaría:

> git pull
>
> \# resolver conflictos si aparecen, luego:
>
> git push

Antes de nada, revisaría con git status y git log --oneline --graph
--all (tras un git fetch) para ver cuántos commits hay de diferencia. Lo
que **no** haría es git push --force, porque eliminaría los commits del
remoto.

## 12. Tipo de Conventional Commit para cada caso

| **Cambio**                                  | **Tipo** | **Ejemplo de mensaje**                           |
|---------------------------------------------|----------|--------------------------------------------------|
| Añadir un índice de rendimiento a una tabla | perf     | perf(db): add index on customer.email            |
| Corregir una restricción mal definida       | fix      | fix(db): correct customer foreign key constraint |
| Actualizar el README                        | docs     | docs: update README with setup instructions      |

El índice es perf y no feat porque no añade funcionalidad nueva para el
usuario: las consultas devuelven lo mismo, solo que más rápido.

# 3. Lecciones aprendidas

## Sobre el funcionamiento de Git

- Un cambio recorre cuatro zonas (Working Directory → Staging Area →
  Local Repository → Remoto) y cada comando mueve el cambio de una a
  otra. Entender esto explica casi todos los "misterios" de git status.

- Una rama no es una carpeta, es un puntero ligero. Por eso crear ramas
  es prácticamente gratis y conviene crear una para cada cambio.

- Git versiona archivos, no carpetas: la estructura de directorios vacía
  necesita .gitkeep o un README.

- Un conflicto no es un error: es Git pidiendo una decisión que solo una
  persona puede tomar. Con git merge --abort siempre se puede volver
  atrás.

## Sobre buenas prácticas

- Ejecutar git status y git diff --staged antes de cada commit evita la
  mayoría de errores (como el commit vacío de la Parte E).

- Los mensajes con Conventional Commits hacen que git log --oneline se
  lea como un resumen del proyecto.

- Un commit, un cambio lógico: facilita revisar, encontrar errores y
  deshacer.

- --amend solo en local. Lo publicado no se reescribe.

- Configurar user.name y user.email antes del primer commit, porque
  quedan grabados para siempre.

- Crear .gitignore desde el principio y no subir nunca secretos.

## Problemas que encontré y cómo los resolví

- Abrí primero el Símbolo del sistema (cmd) en lugar de Git Bash.
  Comandos como touch, ls -la o mkdir -p no existen en cmd; el prompt de
  Git Bash (MINGW64) es la forma de saber que estás en el terminal
  correcto.

- Al pegar varias líneas a la vez en Git Bash se coló un carácter de
  control (^\[\[200~) y el primer comando falló con command not found.
  Lo solucioné pegando los comandos de uno en uno.

- Escribí cd /Documents en lugar de cd ~/Documents. La barra inicial
  busca en la raíz del disco; ~ es mi carpeta de usuario. Creé el
  proyecto directamente en ~/oracle-database-lab.

- git init creó la rama como master. Antes del primer commit la renombré
  con git branch -m main y configuré git config --global
  init.defaultBranch main para los repositorios futuros.

- Ya tenía configurada globalmente la identidad de mi trabajo. Para no
  mezclarla, configuré nombre y correo de la universidad **solo en este
  repositorio** (git config user.email ... sin --global). git show HEAD
  confirmó que los commits salen con mi correo de la UCAM.

- Git mostraba el aviso LF will be replaced by CRLF. Es normal en
  Windows: Git convierte los finales de línea y no afecta al contenido.

- El push fue rechazado (fetch first) tras editar main desde la web; lo
  resolví con git pull antes de volver a subir, sin usar nunca --force.

# 4. Comandos clave del laboratorio

| **Comando**                     | **Para qué sirve**                                   |
|---------------------------------|------------------------------------------------------|
| git init                        | Crea el repositorio (la carpeta oculta .git)         |
| git status                      | Muestra en qué zona está cada cambio                 |
| git add \<archivo\>             | Pasa un cambio a la Staging Area                     |
| git diff --staged               | Muestra exactamente lo que se va a confirmar         |
| git commit -m "tipo: desc"      | Graba lo que hay en la Staging Area                  |
| git commit --amend              | Rehace el último commit (solo si no se ha subido)    |
| git log --oneline --graph --all | Historial resumido con ramas                         |
| git switch -c \<rama\>          | Crea una rama y se cambia a ella                     |
| git merge \<rama\>              | Fusiona otra rama en la actual                       |
| git merge --abort               | Cancela un merge con conflictos                      |
| git remote add origin \<url\>   | Vincula el repositorio con GitHub                    |
| git push -u origin main         | Sube la rama y la deja vinculada al remoto           |
| git pull                        | Trae y fusiona los cambios remotos de la rama actual |
