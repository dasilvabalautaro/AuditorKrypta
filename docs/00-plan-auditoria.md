# Plan de auditoría criptográfica de Krypta

**Estado:** aprobado como plan inicial; auditoría aún no ejecutada

**Fecha de inicio:** 14 de septiembre de 2026

**Repositorio auditado:** `/Users/davidsilva/AndroidStudioProjects/Krypta`

**Repositorio de trabajo:** `/Users/davidsilva/VisualStudioCodeProjects/AuditorKrypta`

**Commit objetivo:** `a97cbabd95e3787deca872e2157a64473073af6c`

**Asunto del commit:** `fix(cripto): revisión del protocolo — clave y nonce repetidos, mensajes perdidos y versión que bajaba`

## 1. Objetivo

Realizar una revisión independiente del diseño criptográfico del protocolo de sesión de Krypta y del camino criptográfico implementado en Kotlin y Go. La revisión cubrirá el ratchet por épocas, la negociación de capacidades, las claves de llamada y las etiquetas del buzón ciego.

El resultado principal será un informe apto para publicación que distinga con precisión:

- lo afirmado por el diseño y la documentación;
- lo que implementa el código;
- lo que demuestran las pruebas;
- lo que no se pudo verificar o queda fuera del alcance.

La auditoría no emitirá afirmaciones absolutas como «Krypta es seguro». Sus conclusiones estarán limitadas al commit, componentes, adversarios, propiedades y métodos expresamente documentados.

## 2. Principios de trabajo

1. **Objeto inmutable.** Las conclusiones iniciales se referirán al commit objetivo. Los cambios posteriores se evaluarán como correcciones separadas.
2. **Independencia.** La documentación y las revisiones previas del autor son entradas, no evidencia suficiente de corrección.
3. **Trazabilidad.** Cada propiedad se vinculará con especificación, código, pruebas y conclusión.
4. **Reproducibilidad.** Todo hallazgo tendrá evidencia y, cuando sea viable, una prueba automatizada o procedimiento mínimo de reproducción.
5. **Divulgación responsable.** Los detalles que faciliten explotación de problemas sin corregir se mantendrán en un anexo privado hasta acordar su publicación.
6. **Preservación.** La auditoría no modificará el repositorio Krypta salvo autorización expresa. Notas, modelos, resultados y pruebas externas vivirán en `AuditorKrypta`.

## 3. Alcance

### 3.1 Alcance principal

- Transporte legado y transporte con ratchet.
- Ratchet doble por épocas, estado, linajes, claves efímeras, cadenas simétricas, contadores y mensajes omitidos.
- Formatos v1, v2 y v3, cabeceras, AAD y relleno por tramos.
- Negociación de capacidades y resistencia a downgrade, replay y rollback.
- Derivación, autenticación y ciclo de vida de las claves de llamada de voz y vídeo.
- Derivación, rotación y uso de etiquetas de buzón y wake.
- Frontera Kotlin–Go/gomobile y correspondencia del AAR distribuido con sus fuentes.
- Persistencia, exclusión mutua, atomicidad, deduplicación, reintentos y recuperación de sesión.
- Uso de X25519, HKDF, HMAC, AES-GCM, aleatoriedad y separación de dominios en el camino anterior.

### 3.2 Superficies adyacentes

Se revisarán solo en la medida en que afecten al alcance principal:

- identidad Ed25519/X25519 y derivación del secreto estático;
- Android Keystore, SQLCipher y almacenamiento del estado del ratchet;
- copia y restauración de identidad;
- framing y transporte libp2p;
- KEM poscuántico;
- adjuntos cifrados y recibos asociados al mismo camino de sesión.

Una revisión completa de seguridad de Android, interfaz, infraestructura, cadena de suministro o libp2p no forma parte de este encargo.

## 4. Fuentes iniciales

### 4.1 Documentación

- `docs/ESPECIFICACION-protocolo.md`
- `docs/DISENO-ratchet.md`
- `docs/DISENO-buzon-ciego.md`
- `docs/REVISION-protocolo-2026-09-14.md`
- `docs/security-model.md`
- `SECURITY.md`
- `README.md`
- `CLAUDE.md`

### 4.2 Kotlin

