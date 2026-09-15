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

| ID | Severidad provisional | Descripción | Estado | Recomendación |
|---|---|---|---|---|
| H-4 | Media | Linaje alto forjado por quien conoce `S` puede secuestrar la sesión | Abierto, declarado | Rediseñar adopción de linaje con autenticación adicional |
| H-5 | Baja–media | KCI: autenticación al nivel de `S`, no de identidad concreta | Abierto, no prometido | Binding a identidad/firma; decidir impacto en negación |
| W-6 | Media disponibilidad | Reloj atrasado tras pérdida de estado rompe un sentido | Abierto, documentado | Linaje monotónico durable o protocolo de reenganche |
| W-14 | Baja | Deduplicación de invites solo en RAM; reinicio permite un timbre adicional | Aceptado/documentado | Persistir ventana `(contacto, callId)` si el modelo lo exige |
| W-1/W-2 | Baja–media | Época 0 re-derivable y replay fuera de ventana de deduplicación | Abierto | Reducir dependencia de época 0 o ampliar autenticación/retención |
| W-10 | Informativa | Etiquetas/rendezvous derivables indefinidamente desde `S` | Abierto | Documentar límites de privacidad frente a compromiso de identidad |
| W-12 | Informativa | Misma semilla Ed25519 convertida para X25519 y firma | Abierto | Separar claves mediante KDF/identidades distintas |
| W-13 | Informativa | Relay, wake y tiempos exponen metadatos | Abierto | Medidas de minimización/batching; fuera del alcance actual |

## Observaciones no elevadas

- **C-001** (timestamp futuro) quedó absorbido por H-7/W-14: el límite superior de 10 minutos está implementado; la raíz era la ausencia de deduplicación por `callId`.
- **W-8** no se eleva frente al relay relayed: las pruebas Go muestran que Noise/TLS rechaza duplicación, reordenamiento y reflexión dentro del circuito. La ausencia de contador de aplicación sigue siendo una limitación frente a endpoints comprometidos.
- La procedencia histórica del AAR de `revision-externa-1` permanece separada de la reproducibilidad posterior `8d028754… → AAR`.

## Criterio de cierre

Un hallazgo abierto solo se cerrará con una corrección de diseño/código y un retest independiente. Las limitaciones aceptadas deben permanecer visibles en el informe publicable y no convertirse en afirmaciones de seguridad positiva.
