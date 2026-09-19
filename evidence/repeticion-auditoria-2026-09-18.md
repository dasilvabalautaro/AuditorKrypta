# Repetición de auditoría — 18 de septiembre de 2026

**Objeto:** copia aislada de Krypta `a97cbabd95e3787deca872e2157a64473073af6c`  
**Repositorio de trabajo:** `AuditorKrypta`  
**Alcance adicional:** ejecución formal focalizada con ProVerif 2.05

## Resultados dinámicos

Las suites se ejecutaron sobre `.audit-work/krypta-a97cbabd`, con las cachés confinadas bajo `.audit-work/cache/` y sin modificar el repositorio original de Krypta.

| Componente | Comando | Resultado |
|---|---|---|
| Puente Go | `go test -count=1 ./...` | PASA |
| Nodo Go | `go test -count=1 ./...` | PASA |
| Puente Go | `go test -race -count=1 ./...` | PASA |
| Nodo Go | `go test -race -count=1 ./...` | PASA |
| Kotlin/Android | `./gradlew --no-daemon --console=plain testDebugUnitTest` | `BUILD SUCCESSFUL`; 279 pruebas, 0 fallos |

Las pruebas Go necesitaron permiso de loopback para crear hosts libp2p locales. El primer intento dentro del sandbox falló únicamente con `bind: operation not permitted`; la repetición con ese permiso pasó.

## Resultados formales

Entorno OPAM: ProVerif 2.05.

- `models/ratchet-secrecy.pv`: `not attacker(message)` es verdadero y la correspondencia de apertura con envío es verdadera.
- `models/ratchet-kci.pv`: la correspondencia de aceptación como par legítimo es falsa; ProVerif genera una traza donde el atacante conoce `S`, fabrica el sobre y el receptor lo acepta.

El resultado positivo está limitado al modelo simbólico mínimo: HKDF, X25519 y AEAD se tratan como funciones ideales. No cubre persistencia, deduplicación, concurrencia, SQLCipher, gomobile, transporte ni errores de implementación. El resultado negativo confirma formalmente H-5/KCI bajo el modelo actual; no constituye una afirmación de que todo compromiso de identidad rompa todas las épocas del ratchet.

## Conclusión de la repetición

No se observaron regresiones en la línea base dinámica. H-4, H-5 y W-6 permanecen abiertos según el registro; H-5 queda además respaldado por la traza formal de ProVerif. Tamarin no se ejecutó en esta repetición.
