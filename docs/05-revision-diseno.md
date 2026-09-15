# Revisión del diseño criptográfico

**Objeto principal:** `a97cbabd95e3787deca872e2157a64473073af6c`  
**Corrección contrastada:** `fe21111f5dfe4d9ee80e8b8e5972152ee57b53b2`.

## Conclusión ejecutiva

Las primitivas usadas son estándar (X25519, HKDF-SHA-256, HMAC-SHA-256 y AES-256-GCM), pero varias propiedades dependen de reglas de estado y de la identidad estática `S`. La revisión posterior corrigió los problemas operativos H-0/H-1/H-2/H-3/H-6 y H-7. Permanecen limitaciones de diseño H-4/H-5, W-1/W-2/W-6/W-10/W-12/W-13, que no deben presentarse como propiedades demostradas.

## Hallazgos de diseño

### H-4 — secuestro por linaje (media, abierto)

La adopción automática de cualquier linaje estrictamente mayor permite a quien conoce `S` fabricar un sobre de época 0 con un linaje artificialmente alto. El receptor puede abandonar su estado legítimo y quedar sin una ruta automática de retorno. La propiedad de recuperación tras pérdida de estado y la resistencia a un contacto malicioso están en tensión; no existe autenticación adicional del linaje ni confirmación externa.

### H-5 — autenticación basada solo en `S` (baja–media, abierto)

El protocolo autentica al par que conoce el secreto estático, no a la identidad concreta que originó cada sobre. Un poseedor de cualquiera de las identidades puede inyectar mensajes v1, época 0 o un linaje nuevo. Esto es una limitación KCI y afecta también buzón y rendezvous, cuyas etiquetas son recalculables desde `S`.

### W-1/W-2/W-6 — época 0, replay y linajes dependientes del reloj

La época 0 es derivable de `S`; la protección contra replay depende de deduplicación y de su ventana de retención. La monotonicidad de `L` depende del reloj local y no de un contador durable por identidad. Estas condiciones afectan unicidad de claves, recuperación y mensajes reentregados, aunque los tests actuales cubran los límites normales.

### W-8 — frames de llamada

No hay contador de aplicación en audio/vídeo. La prueba del commit posterior demuestra que Noise/TLS del circuito relayed rechaza duplicación, orden alterado y reflexión antes de entregar datos. Esto acota el riesgo frente al relay, pero no añade frescura criptográfica contra endpoints comprometidos o APIs que reinyecten frames después del transporte.

## Recomendaciones priorizadas

1. Diseñar una transición de linaje autenticada y no controlable por quien solo conoce `S` (H-4), preservando recuperación tras pérdida de estado.
2. Decidir explícitamente si se promete resistencia KCI; si sí, añadir binding a identidad/firma y revisar negación plausible (H-5).
3. Hacer durable la monotonicidad de linajes o documentar el límite de reinstalación/reloj (W-6).
4. Mantener la deduplicación de señales de llamada persistente si el modelo incluye reinicios; la corrección H-7 actual es solo en RAM.
5. Mantener W-8 como propiedad de transporte verificada, no como sustituto de un contador de frame si en el futuro cambia la topología.

## Estado de verificación

La matriz detallada de derivaciones y pruebas está en `evidence/especificacion-normalizada-fase1-2026-09-14.md`; los retests del commit posterior están en `evidence/revision-dinamica-fe21111-2026-09-15.md`. Este documento es una revisión de diseño, no un análisis formal de seguridad.
