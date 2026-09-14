# Evidencia: AAR reproducible desde `8d028754`

**Auditoría:** AK-2026-001  
**Fecha:** 14 de septiembre de 2026  
**Alcance:** revisión documental y reconstrucción independiente; Krypta original siempre se trató como solo lectura.

## Contexto

La rama `main` observada de Krypta está en `ea088b1f7bc8d029f708b73040d09d42c7e02536`. El cambio de build que se puede evaluar es `8d028754de6ee16085239c027a0d34babe05ae0b` (`feat(puente): AAR reproducible y ligado a su commit`). El tag auditado `revision-externa-1` continúa resolviendo a `a97cbabd…` y no se mueve.

La documentación nueva fija NDK `26.1.10909125`, JDK 25, Go 1.26.4, `-trimpath`, páginas de 16 KB y un `buildCommit` enlazado mediante `-ldflags`. También marca el AAR como suprimido de Git y limita las reglas de proguard a `go.**` y `chat.neto.krypta.bridge.**`.

## Procedimiento

Se crearon dos clones temporales con Git bajo `.audit-work/`, ambos en el commit `8d028754…`, cada uno con `GOCACHE` y `GOPATH` propios. Se ejecutó `native-bridge/libp2p/build-aar.sh` con `ANDROID_HOME` del SDK local y `GOMODCACHE` compartida solo para dependencias. No se ejecutó ningún build en el directorio Krypta original.

## Resultado

Las dos ejecuciones terminaron con código 0. Hash SHA-256 del AAR en ambos clones:

```text
d817bae1d5cfec41cddb8f1213469d9c78f05789806d992ba2c408c2f024f4e0
```

Las comprobaciones incorporadas al script reportaron, para `arm64-v8a`, `armeabi-v7a`, `x86` y `x86_64`:

- páginas ELF de 16 KB;
- commit `8d028754de6ee16085239c027a0d34babe05ae0b` presente;
- cero rutas locales embebidas.

El AAR resultante tiene `.comment` de LLVM/clang 17.0.2 (NDK 26.1), en contraste con el AAR de la release antigua, que fue construido con clang 14.0.7/NDK 25.2. La release pública todavía contiene el AAR antiguo `4b7dcd5130d8bb0c89b4e5bcd2661fea4cbd2e267b777303b2a5d412fb6e49b4` y el APK antiguo `a30ab852127a6cfde2bcf38d1b56ee10be3a03397d9d6bd7ff0bc76d695576ea`.

## Evaluación

La igualdad byte a byte entre dos clones y cachés independientes confirma la afirmación de reproducibilidad del AAR para el commit `8d028754…` en este anfitrión. La evidencia no prueba reproducibilidad cross-host, ni del APK, ni de los binarios históricos de `revision-externa-1`. Por ello P-001 queda cerrada para el build posterior y abierta como cuestión histórica para la release congelada.

Se observó además una frase obsoleta en `infra/fdroid-repo/README.md` que describe el AAR como “comiteado al repo”, aunque el `.gitignore` lo excluye; es deuda documental menor, no hallazgo criptográfico.
