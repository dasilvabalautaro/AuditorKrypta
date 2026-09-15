# Krypta — revisión criptográfica independiente

**Identificador:** AK-2026-001  
**Fecha de corte:** 15 de septiembre de 2026  
**Objeto congelado para la evaluación:** commit `a97cbabd...` (revisión externa inicial)  
**Correcciones posteriores contrastadas:** `fe21111f`, `f119237e`, `f8d9a75d` y `ebf2d43`.

## Resumen ejecutivo

Esta revisión cubre el protocolo de sesión de Krypta —ratchet por épocas, negociación de capacidades, claves de llamada y etiquetas del buzón ciego— y el camino criptográfico Kotlin/Go. La implementación usa primitivas estándar (X25519, HKDF y AES-GCM) y cuenta con controles útiles de autenticación de cabeceras, deduplicación y avance de estado.

La revisión identificó y contrastó correcciones operativas para H-0–H-3, H-6 y H-7. Permanecen abiertas limitaciones de diseño y disponibilidad: H-4 (linaje forjado), H-5 (compromiso de la clave estática del receptor), W-6 (pérdida de estado con reloj atrasado), W-7 (ausencia de protección post-cuántica), W-9 (`PN` no usado), W-11 (respaldo protegido solo por frase), W-14 (volatilidad del estado de invites) y W-13 (metadatos del grafo de parejas, presencia, relay e `identify`). W-8 queda acotado al transporte relay verificado: el relay no puede leer ni fabricar contenido protegido por Noise, aunque sí puede afectar entrega y disponibilidad. La observación C-001 se formula como reproducción de señales de llamada por el camino v1 sin deduplicación por `callId`, cuyo impacto concreto es el spam de “llamada perdida”; la fecha futura y una ventana simétrica no constituyen por sí solas el arreglo.

No se emite una certificación formal ni una afirmación general de forward secrecy, post-compromise security o resistencia KCI. El análisis formal es un modelo abstracto no ejecutado en Tamarin/ProVerif.

## 1. Alcance y metodología

Se inspeccionaron fuentes Kotlin y Go, documentación del protocolo, pruebas unitarias, artefactos AAR/APK disponibles y el historial Git visible. Las pruebas dinámicas se ejecutaron en clones aislados, sin modificar el árbol ni el repositorio de Krypta. Se repitieron suites Gradle y Go, pruebas dirigidas para H-4/H-5/W-6 y controles de integridad de fuentes.

La evidencia primaria y las limitaciones de reproducibilidad se detallan en el [manifiesto y entorno](../docs/01-manifiesto-y-entorno.md), la [revisión de código Kotlin](../docs/06-revision-codigo-kotlin.md), la [revisión de código Go](../docs/07-revision-codigo-go.md) y el [análisis formal focalizado](../docs/09-analisis-formal.md).

## 2. Arquitectura evaluada

- El ratchet deriva claves por épocas y autentica los mensajes con AEAD; el estado local determina el avance y el descarte de claves.
- La negociación de capacidades y las cabeceras autenticadas vinculan el contexto criptográfico con la sesión, pero no convierten automáticamente una clave estática comprometida en una garantía KCI.
- Las llamadas derivan material específico para el contexto de llamada. Las señales de control deben tratarse como eventos identificables y deduplicables, no como mensajes independientes sin `callId`.
- Las etiquetas del buzón ciego permiten direccionar sin exponer directamente el identificador lógico; el relay sigue siendo un adversario de disponibilidad y metadatos.

## 3. Resultados

### Propiedades corregidas o verificadas

H-0–H-3 y H-6 cuentan con correcciones observables en el historial y regresión automatizada; para H-6, `ChatServiceTest` verifica que volver a añadir un contacto bloqueado no lo desbloquea ni olvida su versión. H-7 evita aceptar invites fuera de la política temporal documentada. La prueba de H-5 se reforzó con X25519 reales y confirma que el compromiso de la clave estática del receptor permite reconstruir el secreto de sesión bajo el modelo actual.

### Limitaciones abiertas

