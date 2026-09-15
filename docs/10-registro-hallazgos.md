# Registro de hallazgos y estado de retest

**Auditoría:** AK-2026-001  
**Objeto:** Krypta `a97cbabd…`; correcciones posteriores verificadas hasta `ebf2d43…`.

## Hallazgos corregidos

| ID | Severidad | Descripción | Estado | Evidencia |
|---|---|---|---|---|
| H-0 | Crítica | Carrera por conversación reutilizaba clave/nonce AES-GCM | Corregido | `RatchetSessionsTest`; línea base |
| H-1 | Alta | Pérdida permanente tras borrar/reimportar contacto | Corregido | `ChatServiceTest`; retest completo |
| H-2/H-3 | Media | Copias antiguas o `V` menor degradaban `peerProtocol` | Corregido | `ContactUpsertSqlTest`, `ChatServiceTest` |
| H-6 | Baja | Reañadir contacto bloqueado lo desbloqueaba | Corregido | suite JVM |
| H-7 | Baja | Replay v1 de invite repetía timbres/avisos | Corregido parcialmente | `CallServiceTest`; queda W-14 tras reinicio |

## Limitaciones abiertas

| ID | Severidad provisional | Descripción | Estado | Evidencia | Recomendación |
|---|---|---|---|---|---|
| H-4 | Media | Linaje alto forjado por quien conoce `S` puede secuestrar la sesión | Abierto, declarado | `RatchetTest`: linaje forjado secuestra la sesión y no hay vuelta atrás | Rediseñar adopción de linaje con autenticación adicional |
| H-5 | Baja–media | KCI: autenticación al nivel de `S`, no de identidad concreta | Abierto, no prometido | `ChatServiceTest`: identidad robada permite escribir como cualquier contacto por buzón ciego | Binding a identidad/firma; decidir impacto en negación |
| W-6 | Media disponibilidad | Reloj atrasado tras pérdida de estado rompe un sentido | Abierto, documentado | `RatchetTest`: tras perder estado con reloj atrasado un sentido queda roto | Linaje monotónico durable o protocolo de reenganche |
| W-14 | Baja | Deduplicación de invites solo en RAM; reinicio permite un timbre adicional | Aceptado/documentado | `CallServiceTest` cubre deduplicación en proceso; el reinicio queda como límite | Persistir ventana `(contacto, callId)` si el modelo lo exige |
| W-7 | Media (largo plazo) | Sin protección post-cuántica: todo es X25519 y el PeerID es la clave pública | Abierto; diseño PQ sin implementar a propósito | `DISENO-postcuantico.md`; no hay protección híbrida implementada | Ratchet PQ lento híbrido tras la revisión externa |
| W-11 | Media | El `.krbk` solo lo protege la frase (PBKDF2, 310 000 iteraciones); permite probar frases offline y conduce a H-5 | Abierto, documentado | `IdentityBackup` y especificación; no hay factor adicional | KDF resistente a memoria o factor adicional |
| W-9 | Informativa | `PN` se transmite y no se usa | Abierto | Especificación del protocolo §8; no hay comprobación efectiva | Aclarar si falta una comprobación o retirar el campo |
| W-1/W-2 | Baja–media | Época 0 re-derivable y replay fuera de ventana de deduplicación | Abierto | Especificación y pruebas de ratchet | Reducir dependencia de época 0 o ampliar autenticación/retención |
| W-10 | Informativa | Etiquetas/rendezvous derivables indefinidamente desde `S` | Abierto | Especificación del buzón ciego | Documentar límites de privacidad frente a compromiso de identidad |
| W-12 | Informativa | Misma semilla Ed25519 convertida para X25519 y firma | Abierto | Especificación/derivaciones; cambio afecta formato de red y secretos compartidos | Separar claves mediante KDF/identidades distintas |
| W-13 | Informativa | El nodo ve el grafo diario de parejas en la DHT, la presencia (wake), quién habla con quién en vivo (relay) y, por `identify`, qué protocolos admite cada teléfono | Abierto | Documentación y pruebas de señalización/metadatos | Medidas de minimización/batching; fuera del alcance actual |

## Observaciones no elevadas

- **C-001** (timestamp futuro) quedó absorbido por H-7/W-14: el límite superior de 10 minutos está implementado; la raíz era la ausencia de deduplicación por `callId`.
- **W-8** se acota al transporte verificado: dos pruebas Go muestran que un intermediario que duplica, reordena o refleja bytes provoca `tls: bad record MAC` y la caída de la conexión sin entregar datos repetidos; otra prueba muestra que un relay con TLS espía no ve el contenido del circuito. Se negoció TLS 1.3. La vía QUIC directa no tiene test específico; no hay contador de aplicación frente a extremos comprometidos.
- La procedencia histórica del AAR de `revision-externa-1` permanece separada de la reproducibilidad posterior `8d028754… → AAR`.
- Las recomendaciones H-4/H-5/W-6/W-12 tienen coste de compatibilidad: pueden cambiar el formato de red o la regla de linaje. W-6 tampoco resuelve por sí solo importar un `.krbk` en un móvil nuevo salvo que el último linaje viaje en el respaldo. Estos elementos están congelados hasta el informe externo de Krypta.
- El commit `ebf2d43` eliminó tres bytes NUL literales de `CallService.kt` y `ChatService.kt`; no cambió el comportamiento criptográfico, pero sí la auditabilidad de las fuentes.

## Criterio de cierre

Un hallazgo abierto solo se cerrará con una corrección de diseño/código y un retest independiente. Las limitaciones aceptadas deben permanecer visibles en el informe publicable y no convertirse en afirmaciones de seguridad positiva.
