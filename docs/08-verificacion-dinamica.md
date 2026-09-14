# Verificación dinámica — lote inicial

**Objeto:** `a97cbabd95e3787deca872e2157a64473073af6c`  
**Fecha:** 14 de septiembre de 2026  
**Ejecución:** copia aislada `.audit-work/krypta-a97cbabd`; las pruebas añadidas fueron temporales y no forman parte de Krypta.

## Replay de frames de llamada (W-8)

Se añadió temporalmente `AuditFrameReplayTest`, que cifra un frame con `AesGcmMessageCipher` y descifra dos veces el mismo wire. La prueba pasó en ambas lecturas. Esto confirma el comportamiento esperado por inspección: cada frame usa AES-GCM con nonce aleatorio, pero no lleva contador ni estado de números usados (`CallService.kt` bombea `cipher.decrypt(key, frame)` directamente en el bucle de recepción).

**Conclusión:** un relay que duplique o reordene bytes de un stream puede provocar que el receptor procese de nuevo un frame válido. La autenticidad e integridad del frame se mantienen; la frescura y el orden no. El impacto exacto en audio/vídeo (repetición audible, frame antiguo o desincronización) requiere prueba de integración con un stream adversarial. Se conserva como W-8 abierto, sin elevarlo todavía a hallazgo independiente.

## Invite con timestamp futuro (C-001)

Se añadió temporalmente a `CallServiceTest` una prueba que entrega un invite E2EE con `ts = ahora + 1 hora`. La prueba pasó: el receptor entró en `CallPhase.RINGING`.

La condición implementada es `System.currentTimeMillis() - sig.ts > 45_000` (`CallService.kt:247-253`). No existe límite superior para timestamps futuros. Un contacto que ya conoce `S` puede reproducir un invite auténtico con fecha adelantada y provocar timbrado inmediato; el `callId` y el estado de llamada limitan llamadas simultáneas, pero no se observó una deduplicación persistente por `callId`.

**Clasificación provisional:** candidato de disponibilidad/replay, severidad por determinar. Recomendación para la fase de diseño: aceptar solo una ventana simétrica (`abs(now - ts) ≤ 45 s`) o introducir una política explícita de tolerancia de reloj y deduplicación de `callId`.

## Integridad del objeto original

Después de las ejecuciones, `git status --porcelain=v1 --untracked-files=all` en Krypta original siguió vacío. Los cambios de prueba existieron únicamente en la copia bajo `.audit-work/`, ignorada por AuditorKrypta.