| ID | Severidad | Estado | Consecuencia |
|---|---|---|---|
| H-4 | Alta | Abierto | Un linaje o secreto de época forjado puede ser aceptado si el atacante satisface las entradas previstas por el diseño. |
| H-5 | Alta | Abierto | El compromiso de la estática receptora permite reconstruir material de sesión; no debe prometerse resistencia KCI/PFS adicional. |
| W-6 | Media | Abierto | La pérdida o restauración atrasada del estado puede impedir convergencia y causar indisponibilidad. |
| W-14 | Media | Abierto | El estado de invites es volátil; reinicios pueden reabrir condiciones de aceptación o producir inconsistencias. |
| W-7 | Media (largo plazo) | Abierto | Todo el intercambio actual usa X25519; el diseño híbrido PQ está documentado, pero no implementado. |
| W-11 | Media | Abierto | El `.krbk` usa PBKDF2-HMAC-SHA256 con 310 000 iteraciones y no tiene factor adicional; permite ataques offline de frase. |
| W-9 | Informativa | Abierto | `PN` se transmite, pero el receptor no lo usa como comprobación efectiva. |
| W-13 | Informativa | Abierto | El nodo observa el grafo diario de parejas en la DHT, la presencia, las parejas activas del relay y los protocolos anunciados por `identify`. |

W-1/W-2/W-10/W-12 permanecen como recomendaciones de robustez y documentación en el registro consolidado; W-12 y las correcciones de H-4/H-5/W-6 pueden cambiar el formato de red o la regla de linaje. C-001 no se trata como hallazgo independiente: queda absorbido por H-7/W-14, con el impacto concreto de spam de “llamada perdida”. No se presentan como vulnerabilidades explotables demostradas sin el modelo de amenaza correspondiente.

## 4. Verificación dinámica

El retest del estado `f8d9a75` completó la suite Gradle sin fallos y la suite Go con `-race` sin fallos. El retest de `ebf2d43` volvió a ejecutar `RatchetTest` y `ChatServiceTest` con resultado `BUILD SUCCESSFUL`; el escaneo de fuentes Kotlin/Go/Markdown no encontró bytes NUL. Las pruebas dirigidas de H-4, H-5 y W-6 reproducen las propiedades descritas arriba. La evidencia reproducible está en [retest completo](../evidence/retest-completo-f8d9a75-2026-09-15.md), [retest H-4/H-5/W-6](../evidence/retest-h4-h5-w6-2026-09-15.md) y [retest `ebf2d43`](../evidence/retest-ebf2d43-h5-2026-09-15.md).

## 5. Recomendaciones

1. Documentar explícitamente el modelo de compromiso de claves y retirar cualquier lenguaje que sugiera PFS/KCI no demostrado.
2. Añadir autenticación de linaje y vinculación de identidad que cierre H-4/H-5, con pruebas negativas y vectores de regresión.
3. Persistir o invalidar de forma segura el estado de invites y diseñar recuperación ante rollback para W-6/W-14.
4. Versionar señales de llamada con `callId`, tipo, emisor y estado monotónico; deduplicar antes de generar notificaciones.
5. Mantener pruebas de interoperabilidad y reproducibilidad de AAR/APK separadas de las afirmaciones de seguridad del protocolo.
6. Tratar W-7 y W-11 como riesgos de largo plazo y de recuperación de identidad, no como detalles cosméticos: requieren un ratchet PQ híbrido y una KDF/factor adicional, respectivamente.

## 6. Limitaciones y declaración

La revisión es independiente y de alcance limitado al material disponible en la fecha de corte. No incluye auditoría del sistema operativo, hardware seguro, generación de entropía del dispositivo, infraestructura de despliegue ni revisión criptográfica formal automatizada. Un resultado de prueba indica comportamiento observado, no una prueba matemática de seguridad.

## Referencias

- [Registro de hallazgos](../docs/10-registro-hallazgos.md)
- [Revisión de diseño](../docs/05-revision-diseno.md)
- [Revisión de código Kotlin](../docs/06-revision-codigo-kotlin.md)
- [Revisión de código Go](../docs/07-revision-codigo-go.md)
- [Verificación dinámica](../docs/08-verificacion-dinamica.md)
- [Especificación normalizada](../evidence/especificacion-normalizada-fase1-2026-09-14.md)
