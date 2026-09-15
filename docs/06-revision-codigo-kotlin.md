# Revisión de código Kotlin — camino criptográfico

**Objeto de comparación:** `a97cbabd…` (auditado) y corrección posterior `fe21111f…`.  
**Alcance:** `Ratchet`, `RatchetSessions`, `ChatService`, `CallService`, `MailboxLabel` y `AesGcmMessageCipher`.

## Resultados

- `Ratchet.decrypt` valida la cabecera como AAD antes de devolver el estado nuevo; el rechazo de DH degenerado se hace comprobando secreto todo-cero. La persistencia y el mutex por conversación están en `RatchetSessions`.
- `ChatService` mantiene recepción v1 por compatibilidad. La decisión de enviar por ratchet/buzón depende de `peerProtocol` monotónico y de umbrales separados por capacidad.
- `MailboxLabel` deriva etiquetas semanales de 32 bytes con dirección canónica. La etiqueta protege el grafo en disco, no la correlación viva ni el compromiso de `S`.
- Antes de `fe21111f…`, `CallService.onInvite` no deduplicaba por `callId` y aceptaba cualquier timestamp futuro. La corrección añade `firstSighting` en RAM, una ventana futura de 10 minutos e idempotencia de `recordMissedCall` mediante un ID derivado de `(contacto, callId)`.
- La deduplicación de invites no sobrevive al reinicio y el límite de 256 entradas permite expulsión por volumen; ambas son limitaciones de disponibilidad, no una recuperación de confidencialidad.
- Los frames de medios siguen usando la misma clave de llamada y no llevan contador a nivel de aplicación. Esta propiedad queda mitigada contra el relay por el transporte relayed probado, pero no contra un endpoint comprometido.

## Candidatos y estado

| ID | Observación | Estado |
|---|---|---|
| H-7 | Replay v1 de `invite` sin deduplicación por `callId`, con filas/avisos repetidos | Corregido en `fe21111f…`; retest JVM pasa |
| C-001 | Timestamp futuro sin límite superior | Corregido en `fe21111f…`; retest JVM pasa |
| W-8 | Sin contador de frame | Abierto como propiedad de formato; relay no puede atravesar Noise/TLS relayed en prueba Go |

## Limitaciones de la revisión

No se evaluó persistencia de `seenInvites` tras muerte del proceso ni una matriz completa de streams directos, NAT y relay simultáneos. Las pruebas dinámicas cubren los módulos afectados, no sustituyen una prueba de dos dispositivos.
