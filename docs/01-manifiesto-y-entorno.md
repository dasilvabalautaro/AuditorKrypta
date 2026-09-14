# Manifiesto del objeto auditado y del entorno

**Auditoría:** AK-2026-001

**Fase:** 0 — congelación y reproducibilidad

**Captura:** 14 de septiembre de 2026, 10:30:57 UTC

**Estado:** línea base completada; correspondencia fuente–AAR posterior verificada para el commit `8d028754…`

## 1. Regla de preservación

Por instrucción expresa del propietario, `/Users/davidsilva/AndroidStudioProjects/Krypta` y el repositorio remoto `dasilvabalautaro/Krypta` son estrictamente de solo lectura. Ninguna prueba, compilación o herramienta que pueda escribir archivos se ejecutará en ese directorio.

Las pruebas se ejecutarán sobre una copia aislada fijada al commit objetivo. Los resultados y cualquier código auxiliar se conservarán únicamente en `AuditorKrypta`.

## 2. Identidad del objeto auditado

| Campo | Valor |
|---|---|
| Repositorio local leído | `/Users/davidsilva/AndroidStudioProjects/Krypta` |
| Remoto declarado | `git@github.com:dasilvabalautaro/Krypta.git` |
| Rama observada | `main` |
| Commit | `a97cbabd95e3787deca872e2157a64473073af6c` |
| Árbol Git | `7ba7212adc5d3be89d316c214decf9378116c8d4` |
| Padre | `8c4d87418a6a3d9ef99d9bf2a55a92ad0f9c8283` |
| Fecha del commit | `2026-09-14T06:00:53-04:00` |
| Asunto | `fix(cripto): revisión del protocolo — clave y nonce repetidos, mensajes perdidos y versión que bajaba` |
| Etiqueta en HEAD | `revision-externa-1` |
| SHA-256 de `git archive --format=tar HEAD` | `73c697271f99e331f0180e3b3d79b18cd0d838a61eccd87374902c1c7e1b5371` |

El árbol versionado no presentaba modificaciones ni archivos no rastreados según `git status --porcelain=v1 --untracked-files=all` en las dos comprobaciones de esta captura.

La etiqueta es una referencia conveniente, pero todas las conclusiones se anclarán al hash completo del commit.

## 3. Contenido versionado

| Medida | Cantidad |
|---|---:|
| Archivos rastreados | 269 |
| Archivos Kotlin | 133 |
| Archivos Go | 39 |
| Archivos Markdown | 22 |
| Pruebas rastreadas, según patrón de rutas/nombres | 71 |
| Candidatos relacionados con criptografía y protocolo, según filtro nominal amplio | 81 |

Distribución inicial por módulo:

| Módulo | Archivos rastreados | Kotlin | Go |
|---|---:|---:|---:|
| `app` | 76 | 48 | 0 |
| `core` | 18 | 17 | 0 |
| `data` | 31 | 24 | 0 |
| `native-bridge` | 42 | 6 | 30 |
| `p2p-signaling` | 39 | 38 | 0 |
| `infra/node` | 21 | 0 | 9 |

Los conteos ayudan a dimensionar el trabajo; no definen por sí solos el alcance. El inventario criptográfico posterior resolverá clases, funciones y líneas exactas.

No se observaron entradas Git con modo `160000`, por lo que el commit no declara submódulos. Git LFS no está instalado en el entorno; no se observaron punteros LFS durante esta captura, pero esta comprobación se repetirá de forma explícita sobre el árbol aislado.

## 4. Toolchain observado y declarado

### 4.1 Entorno anfitrión

| Componente | Versión observada |
|---|---|
| Sistema | macOS 26.6.2, build `25G83`, Darwin `25.6.0`, x86_64 |
| Java | OpenJDK `25.0.1+8-27` |
| Go | `go1.26.4 darwin/amd64` |
| Git | `2.50.1 (Apple Git-155)` |
| OpenSSL | `3.6.3`, biblioteca `3.6.3` |

### 4.2 Versiones declaradas por el proyecto

| Componente | Versión |
|---|---|
| Gradle wrapper | `9.4.1` |
| Android Gradle Plugin | `9.2.1` |
| Kotlin | `2.2.10` |
| Coroutines | `1.11.0` |
| SQLCipher Android | `4.6.1` |
| Room | `2.8.4` |
| Go del puente nativo | `1.26` |
| `golang.org/x/crypto` del puente | `0.53.0` |
| go-libp2p del puente | `0.48.0` |
| Go del nodo de infraestructura | `1.22.0`, toolchain `1.22.12` |
| go-libp2p del nodo | `0.38.1` |

La diferencia deliberada entre versiones de go-libp2p del cliente y del nodo se tratará como una frontera de interoperabilidad. No constituye por sí sola un hallazgo.

## 5. Artefactos binarios relevantes presentes en el directorio local

Los siguientes archivos existen físicamente, pero están ignorados y no forman parte del commit objetivo:

