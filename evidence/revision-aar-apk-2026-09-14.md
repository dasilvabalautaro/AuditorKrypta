# Revisión de la actualización AAR/APK

**Auditoría:** AK-2026-001

**Fecha:** 14 de septiembre de 2026

**Estado:** cadena AAR publicado → APK arm64 verificada; fuente → AAR parcialmente verificada

## 1. Motivo

Después de congelar Krypta en `a97cbabd95e3787deca872e2157a64473073af6c`, el proyecto añadió dos commits con información de procedencia y publicación de binarios:

| Commit | Fecha | Resumen |
|---|---|---|
| `26bd4056cc90cae9e42b67e5d454742e845d0ed9` | 2026-09-14 07:01:14 BOT | Corrige `build-aar.sh` para un clon limpio y documenta AAR/APK. |
| `d820609195513eb242165366aeecee76c2be4b5a` | 2026-09-14 07:12:32 BOT | Actualiza la documentación tras publicar la release. |

La etiqueta anotada no se movió:

```text
tag object: d6795265cb4f591352506315e2e8ec23eb2a3462
commit:     a97cbabd95e3787deca872e2157a64473073af6c
```

Por ello el alcance criptográfico permanece fijado al mismo commit. Los cambios posteriores se tratan como evidencia suplementaria de build y distribución.

## 2. Revisión de la documentación

Se compararon con la etiqueta:

- `README.md`;
- `CLAUDE.md`;
- `docs/PLAN-privacidad-y-confianza.md`;
- `docs/PRUEBAS-PENDIENTES.md`;
- `docs/REVISION-protocolo-2026-09-14.md`;
- `docs/SOLICITUD-revision-externa.md`;
- `native-bridge/libp2p/build-aar.sh`.

Las afirmaciones principales son coherentes entre documentos:

1. AAR y APK arm64 se construyeron desde un clon limpio de la etiqueta.
2. El script de la etiqueta fallaba si `native-bridge/libs/` no existía.
3. El commit posterior lo corrige con `mkdir -p ../libs`.
4. Los binarios se publicaron en la release `revision-externa-1` con SHA-256.
5. El APK publicado está instalado y fue probado por el autor en su dispositivo.
6. La compilación todavía no es reproducible.

La auditoría confirma 2, 3, 4 y la no reproducibilidad mediante evidencia independiente. Los puntos 1 y 5 son atestaciones del autor: no pueden demostrarse solo desde el binario o la documentación.

Una frase antigua de `infra/fdroid-repo/README.md` afirma que el AAR está «comiteado al repo». Es contradictoria con `.gitignore`, README y el estado real: el AAR no está versionado. Se registra como deuda documental menor, fuera del camino criptográfico.

## 3. Release pública

