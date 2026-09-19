# Retest de actualización — HEAD `58b2286` — 19 de septiembre de 2026

**Objeto:** Krypta `58b228639ee456a122627119fb8d7188f73504c0`

**Checkout de ejecución:** copia temporal aislada; el repositorio original de Krypta permaneció limpio.

## Procedencia del AAR

Se ejecutó `native-bridge/libp2p/build-aar.sh` desde el HEAD actual. El script terminó correctamente y comprobó para las cuatro ABI:

- páginas de memoria de 16 KB;
- commit `58b228639ee456a122627119fb8d7188f73504c0` embebido;
- ausencia de rutas locales.

SHA-256 del AAR generado:

```text
ce1a3e515ac613e45256c53cbf77b5191323dbcde4cb752b08d051ca7310711d
```

## Resultados dinámicos

| Componente | Comando | Resultado |
|---|---|---|
| Kotlin/Android | `./gradlew --no-daemon --console=plain :p2p-signaling:testDebugUnitTest` | **PASA**, 218 tests, 0 omitidos, 0 fallos, 0 errores |
| Kotlin dirigido | `CallServiceTest`, `ChatServiceTest`, `RatchetTest` | **PASA** |
| Puente Go | `go test -count=1 ./...` | **PASA** |
| Nodo Go | `go test -count=1 ./...` | **PASA** |
| Puente Go | `go test -race -count=1 ./...` | **PASA** |
| Nodo Go | `go test -race -count=1 ./...` | **PASA** |

El checkout fuente original siguió limpio después de la revisión. No había ningún dispositivo listado por `adb devices`, por lo que no se ejecutaron pruebas instrumentadas Android ni validación en hardware.

## Resultados formales

Se volvieron a ejecutar los modelos ProVerif 2.05 existentes:

- `models/ratchet-secrecy.pv`: `not attacker(message)` es verdadero y la correspondencia `opened(message) ==> sent(message)` es verdadera, bajo el modelo simbólico idealizado.
- `models/ratchet-kci.pv`: la correspondencia de aceptación como par legítimo es falsa y se reproduce la traza KCI cuando el atacante conoce `S`.

## Evaluación

No se observaron regresiones en el HEAD `58b2286`. Los hallazgos y limitaciones ya registrados permanecen vigentes: H-4, H-5, W-6 y W-14; W-8 continúa condicionado a la seguridad de transporte de libp2p. Las actualizaciones documentales sobre wake v2, metadatos y operador único hacen más preciso el modelo de amenazas, pero no corrigen esos límites criptográficos.

Este retest no sustituye una futura auditoría completa si se modifica el ratchet, la derivación de claves, los formatos, la persistencia del estado o el transporte de llamadas.
