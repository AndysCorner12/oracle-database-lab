# Laboratorio 2 — GitHub Team Workflow: preguntas de comprobación

**Alumna:** Andrea Guillén Fernández-Santamaría
**Repositorio:** https://github.com/AndysCorner12/oracle-database-lab
**Asignatura:** Administración de Bases de Datos — Prof. Dr. Richard Avilés López

> **Modalidad:** laboratorio realizado de forma individual (compagino estudios y trabajo, no pude formar pareja y la entrega es fuera de plazo). Siguiendo la sección 2 del enunciado, usé una segunda cuenta de GitHub como revisora: **aguillen31ucam** como Autora y **AndysCorner12** (propietaria del repositorio) como Revisora.

---

### 1. ¿Por qué un Issue sin criterios de aceptación es un problema, aunque la descripción parezca clara?

Porque no hay forma objetiva de saber cuándo está terminado. Una descripción como "hace falta una guía de contribución" deja abierto qué debe contener, dónde va o si hay que enlazarla. Los criterios de aceptación convierten la intención en una lista comprobable: en mi Issue #1, el Revisor pudo verificar uno a uno que `CONTRIBUTING.md` estaba en la raíz, que explicaba la convención de ramas y Conventional Commits y que estaba enlazado desde el README. Sin esa lista, la revisión depende de la interpretación de cada persona.

### 2. Diferencia entre "Refs #N" y "Closes #N"

`Refs #N` solo crea una referencia cruzada: el commit o PR aparece enlazado en el Issue, pero el Issue sigue abierto. `Closes #N` (también `Fixes` o `Resolves`) cierra el Issue automáticamente cuando el cambio llega a `main`. En mi caso usé `Refs #1` en los dos commits de la rama, porque el trabajo aún no estaba integrado, y `Closes #1` en la descripción del PR #2; el Issue #1 se cerró solo en el momento del merge.

### 3. ¿Qué ocurre si haces git push directamente sobre una main protegida? ¿Es un error tuyo o del sistema?

GitHub rechaza el push. Lo comprobé en la Parte H: obtuve `GH006: Protected branch update failed for refs/heads/main` y `Changes must be made through a pull request`. No es un fallo del sistema: es la protección funcionando como se configuró. El "error" es del usuario que intenta saltarse el proceso; la solución nunca es forzar el push, sino crear una rama y abrir un Pull Request. En mi repositorio el rechazo afectó incluso a la cuenta administradora porque activé *Do not allow bypassing the above settings*.

### 4. "He aprobado el PR sin mirar los archivos, total ya me fío". ¿Qué riesgo tiene?

La aprobación deja de tener valor: GitHub la cuenta como una garantía de calidad, pero nadie ha comprobado nada. Pueden entrar en `main` errores, archivos que no debían subirse, secretos o cambios que no cumplen el Issue. Además, la descripción del PR puede no coincidir con lo que realmente cambia. Por eso en la segunda revisión volví a la pestaña *Files changed* para comprobar que la línea de tipos incluía `security`, en lugar de fiarme solo de la respuesta "Hecho" de la Autora.

### 5. Si el reviewer pide un cambio y ya habías hecho push, ¿abres un PR nuevo?

No. Un Pull Request no apunta a commits concretos, sino a una rama. Al hacer un nuevo commit en la misma rama y hacer push, el PR existente se actualiza solo: aparece el commit en la pestaña *Commits*, el diff de *Files changed* se recalcula y la conversación conserva todo el historial de la revisión. Así lo hice: el commit de corrección `e6c092c` apareció automáticamente en el PR #2. Abrir un PR nuevo rompería el hilo de la revisión.

### 6. Merge commit, Squash and merge y Rebase and merge. ¿Cuál para commits "wip", "fix", "fix2", "ok ya"?

- **Merge commit:** conserva todos los commits de la rama y añade un commit de fusión que une ambas historias.
- **Squash and merge:** junta todos los commits de la rama en uno solo sobre `main`.
- **Rebase and merge:** copia los commits uno a uno encima de `main`, sin commit de fusión, dejando un historial lineal.

Para una rama con "wip", "fix", "fix2", "ok ya" usaría **Squash and merge**: esos commits no aportan nada por separado y ensuciarían `main`. Es lo que hice en el PR #2: los dos commits de la rama quedaron como un único commit `c8844fd` en `main`.

### 7. ¿Por qué borrar una rama después del merge no elimina el trabajo?

Porque una rama es solo un puntero a un commit; el trabajo vive en los commits. Tras el merge, los cambios ya forman parte de la historia de `main`, así que el puntero de la rama sobra. En mi caso, con squash, el contenido quedó en el commit `c8844fd` de `main`. Por eso `git branch -d` se negaba a borrar la rama local (Git no reconoce el commit de squash como los originales) y tuve que usar `git branch -D`, sabiendo que el contenido ya estaba en `main`.

### 8. ¿Qué debe contener como mínimo la descripción de un Pull Request?

Qué hace el cambio (*Summary*), qué archivos o partes modifica (*Changes*), cómo se ha probado (*Testing*) y qué necesidad resuelve, enlazando el Issue (*Related Issue*, con `Closes #N`). Cuando hay cambios visuales, también capturas del antes y el después. Con eso el Revisor puede entender el cambio sin tener que preguntar.

### 9. Un reviewer escribe solo "esto está mal". ¿Qué le falta? Reescríbelo.

Le falta decir qué está mal, por qué importa, qué propone y si bloquea o no el merge. Es vago y suena a crítica personal. Ejemplo reescrito:

> `issue (blocking): El ejemplo de rama "fix/85- customer-email-validation" tiene un espacio después del guion, y Git no admite espacios en los nombres de rama. Propongo cambiarlo por "fix/85-customer-email-validation".`

### 10. ¿Qué diferencia hay entre proteger main y que el equipo "se ponga de acuerdo"?

Un acuerdo depende de la disciplina y la memoria de cada persona: basta un despiste, una prisa o alguien nuevo para que se rompa. La protección convierte la norma en una regla técnica que GitHub aplica siempre y a todos. Lo vi en la Parte H: aunque yo conocía la norma, el push directo se hizo igualmente y fue GitHub quien lo bloqueó.

### 11. Ejemplo propio: issue (blocking) frente a nitpick (if-minor)

- `issue (blocking): La migración crea la tabla CUSTOMER pero no existe el script de rollback en database/rollback/. Sin él no podemos deshacer el cambio si falla en producción.` → Debe resolverse antes de aprobar.
- `nitpick (if-minor): En el README pone "Oracle Database Lab" y en CONTRIBUTING "oracle-database-lab". Quizá convenga unificar el nombre.` → Detalle menor de estilo; el Autor decide si lo cambia y no impide aprobar.

### 12. Si el próximo commit es `feat!: cambia la firma de la función principal de la API`, ¿qué versión SemVer se dispara?

Una versión **MAJOR** (por ejemplo, de 1.4.2 a 2.0.0). El `!` indica un *breaking change*: cambiar la firma de la función principal obliga a quienes usan la API a modificar su código, porque deja de ser compatible con la versión anterior.

### 13. ¿Por qué abrir un Draft PR desde el primer commit estructural ahorra tiempo al equipo?

Porque permite enseñar pronto el enfoque y recibir feedback antes de invertir horas en una dirección equivocada (*fail fast*). Si el equipo detecta que la estructura o el diseño no encajan, el cambio cuesta poco; si se descubre al final, hay que rehacer mucho trabajo. Además, el Draft deja claro que aún no está listo para fusionar, así que nadie lo aprueba por error.