[Release `revision-externa-1`](https://github.com/dasilvabalautaro/Krypta/releases/tag/revision-externa-1)

Consulta independiente mediante `gh release view`:

| Campo | Valor |
|---|---|
| Estado | publicada; no borrador; prerelease |
| Publicada | `2026-09-14T11:11:21Z` |
| Nombre | `revision-externa-1 — versión sometida a revisión externa` |
| Etiqueta | `revision-externa-1` |

Assets:

| Nombre | Tamaño | Digest informado por GitHub |
|---|---:|---|
| `krypta-p2p-revision-externa-1.aar` | 79.250.313 bytes | `sha256:4b7dcd5130d8bb0c89b4e5bcd2661fea4cbd2e267b777303b2a5d412fb6e49b4` |
| `krypta-arm64-debug-revision-externa-1.apk` | 69.849.412 bytes | `sha256:a30ab852127a6cfde2bcf38d1b56ee10be3a03397d9d6bd7ff0bc76d695576ea` |
| `SHA256SUMS.txt` | 637 bytes | `sha256:f82c9df9b8624bc0b19b41b7d94310a373b762f3f1e85d297442605075b5cae5` |

El APK y `SHA256SUMS.txt` se descargaron y sus hashes recalculados coincidieron. El AAR local presente en Krypta coincide en tamaño y SHA-256 con el asset de GitHub.

## 4. Contenido del AAR

El AAR contiene:

- `classes.jar` y recursos mínimos del binding;
- cuatro `libgojni.so`: arm64-v8a, armeabi-v7a, x86 y x86_64;
- reglas de consumidor ProGuard para `go.**` y `chat.neto.krypta.bridge.**`.

Hashes publicados y recalculados para las bibliotecas nativas:

| ABI | SHA-256 dentro del AAR |
|---|---|
| arm64-v8a | `6b005d71bc257b07707b21a5db96de66236ad222aaa22c1d57e0946fa172ed3b` |
| armeabi-v7a | `3d77f2df73f11a89ab469991197adc8cef33604e7b14feee696a58d23ec2d0e6` |
| x86 | `b503e96170ebb9da5b8f5d420a9c6649dfa44d277584013b8ede4208b1226b10` |
| x86_64 | `a6119c15394e8876592f1968fce0e43434be5c01ab1b6146b1a2d76f0e4bbfd8` |

El JAR denominado `krypta-p2p-sources.jar` contiene las fuentes Java generadas del binding y clases `go.*`; no contiene las fuentes Go originales. Por tanto, no es prueba de correspondencia fuente Go → AAR.

## 5. Vinculación AAR → APK

El APK contiene únicamente estas bibliotecas arm64:

- `libandroidx.graphics.path.so`;
- `libgojni.so`;
- `libsqlcipher.so`.

El `libgojni.so` del AAR mide 41.332.064 bytes y conserva información de depuración. El del APK mide 30.082.912 bytes y está stripped. Ambos comparten:

- el mismo Go Build ID;
- el mismo ELF Build ID SHA-1 `f349cd81207601b9b63cd0651a139509e30ad724`.

Se aplicó la herramienta fijada por el proyecto:

```sh
llvm-strip --strip-unneeded -o libgojni-unneeded.so libgojni-from-aar.so
```

Resultado:

```text
SHA-256 APK:      e1cf58bc0a3a631d7d6c13ed7e5260b73e2fa5a57f9a84902fa4144dfffbcc77
SHA-256 stripped: e1cf58bc0a3a631d7d6c13ed7e5260b73e2fa5a57f9a84902fa4144dfffbcc77
cmp: idénticos
```

Conclusión: el APK publicado incorpora exactamente la biblioteca arm64 del AAR publicado después de la transformación normal `--strip-unneeded` de AGP/NDK.

## 6. Metadatos y firma del APK

| Campo | Valor |
|---|---|
| Paquete | `chat.neto.krypta` |
| `versionName` | `1.5` |
| `versionCode` | `6` |
| minSdk | 30 |
| targetSdk | 36 |
| ABI | arm64-v8a |
| Firma | APK Signature Scheme v2 |
| Certificado | `C=US, O=Android, CN=Android Debug` |
| SHA-256 del certificado | `4a223c5453502aaa62afd7d59c33072d6d75bf7876772c89b7dd9baf35bbd40f` |

`apksigner` devolvió `Verifies`. No hay firma v1, v3, v3.1, v4 ni SourceStamp. Esto coincide con la descripción «debug APK» y no debe confundirse con un binario de producción firmado para Play.

## 7. Vinculación fuente → AAR

`go version -m` sobre `libgojni.so` confirma:

- Go 1.26.4;
- dependencias y sumas coherentes con el `go.mod` de la etiqueta;
- módulo principal de binding `gobind (devel)`;
- dependencia local `chat.neto.krypta/nativego` con versión cero y reemplazo a una ruta local;
- ausencia de hash de commit.

La cadena de dependencias es consistente, pero el binario no contiene una identidad verificable de las fuentes de Krypta. Además, contiene rutas locales y la función de versión devuelve `0.0.18-rdv1pass`, valor anterior a la etiqueta.

Por ello no es posible concluir solo desde el AAR que se compiló desde `a97cbabd…`. La declaración del autor es plausible y la release mejora mucho la auditabilidad, pero hace falta una reconstrucción independiente y comparación semántica, o un build posterior reproducible con commit inyectado.

## 8. Repetición de pruebas con el AAR publicado

La primera línea base Gradle había usado el AAR local anterior `c1b4f1eb…`. Se sustituyó únicamente en la copia aislada por `4b7dcd51…` y se ejecutó:

```sh
gradle --no-daemon --console=plain --rerun-tasks testDebugUnitTest
```

Resultado:

```text
BUILD SUCCESSFUL in 50s
127 actionable tasks: 127 executed
tests=279 skipped=0 failures=0 errors=0
```

## 9. Evaluación de P-001

| Eslabón | Estado | Evidencia |
|---|---|---|
| Etiqueta → commit fuente | Verificado | La etiqueta anotada resuelve a `a97cbabd…`. |
| Release → etiqueta | Verificado | Release pública con nombre/tag `revision-externa-1`. |
| Asset AAR → hash publicado | Verificado | GitHub, `SHA256SUMS.txt` y AAR local coinciden. |
| Asset APK → hash publicado | Verificado | Descarga y recálculo local coinciden. |
| AAR arm64 → APK arm64 | Verificado | Build IDs iguales y stripping reproducido byte por byte. |
| Fuentes del tag → AAR | Pendiente | No hay commit en el binario y el build no es reproducible. |
| APK → dispositivo del autor | No verificado independientemente | Atestación documental del autor. |

P-001 permanece abierta, pero reducida al eslabón fuente → AAR y a la procedencia del dispositivo. No se deriva de esta revisión un hallazgo criptográfico nuevo.