| Artefacto | Tamaño | SHA-256 | Fecha de modificación observada |
|---|---:|---|---|
| `native-bridge/libs/krypta-p2p.aar` | 79,250,831 bytes | `c1b4f1eb2a7eada06ff536fad396d4f31ba5ddb8dca20c73984491c913f1a400` | `2026-09-12T07:28:50-04:00` |
| `native-bridge/libs/krypta-p2p-sources.jar` | 17,283 bytes | `fd26966fe8d0f0bb016f7f0bccc7610211aa680c0896366fa3482acd68bf287e` | `2026-09-12T07:28:49-04:00` |
| `app/release/krypta.aab` | 81,898,484 bytes | `62bcd74994319758f4e299ee680610c0d92804468e972979e3737d8c36ee7b64` | `2026-07-12T20:19:35-04:00` |

También existen numerosos productos bajo directorios `build/`, todos ignorados. No se utilizarán como sustituto del código fuente ni como evidencia de una compilación reproducible.

### Cuestión de procedencia P-001 — parcialmente resuelta

El módulo Android declara una dependencia de archivo sobre `native-bridge/libs/krypta-p2p.aar`, pero el AAR está excluido de Git. Por tanto, el commit fuente por sí solo no determina los bytes del componente Go que consume Android.

Esto se registra como **cuestión de procedencia**, no como vulnerabilidad. Una actualización posterior publicó el AAR y el APK arm64 de la etiqueta. La revisión independiente de esa actualización se encuentra en `evidence/revision-aar-apk-2026-09-14.md`.

Quedó demostrado que:

- la release pública está unida a la etiqueta que resuelve al commit auditado;
- el AAR local actual coincide en tamaño y SHA-256 con el asset publicado;
- el APK publicado coincide con su digest y suma declarada;
- el `libgojni.so` arm64 del APK es exactamente el resultado de aplicar `llvm-strip --strip-unneeded` al del AAR publicado.

Sigue sin quedar demostrado a partir del binario que el AAR fue producido por las fuentes del commit: lleva el módulo como `(devel)`, una ruta local y ningún identificador del commit. La afirmación «compilado desde un clon limpio de la etiqueta» sigue dependiendo de la atestación del autor y de una futura comparación/reconstrucción no bit a bit.

Para cerrar P-001 será necesario:

1. inspeccionar el contenido y metadatos del AAR;
2. comparar las fuentes adjuntas con `native-bridge/libp2p`;
3. reconstruir el AAR desde el commit congelado en el entorno aislado;
4. comparar artefactos o, si la compilación no es bit a bit reproducible, comparar símbolos, API, fuentes y comportamiento;
5. establecer una vinculación verificable entre las fuentes y el AAR, aun cuando la compilación actual no sea reproducible bit a bit.

### Actualización observada después del commit congelado

El 14 de septiembre de 2026 se observaron dos commits posteriores:

| Commit | Propósito |
|---|---|
| `26bd4056cc90cae9e42b67e5d454742e845d0ed9` | Añade `mkdir -p ../libs` a `build-aar.sh` y documenta los binarios construidos desde la etiqueta. |
| `d820609195513eb242165366aeecee76c2be4b5a` | Documenta la publicación de la release con AAR, APK y sumas. |

El único cambio ejecutable de estos commits dentro de la ruta revisada es la creación del directorio de salida en el script de build; no cambia el código criptográfico ni el contenido de la etiqueta. El objeto de auditoría se mantiene en `a97cbabd…`; los dos commits se aceptan como evidencia documental suplementaria.

### Actualización observada el 14 de septiembre (HEAD posterior)

La rama `main` de Krypta avanzó hasta `ea088b1f7bc8d029f708b73040d09d42c7e02536`, dos commits posteriores a la captura congelada. El commit ejecutable relevante es `8d028754de6ee16085239c027a0d34babe05ae0b`, que rehace `build-aar.sh` para una salida reproducible, fija NDK 26.1.10909125/JDK 25, usa `-trimpath`, limita el proguard y enlaza `buildCommit` con el SHA de Git (o `-modificado`). El commit `ea088b1f…` documenta la comprobación del autor y corrige la afirmación del NDK de los binarios antiguos.

Se revisó la documentación y se hizo una reconstrucción independiente en dos clones temporales con Git, cachés Go separadas y el mismo commit `8d028754…`. Ambas ejecuciones terminaron correctamente y produjeron el mismo AAR:

```text
d817bae1d5cfec41cddb8f1213469d9c78f05789806d992ba2c408c2f024f4e0
```

Los cuatro ABIs incluyeron el commit esperado, no conservaron rutas locales y respetaron alineación ELF de 16 KB. Esto verifica de forma independiente la cadena fuente posterior `8d028754… → AAR`, pero no cambia el objeto congelado ni demuestra la reproducibilidad del AAR/APK antiguos de `revision-externa-1`. La release pública sigue sirviendo esos binarios antiguos (`4b7dcd…` AAR y `a30ab8…` APK); el APK nuevo mencionado en la documentación no está publicado como asset verificable.

La limitación residual es de alcance: la reconstrucción se hizo en un único Mac y no cubre reproducibilidad del APK ni de otra máquina. La evidencia detallada está en `evidence/revision-aar-reproducible-8d-2026-09-14.md`.

