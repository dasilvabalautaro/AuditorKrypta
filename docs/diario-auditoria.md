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

### 07:12–07:19 BOT — Actualización documental y binarios publicados

- Se detectó que Krypta avanzó dos commits después de la etiqueta auditada:
  - `26bd4056…`: corrección de `build-aar.sh` para crear `../libs` en un clon limpio;
  - `d8206091…`: documentación de la release ya publicada.
- Se revisaron los diffs completos de README, solicitud externa, plan de confianza, pruebas pendientes, revisión del protocolo y notas del proyecto.
- Se confirmó que `revision-externa-1` es una etiqueta anotada cuyo commit resuelto sigue siendo `a97cbabd…`.
- Mediante la API de GitHub a través de `gh release view`, en modo lectura, se verificó que la release es pública, no borrador y prerelease, publicada a las `2026-09-14T11:11:21Z`.
- Se verificaron tres assets: APK arm64, AAR y `SHA256SUMS.txt`; sus digests de GitHub coinciden con la documentación.
- Se descargaron el APK y `SHA256SUMS.txt` únicamente en `.audit-work/` y se recalcularon sus hashes correctamente.
- El AAR local actual coincide con el asset publicado: SHA-256 `4b7dcd51…49b4`, 79.250.313 bytes.
- El APK coincide con el asset publicado: SHA-256 `a30ab852…76ea`, 69.849.412 bytes.
- Se comprobó que el APK es `chat.neto.krypta` 1.5 (`versionCode` 6), minSdk 30, targetSdk 36 y solo contiene ABI arm64.
- `apksigner` confirmó firma válida v2 con un certificado `CN=Android Debug`; no hay v1, v3, v3.1, v4 ni SourceStamp.
- El `.so` arm64 del AAR y el del APK comparten Go Build ID y ELF Build ID.
- Aplicar `llvm-strip --strip-unneeded` del NDK 26.1 al `.so` del AAR produjo byte por byte el `.so` del APK, SHA-256 `e1cf58bc…cc77`.
- `go version -m` confirmó Go 1.26.4, dependencias coherentes con `go.mod`, módulo `(devel)`, ruta local embebida y ausencia de commit.
- Se actualizó P-001 a **parcialmente resuelta**: la cadena AAR publicado → APK arm64 está demostrada; fuente etiquetada → AAR sigue dependiendo de atestación/no reproducibilidad.
- La copia aislada sustituyó el AAR antiguo por el publicado. Gradle repitió las 127 tareas con `--rerun-tasks`: 279 pruebas pasaron, sin omisiones ni fallos.
- Krypta original volvió a comprobarse limpio.

### Incidente operacional F0-I05

El primer intento de repetir Gradle con el AAR publicado se ejecutó sin acceso a sockets locales. Gradle terminó antes de configurar el proyecto porque `FileLockContentionHandler` no pudo crear su socket. Se descartó y se repitió con el permiso local necesario; el build pasó.

### Próximo paso

Intentar una reconstrucción independiente del AAR desde la etiqueta y diseñar una comparación semántica que tolere las rutas y metadatos no reproducibles. Después cerrar o acotar definitivamente P-001.

### 07:25–08:05 BOT — Revisión de actualizaciones y reconstrucción AAR posterior

- Se revisó la documentación nueva de Krypta hasta `ea088b1f…` y se confirmó que el tag `revision-externa-1` no cambió.
- Se inspeccionó el cambio `8d028754…`: NDK 26.1 fijo, `-trimpath`, commit inyectado, comprobaciones de 16 KB y ausencia de rutas locales.
- Se construyó el AAR en dos clones Git temporales, con cachés Go independientes y sin ejecutar herramientas en Krypta.
- Ambas ejecuciones pasaron las comprobaciones de los cuatro ABIs y produjeron SHA-256 idéntico `d817bae1…f4e0`.
- La release pública sigue conteniendo únicamente los binarios antiguos de `revision-externa-1`; no se observó un asset público para el AAR/APK posterior.
- P-001 queda cerrada para la cadena posterior `8d028754… → AAR` en este anfitrión, pero abierta para los binarios históricos y para reproducibilidad cross-host/APK.
- Se registró deuda documental menor en `infra/fdroid-repo/README.md` (referencia al AAR como rastreado).

