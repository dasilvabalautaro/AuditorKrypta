**Asunto:** AK-2026-001 — observaciones al borrador de informe (corte 15 sep 2026)

Gracias por el borrador. Incorpora casi todo lo que pedimos en la respuesta al registro: W-7, W-9 y
W-11 en la tabla, W-13 completo, el test de H-6, el coste de los cambios de formato y la formulación
de C-001. Hemos comprobado los hashes: `a97cbabd…` es el commit de la etiqueta `revision-externa-1`,
y `fe21111f`, `f119237e`, `f8d9a75d` y `ebf2d43` existen. También valoramos que no afirme ni
certificación, ni secreto hacia adelante, ni resistencia KCI.

**Uso del documento.** Krypta tratará este informe como **documento interno de preparación** para una
auditoría pública oficial: no lo publicará ni lo incorporará al repositorio. Aun así, las correcciones
de fondo son necesarias, porque servirá de base para preparar esa auditoría.

## 1. H-5 está mal descrita

El borrador dice que el compromiso de la clave del receptor permite «reconstruir el secreto de sesión»
(§3) o «material de sesión» (tabla). H-5 es **suplantación**, no descifrado. Lo que prueba el test es
que quien obtiene la identidad de la víctima calcula el mismo `S` que cualquiera de sus contactos y
puede escribirle como ese contacto. Con `S` se leen la época 0 y el tráfico v1 (W-1, W-5), pero **no**
las épocas ≥ 1 del ratchet frente a un atacante pasivo. La redacción actual sugiere que robar la
identidad rompe el ratchet.

Proponemos:

- **Fila de la tabla:** «Quien comprometa la identidad del receptor calcula `S` con cualquiera de sus
  contactos y puede escribirle como ese contacto (KCI). Entra por el buzón ciego; por el buzón con
  PeerID, solo si el nodo miente sobre el remitente. Con `S` se leen la época 0 y el tráfico v1, no
  las épocas ≥ 1 frente a un atacante pasivo. No se promete resistencia KCI.»
- **Frase de §3:** «La prueba de H-5 se reforzó con X25519 reales y confirma que, bajo el modelo
  actual, el compromiso de la clave del receptor permite suplantar ante él a cualquiera de sus
  contactos.»

## 2. Las severidades contradicen el registro

| ID | Registro (`10-registro-hallazgos`) | Borrador |
|---|---|---|
| H-4 | Media | Alta |
| H-5 | Baja–media | Alta |
| W-14 | Baja | Media |

El informe no puede contradecir a su anexo sin explicarlo. O se justifica el cambio o se alinea con el
registro.

## 3. W-14 está exagerada

«Reinicios pueden reabrir condiciones de aceptación o producir inconsistencias» no describe lo que
pasa. Proponemos: «Si la app se reinicia en los 10 min 45 s siguientes a un invite, quien lo reenvíe
puede hacerlo sonar **una vez más**. La fila de llamada perdida es persistente y no se repite.
Aceptado y documentado: persistir esa memoria dejaría metadatos de llamadas fuera de la base cifrada.»

## 4. H-7 está mal caracterizada

- **§3** dice que H-7 «evita aceptar invites fuera de la política temporal documentada». El arreglo de
  fondo es atender cada invite **una sola vez por `(contacto, callId)`** y hacer idempotente la fila de
  llamada perdida; el límite de 10 minutos hacia el futuro es complementario. Así lo dice también el
  propio borrador al tratar C-001.
- **§2** dice que las señales «deben tratarse como eventos identificables y deduplicables, no como
  mensajes independientes sin `callId`». Las señales **siempre** llevaron `callId`; lo que faltaba era
  deduplicar por él.

## 5. La consecuencia de H-4 es vaga

«Si el atacante satisface las entradas previstas por el diseño» no dice lo importante. Proponemos:
«Quien tenga `S` (cualquiera de las dos identidades) envía un sobre de época 0 con un linaje forjado
muy alto. El receptor lo adopta, el atacante lee en pasivo todo lo que el receptor escriba después, el
extremo legítimo queda fuera y no hay recuperación automática.»

## 6. W-8: no fue Noise

El resumen dice que el relay no puede leer ni fabricar «contenido protegido por Noise». En las pruebas
se negoció **TLS 1.3**, y la manipulación de bytes se probó sobre TCP directo, no a través del relay.
Proponemos: «Dos pruebas de Go lo acotan: manipular bytes en una conexión TCP directa provoca
`tls: bad record MAC` y corta la conexión sin entregar nada repetido, y un relay con un TLS espía no ve
lo que viaja por el circuito, cifrado de extremo a extremo. Se negoció TLS 1.3. La vía QUIC directa no
tiene test. Sigue sin haber contador de aplicación frente a extremos comprometidos.»