## 6. Integridad del wrapper

| Archivo rastreado | SHA-256 |
|---|---|
| `gradle/wrapper/gradle-wrapper.jar` | `76805e32c009c0cf0dd5d206bddc9fb22ea42e84db904b764f3047de095493f3` |
| `gradle/wrapper/gradle-wrapper.properties` | `b8499696dbb33f8032dfbb6f8403565da2ab923c99745b9067c1fab63e25e17f` |

La comprobación de estos hashes solo fija los bytes observados. La autenticidad del JAR y de la distribución Gradle se evaluará antes de confiar en el wrapper.

## 7. Copia aislada y línea base de ejecución

Se extrajo `git archive` del commit objetivo en la ruta ignorada `.audit-work/krypta-a97cbabd`. La copia contiene los 269 archivos versionados y no contiene metadatos Git ni referencia mutable a Krypta.

Se comparó el blob Git esperado con `git hash-object` para cada archivo extraído:

```text
checked=269 mismatches=0
```

Después de esa comprobación se copiaron por separado el AAR, el JAR de fuentes y `local.properties`. Los hashes del AAR y del JAR copiados coinciden con la sección 5. Las cachés de Go y Gradle se confinaron bajo `.audit-work/cache/`.

### 7.1 Resultados

| Suite | Comando conceptual | Resultado válido |
|---|---|---|
| Puente Go | `go test -count=1 ./...` | pasa |
| Nodo Go | `go test -count=1 ./...` | pasa |
| Puente Go con detector de carreras | `go test -race -count=1 ./...` | pasa |
| Nodo Go con detector de carreras | `go test -race -count=1 ./...` | pasa |
| Unitarias Gradle debug con AAR publicado | `gradle --no-daemon --rerun-tasks testDebugUnitTest` | 279 pasan; 0 omitidas; 0 fallos; 0 errores |

El puente declara 49 funciones de prueba y el nodo 24. En una ejecución JSON del puente se omitieron correctamente diez sondas que exigen infraestructura live. El nodo ejecutó 24 pruebas sin omisiones.

Una repetición JSON del puente observó un fallo aislado en `TestMailboxRedeliverUnacked`. La prueba pasó antes, pasó bajo `-race` y después pasó 220 repeticiones dirigidas consecutivas. Se conserva como observación de estabilidad **T-001**, no como vulnerabilidad ni fallo confirmado.

La primera ejecución Gradle usó el AAR local anterior `c1b4…`, porque el autor lo sustituyó después de capturar la línea base. Se corrigió la copia aislada con el AAR publicado `4b7d…` y se forzó la ejecución de las 127 tareas: las mismas 279 pruebas pasaron. Las ejecuciones produjeron advertencias de compilación relativas a APIs experimentales, destinos futuros de anotaciones y APIs obsoletas. No impidieron la compilación ni las pruebas; se revisarán únicamente si afectan al camino auditado.

### 7.2 Intentos no válidos

Se intentó inicialmente consultar `./gradlew --version` desde Krypta. El wrapper intentó abrir un archivo de bloqueo en la caché global `~/.gradle` y el sandbox denegó la operación antes de iniciar Gradle. El intento no produjo cambios rastreados en Krypta.

La primera ejecución `-race` se lanzó sin permiso de sockets locales y falló al hacer `bind`. Se repitió con loopback habilitado y ambas suites pasaron. Los fallos del primer intento son limitaciones del sandbox y no resultados de Krypta.

Los detalles y comandos se encuentran en `evidence/linea-base-pruebas-2026-09-14.md`.

## 8. Comandos de identificación empleados

Los comandos fueron de solo lectura respecto de Krypta:

```sh
git rev-parse HEAD
git show -s --format=... HEAD
git status --porcelain=v1 --untracked-files=all
git branch --show-current
git remote -v
git tag --points-at HEAD
git archive --format=tar HEAD | shasum -a 256
git ls-files
git ls-files -s
shasum -a 256 <artefactos seleccionados>
git check-ignore -v <artefactos seleccionados>
```

El intento fallido de `./gradlew --version` queda documentado en la sección anterior y en el diario.

## 9. Condiciones para cerrar la Fase 0

- [x] Commit, árbol, padre, etiqueta y remoto identificados.
- [x] Estado versionado limpio confirmado.
- [x] Hash canónico del archivo del commit registrado.
- [x] Toolchains observados y declarados registrados.
- [x] Artefactos binarios locales relevantes identificados y hasheados.
- [x] Restricción de solo lectura operacionalizada.
- [x] Copia de ejecución aislada creada y verificada.
- [x] Línea base de pruebas local ejecutada en la copia.
- [x] Correspondencia fuente–AAR evaluada para el build posterior `8d028754…`.

La procedencia del AAR de la etiqueta congelada sigue siendo una cuestión histórica separada; no se cierra con la reconstrucción posterior.

La correspondencia fuente–AAR es la actividad pendiente de la Fase 0. Las sondas live, pruebas instrumentadas en dispositivo y reproducción por una segunda máquina quedan fuera de esta línea base local y se registrarán como trabajo posterior.
