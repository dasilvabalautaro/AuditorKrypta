# Diario de auditoría

Este documento registra cronológicamente las actividades, decisiones, errores y límites de la auditoría AK-2026-001. Las conclusiones técnicas se desarrollan en documentos separados; el diario conserva la trazabilidad del proceso.

## 14 de septiembre de 2026

### 06:08–06:19 BOT — Preparación del plan

- Se inspeccionó en modo lectura la estructura general de Krypta.
- Se verificó que existían los componentes Kotlin y Go solicitados, documentación de diseño y pruebas relevantes.
- Se fijó provisionalmente el alcance al commit `a97cbabd95e3787deca872e2157a64473073af6c`.
- Se creó `docs/00-plan-auditoria.md` en AuditorKrypta.
- No se modificó Krypta.

### 06:22–06:27 BOT — Configuración de AuditorKrypta y GitHub

- Se revisó `keys-git.md` de forma redactada.
- Se confirmó que contiene una clave pública y que el campo de clave privada está vacío.
- Se verificó la existencia local de una identidad SSH sin leer ni copiar su material secreto.
- Se añadió `keys-git.md` a `.gitignore` junto con rutas para material privado.
- Se inicializó el repositorio local, se preservó el commit remoto inicial y se vinculó `main` con `origin/main`.
- Se publicaron el plan y la restricción de solo lectura en `dasilvabalautaro/AuditorKrypta`.
- HEAD de AuditorKrypta al terminar: `bc98741c4b97988707321daa4ef4ddbb30bc9e02`.

### 06:30 BOT — Inicio de implementación, Fase 0

- Se volvió a confirmar que AuditorKrypta estaba limpio y sincronizado antes de empezar.
- Se identificó el objeto Krypta:
  - commit `a97cbabd95e3787deca872e2157a64473073af6c`;
  - árbol `7ba7212adc5d3be89d316c214decf9378116c8d4`;
  - etiqueta `revision-externa-1`;
  - remoto `git@github.com:dasilvabalautaro/Krypta.git`.
- Dos consultas de `git status --porcelain=v1 --untracked-files=all` no devolvieron cambios.
- Se calculó el SHA-256 `73c697271f99e331f0180e3b3d79b18cd0d838a61eccd87374902c1c7e1b5371` para `git archive --format=tar HEAD`.
- Se registraron versiones de sistema, Java, Go, Git, OpenSSL, Gradle, AGP y Kotlin.
- Se inventariaron 269 archivos rastreados: 133 Kotlin, 39 Go y 22 Markdown.
- Se identificaron y hashearon el AAR del puente, su JAR de fuentes y el AAB local. Los tres están ignorados por Git.
- Se abrió la cuestión de procedencia P-001: el commit no contiene el AAR consumido por Android, por lo que todavía no está demostrada su correspondencia con las fuentes.
- Se comprobó que no hay submódulos declarados. Git LFS no está instalado; la comprobación se repetirá sobre la copia aislada.

### Incidente operacional F0-I01

Al solicitar información de entorno se incluyó `./gradlew --version` desde Krypta. El wrapper intentó abrir el bloqueo:

```text
~/.gradle/wrapper/dists/gradle-9.4.1-bin/.../gradle-9.4.1-bin.zip.lck
```

El sandbox lo denegó con `Operation not permitted`; Gradle no llegó a arrancar. El estado versionado de Krypta permaneció limpio.

Decisión adoptada: no volver a ejecutar herramientas de compilación o prueba desde Krypta. Toda ejecución se realizará sobre una copia aislada fijada al commit objetivo, con sus cachés fuera del proyecto original.

### Artefactos documentales creados

- `docs/01-manifiesto-y-entorno.md`
- `docs/diario-auditoria.md`
- entrada `.audit-work/` en `.gitignore` para futuras copias desechables locales.

### Siguiente paso

Crear una copia aislada exacta del commit, verificar su hash, configurar caches confinadas y ejecutar la línea base de pruebas sin tocar Krypta.

### 06:32–06:44 BOT — Copia aislada y línea base

- Se extrajo el archivo Git del commit objetivo en `.audit-work/krypta-a97cbabd`.
- Se verificaron los 269 archivos contra sus blobs Git: cero discrepancias.
- Se añadieron solo a la copia el AAR local, su JAR de fuentes y `local.properties`.
- Se confinaron `GOPATH`, `GOMODCACHE`, `GOCACHE` y `GRADLE_USER_HOME` bajo `.audit-work/cache/`.
- Se descargaron las dependencias fijadas de los dos módulos Go.
- `go test -count=1 ./...` pasó en el puente y el nodo.
- `go test -race -count=1 ./...` pasó en el puente y el nodo en la repetición válida con loopback permitido.
- Gradle 9.4.1 ejecutó `testDebugUnitTest`: 279 pruebas pasaron, sin omisiones, fallos ni errores; 127 tareas fueron ejecutadas.
- Se identificaron 49 funciones de prueba en el puente y 24 en el nodo.
- Diez pruebas live del puente quedaron omitidas por falta de sus variables/infraestructura deliberada.
- Se volvió a comprobar el árbol original de Krypta: cero entradas en `git status --porcelain=v1 --untracked-files=all`.

### Incidente operacional F0-I02

La primera comparación archivo por archivo utilizó `path` como nombre de variable en zsh. `path` es una variable especial enlazada a `PATH`, por lo que el bucle dejó de encontrar `git` y reportó 269 falsas discrepancias.

Se descartó por completo esa salida. La comprobación se repitió usando `file_name` y produjo `checked=269 mismatches=0`.

### Incidente operacional F0-I03

La primera ejecución de `go test -race` no recibió acceso a sockets locales. Las pruebas que crean hosts libp2p fallaron con `bind: operation not permitted`.

Se clasificó como restricción del entorno. La ejecución se repitió sin cambiar código, con loopback permitido, y ambas suites pasaron con código 0.

### Incidente operacional F0-I04

El primer intento de contar funciones Go ejecutó `go test -list` desde la raíz, que no es un módulo Go. Devolvió cero y el error `cannot find main module`. Se descartó el conteo y se repitió desde cada módulo: 49 funciones en el puente y 24 en el nodo.

### Observación de pruebas T-001

Una repetición de la suite del puente en formato JSON falló una vez en `TestMailboxRedeliverUnacked`; esa misma prueba había pasado en la línea base normal y bajo `-race`.

La ejecución dirigida pasó 20 repeticiones y después 200 repeticiones adicionales. Estado: **no reproducido; vigilar durante la revisión de ack/redelivery y concurrencia**. No se clasifica como hallazgo de seguridad.

### Próximo paso actualizado

Analizar la procedencia del AAR y su JAR de fuentes. Después comenzar la reconstrucción independiente de la especificación y el modelo de amenazas.