### 08:10–08:35 BOT — Inicio de Fase 1: reconstrucción de la especificación

- Se relevaron las derivaciones HKDF/HMAC/AES-GCM, campos de cabecera, estados y límites del ratchet desde la especificación y el código del commit congelado.
- Se normalizaron capacidades, claves de llamada y etiquetas de buzón en una matriz de trazabilidad.
- Se confirmó que los frames de llamada no llevan contador; queda como candidato W-8 para la revisión dinámica.
- Se abrió C-001: un timestamp futuro en un invite no se rechaza por la comprobación de antigüedad; queda pendiente determinar impacto de replay/disponibilidad.
- No se modificó Krypta. La evidencia se guardó en `evidence/especificacion-normalizada-fase1-2026-09-14.md`.

### 08:40–09:20 BOT — Verificación dinámica W-8 y C-001

- Se ejecutó una prueba externa en la copia aislada: el mismo frame AES-GCM se descifra dos veces correctamente, confirmando que el canal de medios no tiene anti-replay por contador.
- Se ejecutó una prueba externa con `invite` E2EE cuyo timestamp está una hora en el futuro; el receptor lo aceptó y pasó a `RINGING`.
- W-8 queda abierto como debilidad de frescura/orden; C-001 queda como candidato de disponibilidad/replay, pendiente de medir deduplicación e impacto de llamadas repetidas.
- La suite dirigida terminó con `BUILD SUCCESSFUL`; el árbol original de Krypta permaneció limpio.
- Se documentó el lote en `docs/08-verificacion-dinamica.md`.

### 09:30–11:55 BOT — Revisión del commit remoto `fe21111f`

- GitHub mostró un commit nuevo posterior a `ea088b1f…`: `fe21111f…`, con H-7 y pruebas de transporte relayed.
- Se creó un clon temporal aislado y se fijó al commit remoto; no se ejecutó ninguna herramienta en Krypta original.
- Pasaron `TestRelayNoVeLoQueViajaPorElCircuito`, las pruebas Go de transporte y `go test -count=1 ./...` del puente.
- Pasaron las pruebas JVM de `CallServiceTest` y `ChatServiceTest` (`BUILD SUCCESSFUL`).
- H-7 queda corregido con deduplicación en memoria y ventana futura acotada; la pérdida de deduplicación tras reinicio permanece como limitación.
- W-8 queda acotado: el relay relayed no ve ni puede repetir frames dentro del circuito; la ausencia de contador en AES-GCM sigue siendo una propiedad de la primitiva, no una explotación demostrada contra ese relay.
- C-001 debe reformularse como replay v1 sin deduplicación persistente por `callId`; el timestamp futuro es secundario.
- Los cambios documentales están sin commit, por instrucción del propietario.

### 12:00–12:25 BOT — Revisión de código Kotlin/Go posterior

- Se trazaron `Ratchet`, `RatchetSessions`, `ChatService`, `CallService`, `MailboxLabel` y el framing Go contra `fe21111f…`.
- H-7 y C-001 aparecen corregidos en Kotlin; la deduplicación de invites sigue siendo volátil y acotada a 256 entradas.
- W-8 queda limitado por la capa Noise/TLS del circuito relayed, según la prueba Go del commit actualizado.
- Se añadieron notas de revisión por lenguaje y se mantiene la separación entre propiedades de la primitiva AES-GCM y ataques posibles contra el transporte.

### 12:30–13:00 BOT — Consolidación de la revisión de diseño

- Se consolidaron H-4 (secuestro por linaje) y H-5 (KCI/autenticación basada solo en `S`) como limitaciones de diseño abiertas.
- Se separaron las condiciones W-1/W-2/W-6 de los problemas operativos ya corregidos.
- W-8 quedó descrito como propiedad del transporte relayed verificado, no como contador de aplicación.
- Se añadieron recomendaciones priorizadas y referencias cruzadas al informe publicable.
