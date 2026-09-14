# Evidencia de línea base de pruebas — 14 de septiembre de 2026

**Auditoría:** AK-2026-001

**Objeto fuente:** `a97cbabd95e3787deca872e2157a64473073af6c`

**Copia:** `.audit-work/krypta-a97cbabd` — local, desechable y excluida de Git

## 1. Integridad de la copia

La copia se generó mediante `git archive` y se verificó comparando cada archivo con el blob esperado:

```text
checked=269 mismatches=0
```

Se repitió la verificación después de las pruebas con el mismo resultado. El estado del repositorio Krypta original devolvió cero entradas antes y después.

Los artefactos no versionados se añadieron después de verificar los 269 archivos. La primera captura usó:

```text
c1b4f1eb2a7eada06ff536fad396d4f31ba5ddb8dca20c73984491c913f1a400  native-bridge/libs/krypta-p2p.aar
fd26966fe8d0f0bb016f7f0bccc7610211aa680c0896366fa3482acd68bf287e  native-bridge/libs/krypta-p2p-sources.jar
```

Más tarde el proyecto publicó y sustituyó el AAR por el construido para la etiqueta:

```text
4b7dcd5130d8bb0c89b4e5bcd2661fea4cbd2e267b777303b2a5d412fb6e49b4  native-bridge/libs/krypta-p2p.aar
```

La copia aislada se actualizó con ese AAR y la suite Gradle completa se repitió con `--rerun-tasks`.

## 2. Aislamiento

Variables utilizadas para impedir escrituras en Krypta y separar cachés:

```sh
GOTOOLCHAIN=local
GOTELEMETRY=off
GOPATH=<AuditorKrypta>/.audit-work/cache/gopath
GOMODCACHE=<AuditorKrypta>/.audit-work/cache/gomod
GOCACHE=<AuditorKrypta>/.audit-work/cache/gobuild
GRADLE_USER_HOME=<AuditorKrypta>/.audit-work/cache/gradle
```

Go se ejecutó con `go1.26.4 darwin/amd64`. El módulo del nodo declara Go 1.22.0/toolchain 1.22.12, por lo que queda pendiente repetir su línea base con el toolchain exacto.

## 3. Suites Go

Comando normal, ejecutado desde cada módulo:

```sh
go test -count=1 ./...
```

Resultados válidos iniciales:

```text
ok  chat.neto.krypta/nativego    25.542s
ok  chat.neto.krypta/infra/node   2.905s
```

Comando con detector de carreras:

```sh
go test -race -count=1 ./...
```

Resultados válidos, con sockets loopback permitidos:

```text
ok  chat.neto.krypta/nativego    28.085s
ok  chat.neto.krypta/infra/node   4.724s
```

Inventario mediante `go test -list '^Test' .`:

| Módulo | Funciones `Test*` |
|---|---:|
| `chat.neto.krypta/nativego` | 49 |
| `chat.neto.krypta/infra/node` | 24 |

Una ejecución JSON posterior del puente produjo:

```text
pass=40 skip=10 fail=1
failed-tests=TestMailboxRedeliverUnacked
```

El conteo incluye dos subtests, por eso no se compara directamente con las 49 funciones superiores. Las diez omisiones fueron sondas live:

- `TestBlindMailboxAgainstLiveNode`
- `TestIPLeakAgainstLiveNode`
- `TestIPLeakViaRelayDial`
- `TestMailboxPutAgainstLiveNode`
- `TestMailboxFetchAgainstLiveNode`
- `TestMailboxRoundTripAgainstLiveNode`
- `TestPingAgainstLiveNode`
- `TestRelayLimitsAgainstLiveNode`
- `TestReserveAgainstLocalNode`
- `TestWakeAgainstLiveNode`

El fallo aislado no se reprodujo en:

```sh
go test -run '^TestMailboxRedeliverUnacked$' -count=20 -v .
go test -run '^TestMailboxRedeliverUnacked$' -count=200 .
```

Ambos comandos pasaron: 220/220 repeticiones. Se registra como observación T-001, no como hallazgo.

La ejecución JSON del nodo produjo:

```text
pass=24 skip=0 fail=0
```

## 4. Suite Gradle/JVM

Se utilizó directamente la instalación local de Gradle 9.4.1 con caché aislada:

```sh
gradle --no-daemon --console=plain testDebugUnitTest
```

Resultado:

```text
BUILD SUCCESSFUL in 3m 10s
127 actionable tasks: 127 executed
```

Resumen extraído de los XML JUnit:

```text
tests=279 skipped=0 failures=0 errors=0
```

### Repetición con el AAR publicado

```sh
gradle --no-daemon --console=plain --rerun-tasks testDebugUnitTest
```

```text
BUILD SUCCESSFUL in 50s
127 actionable tasks: 127 executed
tests=279 skipped=0 failures=0 errors=0
```

Un primer intento sin acceso a sockets locales terminó antes del build porque Gradle no pudo crear `FileLockContentionHandler`. Se descartó como limitación del sandbox y se repitió correctamente sin cambiar código.

Suites directamente relacionadas con el alcance:

| Suite | Pruebas | Resultado |
|---|---:|---|
| `AesGcmMessageCipherTest` | 5 | pasa |
| `CallServiceTest` | 11 | pasa |
| `ChatServiceTest` | 88 | pasa |
| `HkdfTest` | 3 | pasa |
| `IdentityBackupTest` | 7 | pasa |
| `KemTest` | 9 | pasa |
| `MailboxLabelTest` | 6 | pasa |
| `MessageEnvelopeTest` | 13 | pasa |
| `PaddingTest` | 8 | pasa |
| `ParserFuzzTest` | 5 | pasa |
| `RatchetPaddingTest` | 6 | pasa |
| `RatchetPropertyTest` | 3 | pasa |
| `RatchetSessionsTest` | 7 | pasa |
| `RatchetTest` | 19 | pasa |
| `RatchetSeenPruneSqlTest` | 4 | pasa |
| `IdentityStoreTest` | 7 | pasa |

Las advertencias de compilación no se trataron como fallos. Incluyeron uso sin opt-in de `ExperimentalCoroutinesApi` en pruebas, cambios futuros de destino de anotaciones y APIs obsoletas en componentes de aplicación.

## 5. Intentos descartados

No se utilizan como evidencia de defectos:

1. `./gradlew --version` desde Krypta: bloqueado antes de iniciar por acceso denegado a la caché global.
2. Primer `go test -race`: sin permiso de `bind` en loopback; repetido correctamente y aprobado.
3. Primera comparación de blobs: el nombre de variable zsh `path` alteró `PATH`; repetida con cero discrepancias.
4. Primer conteo `go test -list`: ejecutado desde una raíz sin `go.mod`; repetido desde ambos módulos.

## 6. Interpretación

Esta línea base demuestra que las pruebas locales existentes pueden compilarse y, salvo la observación T-001, pasar sobre la copia congelada con los artefactos locales aportados. No demuestra que:

- las pruebas cubran todas las propiedades criptográficas;
- el AAR corresponda a las fuentes;
- las sondas live pasen;
- las pruebas instrumentadas pasen en Android;
- no existan carreras fuera de los caminos ejercitados;
- la compilación sea reproducible en otro entorno.