- `p2p-signaling/.../Ratchet.kt`
- `p2p-signaling/.../RatchetState.kt`
- `p2p-signaling/.../RatchetSessions.kt`
- `p2p-signaling/.../ChatService.kt`
- `p2p-signaling/.../MessageEnvelope.kt`
- `p2p-signaling/.../CallService.kt`
- `p2p-signaling/.../MailboxLabel.kt`
- `p2p-signaling/.../AesGcmMessageCipher.kt`
- `native-bridge/.../BridgeCurve25519.kt`
- `native-bridge/.../IdentityStore.kt`
- `native-bridge/.../KeystoreKeyWrapper.kt`
- `data/.../RoomRatchetStore.kt`
- `data/.../RatchetDao.kt`

Las rutas abreviadas se resolverán y registrarán de forma exacta en el inventario de código.

### 4.3 Go

- `native-bridge/libp2p/bridge.go`
- `native-bridge/libp2p/kem.go`
- `infra/node/mailbox.go`
- `infra/node/wake.go`
- framing y streams relacionados con mensajería y llamadas.

### 4.4 Pruebas existentes

- Pruebas `Ratchet*`, `RatchetSessions*`, `ChatService*`, `CallService*`, `MailboxLabel*` y `MessageEnvelope*`.
- Pruebas de claves del ratchet y buzón en `native-bridge/libp2p`.
- Pruebas de buzón y wake en `infra/node`.
- Revisión independiente de los hallazgos internos H-0 a H-6 y de sus regresiones.

## 5. Modelo de amenazas

Se evaluarán, como mínimo, los siguientes adversarios y eventos:

- observador pasivo de la red;
- atacante activo capaz de interceptar, modificar, bloquear, retrasar, duplicar y reordenar;
- nodo de buzón o wake curioso o malicioso;
- contacto malicioso;
- robo temporal o permanente del secreto estático `S`;
- compromiso del estado actual del ratchet;
- restauración de una base de datos, contacto o identidad antiguos;
- ejecución simultánea en hilos, coroutines o procesos;
- cliente antiguo, defectuoso o deliberadamente degradado;
- pérdida, repetición o cruce simultáneo de mensajes de cambio de época.

Para cada propiedad se declarará qué adversarios cubre y qué supuestos requiere.

## 6. Propiedades que se comprobarán

- Confidencialidad e integridad del contenido.
- Autenticación del par, de las claves efímeras y del linaje.
- Forward secrecy y recuperación poscompromiso.
- Unicidad de cada par clave/nonce.
- Separación de dominios entre mensajes, épocas, llamadas, buzón y versiones.
- Resistencia a replay, rollback, downgrade, reflection y unknown-key-share.
- Entrega segura ante duplicados, fallos de envío y caídas entre cómputo y persistencia.
- Borrado y no reutilización de claves consumidas.
- Interoperabilidad segura entre versiones.
- Privacidad de las etiquetas del buzón y límites reales frente a correlación y análisis de tráfico.
- Disponibilidad dentro de límites razonables: saltos, cuotas, entradas grandes y agotamiento de recursos.

## 7. Fases de ejecución

### Fase 0 — Congelación y reproducibilidad

- Confirmar el commit objetivo y el estado del árbol.
- Registrar versiones de JDK, Kotlin, Gradle, Android, Go y dependencias relevantes.
- Inventariar binarios generados y comprobar la procedencia del AAR.
- Definir comandos reproducibles y conservar sus salidas.

**Salida:** manifiesto de auditoría y línea base del entorno.

### Fase 1 — Reconstrucción independiente de la especificación

- Extraer formatos, estados, transiciones y reglas de error.
- Catalogar cada HKDF: IKM, salt, info, longitud y propósito.
- Catalogar claves, nonces, etiquetas, AAD y material persistido.
- Comparar especificación, documentos de diseño y comportamiento del código.
- Abrir incidencias de documentación para ambigüedades o contradicciones.

**Salida:** especificación normalizada, diagramas y matriz de trazabilidad inicial.

### Fase 2 — Revisión del diseño criptográfico

#### Ratchet por épocas