## 7. Estado real de las recomendaciones (§5)

| Recomendación | Estado |
|---|---|
| 5.1 Retirar el lenguaje que sugiera PFS/KCI | **Aplicada** en `f119237`, que figura entre los commits contrastados: README, security-model y especificación |
| 5.2 Autenticación de linaje y vinculación de identidad | **Congelada** hasta la auditoría oficial (cambia el formato de red o la regla del linaje). Firmar los sobres perdería la negación que hoy existe de hecho, y el borrador no lo menciona |
| 5.3 Persistir el estado de invites | **Decidido no hacerlo**, y declarado como W-14. Persistir en preferencias dejaría metadatos fuera de la base cifrada, reutilizar la tabla de vistos tocaría un componente en revisión, y una tabla nueva exige una migración. El informe debería citar esa decisión o rebatirla |
| 5.4 Señales con `callId`, tipo y emisor, y deduplicar antes de notificar | **Ya existe** desde `fe21111`. «Versionar con estado monotónico» sería un cambio del formato de red |
| 5.5 Reproducibilidad separada de las afirmaciones de seguridad | **Práctica actual** desde `8d02875` |
| 5.6 W-7 y W-11 como riesgos de largo plazo | De acuerdo |

## 8. W-1 y W-10 deberían estar en la tabla

El borrador las rebaja a «recomendaciones de robustez y documentación». W-1 es la excepción declarada
al secreto hacia adelante, y el propio informe se niega a afirmar ese secreto. Proponemos:

| ID | Severidad | Estado | Consecuencia |
|---|---|---|---|
| W-1 | A su criterio | Abierto, declarado | La época 0 es derivable de `S` y no tiene secreto hacia adelante; si el primer mensaje del usuario sale en ella depende de los relojes |
| W-10 | Informativa | Abierto | Las etiquetas del buzón y el rendezvous son derivables de `S` indefinidamente: quien obtenga una identidad puede atribuir a esa pareja las etiquetas de volcados pasados |

## 9. Bytes NUL

§4 dice que el escaneo de `ebf2d43` no encontró bytes NUL. Falta el motivo: ese commit **eliminó** los
tres que `fe21111` había introducido en `CallService.kt` y `ChatService.kt`, que hacían que `grep` no
encontrara nada en esos archivos.

## 10. Evidencia que no hemos recibido

De los 11 documentos que enlaza el borrador solo tenemos cuatro (`05-revision-diseno`,
`08-verificacion-dinamica`, `10-registro-hallazgos` y `retest-h4-h5-w6-2026-09-15`). Para contrastar
el informe necesitamos:

- `docs/01-manifiesto-y-entorno.md`
- `docs/06-revision-codigo-kotlin.md`
- `docs/07-revision-codigo-go.md`
- `docs/09-analisis-formal.md`
- `evidence/especificacion-normalizada-fase1-2026-09-14.md`
- `evidence/retest-completo-f8d9a75-2026-09-15.md`
- `evidence/retest-ebf2d43-h5-2026-09-15.md`

## 11. Autoría, método y cómo se nombra

Aunque sea interno, el informe debería decir **quién lo hizo y con qué método**: manual, automatizado
o asistido. Y no debería poder leerse como la auditoría oficial que se está preparando. En concreto:

- **Título:** «revisión criptográfica independiente» se confunde con esa auditoría. Sugerimos algo como
  «revisión preparatoria independiente».
- **Objeto congelado:** llamar a `a97cbab` «revisión externa inicial» refuerza la confusión. Sugerimos:
  «commit de la etiqueta `revision-externa-1`, preparada para la auditoría oficial».
- **Referencia al análisis formal:** que enlace no se titule «análisis formal focalizado», si el propio
  informe dice que el modelo no se ha ejecutado.

---

Estamos de acuerdo, sin reparos, con:

- la formulación de C-001 y su absorción por H-7 y W-14;
- las filas de W-7, W-9, W-11 y W-13;
- que W-12 y las correcciones de H-4, H-5 y W-6 pueden cambiar el formato de red o la regla del linaje;
- la separación entre la reproducibilidad del AAR y las afirmaciones de seguridad;
- la sección de limitaciones y declaración, y el criterio de no presentar como vulnerabilidades
  demostradas lo que no tiene su modelo de amenaza.
