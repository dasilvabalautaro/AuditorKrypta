# Fase 1: especificación criptográfica normalizada

**Objeto:** commit congelado de Krypta `a97cbabd95e3787deca872e2157a64473073af6c`  
**Método:** lectura independiente de la especificación y del código extraído en `.audit-work/krypta-a97cbabd`.

## 1. Cadena de derivaciones

| Uso | IKM | Salt | `info` | Salida |
|---|---|---|---|---:|
| Mensajería v1 | `S` | vacío | `krypta-msg-key-v1` | 32 B |
| Raíz época 0 | `S` | vacío | `krypta-rtc-root:0:<L>` | 32 B |
| Raíz época ≥1 | `X25519(priv_e,pub_e)` | `RK(e−1)` | `krypta-rtc-root:<e>` | 32 B |
| Cadena | `RK(e)` | vacío | `krypta-rtc-chain:<e>:<dir>` | 32 B |
| Clave de mensaje | `CK_N` | — | HMAC con `0x01`; siguiente con `0x02` | 32 B |
| Clave + nonce | `mk_N` | vacío | `krypta-rtc-msg` | 44 B |
| Llamada v2 | `k_llamante ‖ k_contestador` | `S` | `krypta-call-key-v2:<callId>` | 32 B |
| Llamada v1 | `S` | `krypta-call-v1` | `callId` | 32 B |
| Buzón | `S` | vacío | `krypta-mbx:<semana>:<dir>` | 32 B |
| Rendezvous | `S` | vacío | `krypta-rdv:<fecha UTC>` | 32 B |

La implementación Kotlin de HKDF usa HMAC-SHA-256 y sustituye salt vacío por 32 ceros, conforme a RFC 5869 (`p2p-signaling/.../Hkdf.kt`). Las etiquetas de dominio son distintas por inspección; la separación no está formalmente modelada.

## 2. Estados y transiciones

- La época 0 se deriva de `S` y del linaje `L`; no ofrece secreto hacia adelante.
- En épocas posteriores, cada cabecera lleva `cur_pub` y `next_pub`. El receptor solo muta estado después de validar AES-GCM con la cabecera como AAD (`Ratchet.kt:92-121`).
- Un linaje estrictamente mayor se adopta; uno menor se abre como mensaje antiguo pero no se adopta (`Ratchet.kt:97-115`, `247-260`).
- Se conservan como máximo 3 cadenas retiradas y 2000 claves omitidas; cada hueco individual está limitado a 1000.
- La persistencia y deduplicación se realizan fuera de la función pura del ratchet, en `RatchetSessions`/Room.

## 3. Capacidades

La versión anunciada es 3, pero los umbrales son independientes: ratchet, llamada negociada y buzón desde 2; padding desde 3. El contacto solo puede subir de versión. La recepción intenta v2 y, si falla, v1. Esta compatibilidad mantiene una vía estática deliberada y debe evaluarse junto con KCI/downgrade, no como propiedad aislada.

## 4. Llamadas

`CallService` genera mitades de 32 bytes con `SecureRandom` y calcula la clave negociada con orden llamante–contestador (`CallService.kt:247-265`, `573-585`). El canal de medios cifra cada frame con la misma `MessageCipher` y valida un `HELLO:<callId>` o `VHELLO:<callId>`. Los frames no llevan contador; replay y reordenamiento dentro del stream quedan como candidato de revisión W-8.

Observación de robustez: el filtro de antigüedad solo rechaza `now - ts > 45 s` (`CallService.kt:247-253`). Un `ts` futuro se considera fresco; un contacto que ya posee `S` puede repetir un invite con timestamp futuro y provocar timbrado/reintentos. Se registra como candidato **C-001 (disponibilidad/replay de señalización)**, no como hallazgo confirmado: falta medir el impacto de estado y deduplicación en el ciclo completo.

## 5. Buzón ciego

`MailboxLabel` usa semana UTC, dos etiquetas (semana actual y anterior), dirección canónica por comparación lexicográfica de PeerID y salida HKDF de 32 bytes (`MailboxLabel.kt:31-80`). La etiqueta es una credencial al portador: protege la atribución en disco frente a un volcado, pero no frente a correlación en vivo ni frente a quien obtenga una identidad y recalcule `S`.

## 6. Matriz inicial de trazabilidad

| Propiedad | Especificación | Código | Estado de fase 1 |
|---|---|---|---|
| Integridad de cabecera | AEAD con AAD `0..85` | `Ratchet.kt:92-121` | coherente |
| Unicidad clave/nonce | una `mk_N` por `(L,e,N)` | `Ratchet.kt:67-83`; sesiones | condicionada a monotonicidad de `L` y persistencia |
| PFS época ≥1 | borrar privada consumida | `Ratchet.kt:130-155` | afirmada; falta modelo de compromiso |
| Anti-replay época 0 | huella persistida con ventana | `RatchetSessions`, Room | ventana finita; W-2 conocido |
| Clave de llamada v2 | dos mitades + `S` | `CallService.kt:573-585` | coherente; frames sin contador |
| Privacidad de buzón en disco | etiqueta sin `from/to` | `MailboxLabel`, nodo Go | coherente para volcado; metadatos en vivo visibles |

Esta matriz es una reconstrucción de protocolo, no una conclusión de seguridad. Los candidatos pasan a la fase 2/3 para verificación dinámica y revisión de variantes.