- Época 0 derivable de `S` y sus consecuencias.
- Construcción simétrica sin roles fijos.
- Avance con claves cruzadas o simultáneas.
- Linajes rivales, reenganche y selección de época.
- Mensajes perdidos, duplicados, fuera de orden y ráfagas.
- Seguridad de cadenas, claves de mensaje y nonces derivados.
- Cabeceras autenticadas, límites y padding v3.
- Atomicidad entre consumo de clave, persistencia y entrega.

#### Negociación de capacidades

- Autenticidad y frescura de anuncios `V`.
- Monotonicidad y resistencia a downgrade.
- Restauraciones y copias antiguas de contactos.
- Confusión entre versiones y detección heurística de formatos.
- Comportamiento fail-open/fail-closed y compatibilidad v1/v2/v3.

#### Claves de llamada

- Entropía de ambas contribuciones.
- Binding a `callId`, pares, roles, dirección y medio.
- Orden y codificación de `k_caller || k_callee`.
- Replays de invitación/aceptación y llamadas simultáneas.
- Relación entre señalización ratcheted y clave de medios.
- Separación entre voz y vídeo y eliminación al finalizar.
- Consecuencias del compromiso de identidad, señalización o estado.

#### Etiquetas del buzón

- Entropía, derivación, longitud y separación por propósito/nodo.
- Rotación y ventana de solapamiento.
- Enumeración, correlación y seguimiento longitudinal.
- Modelo de autorización basado en conocimiento de la etiqueta.
- Replay, ack malicioso, cuotas, agotamiento y validación de rutas.
- Diferencia entre privacidad prometida y metadatos observables.

**Salida:** análisis de diseño, supuestos y posibles ataques.

### Fase 3 — Revisión del código Kotlin y Go

- Seguir cada secreto desde su creación hasta su destrucción.
- Verificar parámetros y contratos de X25519, HKDF, HMAC y AES-GCM.
- Revisar validación de claves, valores degenerados, longitudes y errores.
- Revisar aleatoriedad, nonces y separación de dominios.
- Revisar serialización, canonicalización, framing y AAD.
- Revisar excepciones que puedan repetir o desincronizar estado.
- Revisar mutexes, transacciones, coroutines y carreras.
- Revisar límites, asignaciones y entradas controladas por red.
- Comparar semántica y tipos a ambos lados de gomobile.
- Verificar que el binario AAR corresponde al código auditado.

**Salida:** notas de revisión por archivo y candidatos a hallazgo.

### Fase 4 — Verificación dinámica

Primero se ejecutará la suite existente sin cambios para establecer una línea base. Después se crearán pruebas de auditoría independientes:

- vectores deterministas de derivación;
- pruebas cruzadas Kotlin–Go;
- property-based testing del ratchet;
- fuzzing de cabeceras, sobres, anuncios y solicitudes de buzón;
- estrés concurrente de cifrado, recepción y reintento;
- caída simulada en cada frontera de persistencia;
- replay, reorder, skip, rollback, epoch jump y linajes rivales;
- matriz de interoperabilidad v1/v2/v3;
- restauración y recreación de contactos/identidades;
- `go test -race` para componentes Go aplicables.

**Salida:** resultados reproducibles, corpus y pruebas de regresión.

### Fase 5 — Análisis formal focalizado

Modelar en Tamarin o ProVerif el núcleo mínimo necesario:

- identidad y secreto estático;
- claves efímeras y transición de épocas;
- cadena simétrica;
- negociación de capacidades;
- compromiso de `S` y del estado;
- reinicio y linajes rivales.

Se documentará explícitamente la correspondencia y las diferencias entre el modelo y la implementación.

**Salida:** modelo, propiedades consultadas, resultados y limitaciones.

### Fase 6 — Hallazgos, correcciones y retest

- Validar independientemente H-0 a H-6.
- Priorizar H-4 y H-5, documentados previamente como no corregidos.
- Buscar variantes y causas sistémicas.
- Comunicar de inmediato problemas críticos por vía privada.
- Revisar parches en commits separados.
- Repetir pruebas y clasificar cada hallazgo como corregido, abierto, aceptado o informativo.

**Salida:** expedientes de hallazgo y registro de retest.

### Fase 7 — Informe publicable

Preparar un informe principal en inglés con resumen ejecutivo en español, salvo decisión posterior distinta. Incluirá:

- alcance, commit y metodología;
- arquitectura y modelo de amenazas;
- evaluación por componente y propiedad;
- tabla de hallazgos y estado de corrección;
- limitaciones y riesgos conocidos;
- recomendaciones priorizadas;
- anexos reproducibles que sea seguro publicar.

Los detalles explotables de problemas abiertos se separarán en un anexo privado.

## 8. Formato de hallazgos

Cada hallazgo contendrá:

- identificador estable;
- título y severidad;
- componente y versiones afectadas;
- propiedad violada;
- modelo y prerrequisitos del atacante;
- impacto técnico y para usuarios;
- evidencia con referencias exactas a código;
- reproducción o prueba automatizada;
- causa raíz;
- recomendación concreta;
- respuesta del mantenedor;
- estado y resultado del retest.

La severidad se asignará con una matriz documentada de impacto por explotabilidad. CVSS y CWE podrán añadirse como referencias, pero no sustituirán el razonamiento específico del protocolo.

## 9. Entregables previstos

1. Manifiesto del entorno y objeto auditado.
2. Diario cronológico de auditoría.
3. Inventario de código, dependencias, primitivas y derivaciones.
4. Modelo de amenazas.
5. Especificación normalizada y diagramas.
6. Matriz propiedad–especificación–código–prueba–conclusión.
7. Notas de revisión de Kotlin y Go.
8. Pruebas, vectores, corpus y resultados reproducibles.
9. Modelo formal y resultados.
10. Expedientes privados de hallazgos.
11. Informe publicable y registro final de retest.

## 10. Estructura documental prevista

```text
AuditorKrypta/
├── docs/
│   ├── 00-plan-auditoria.md
│   ├── 01-manifiesto-y-entorno.md
│   ├── 02-modelo-de-amenazas.md
│   ├── 03-especificacion-normalizada.md
│   ├── 04-inventario-criptografico.md
│   ├── 05-revision-diseno.md
│   ├── 06-revision-codigo-kotlin.md
│   ├── 07-revision-codigo-go.md
│   ├── 08-verificacion-dinamica.md
│   ├── 09-analisis-formal.md
│   ├── 10-registro-hallazgos.md
│   └── diario-auditoria.md
├── findings/
├── models/
├── tests/
├── evidence/
└── report/
```

La estructura se creará progresivamente para evitar archivos vacíos o artefactos que aparenten trabajo todavía no realizado.

## 11. Calendario estimado

- Días 1–2: congelación, entorno, especificación y amenazas.
- Días 3–6: revisión del diseño.
- Días 5–10: revisión Kotlin/Go y persistencia/concurrencia.
- Días 8–12: pruebas adversariales, fuzzing e interoperabilidad.
- Días 10–14: análisis formal focalizado.
- Días 15–17: consolidación, comunicación y borrador.
- Después de correcciones: retest e informe final.

Estimación inicial: **15–17 días de auditoría**, más el periodo de corrección y retest. La estimación se revisará al terminar la especificación normalizada.

## 12. Criterios de finalización

La auditoría estará terminada cuando:

- todo componente del alcance tenga conclusión documentada;
- cada propiedad tenga evidencia o una limitación explícita;
- las discrepancias entre diseño y código estén resueltas o registradas;
- los hallazgos tengan severidad, reproducción y recomendación;
- los parches incluidos se hayan vuelto a probar;
- el informe pueda reproducirse desde el commit y los comandos registrados;
- el texto público no revele detalles peligrosos de vulnerabilidades abiertas.

## 13. Registro inicial de evidencia

Durante la preparación del plan se verificó:

- el repositorio Krypta estaba en la rama `main` sin cambios visibles;
- el commit HEAD era `a97cbabd95e3787deca872e2157a64473073af6c`;
- existen especificaciones y documentos separados para ratchet, buzón y protocolo;
- existen implementaciones y pruebas relevantes en Kotlin y Go;
- existe una revisión interna fechada el 14 de septiembre de 2026 con hallazgos H-0 a H-6;
- `AuditorKrypta` no era todavía un repositorio Git y no contenía una estructura documental previa visible.

Estas comprobaciones solo delimitan el trabajo. No constituyen resultados de seguridad.
