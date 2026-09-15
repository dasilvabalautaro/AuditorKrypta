**Asunto:** AK-2026-001 — observaciones al registro de hallazgos (verificado hasta `ebf2d43`)

Gracias por el registro. Los estados coinciden con los nuestros, y hemos comprobado los hashes que
cita (`ebf2d43…` es el HEAD y `8d028754…` el commit del AAR reproducible). Proponemos estas
correcciones antes de que el registro pase al informe publicable.

**1. Faltan tres limitaciones abiertas.** Ya lo indicamos en la respuesta a la revisión de diseño.
Las tres están declaradas en la especificación §13, y el criterio de cierre del propio registro pide
que las limitaciones sigan visibles. Proponemos añadir estas filas (la severidad queda a su criterio):

| ID | Severidad provisional | Descripción | Estado | Recomendación |
|---|---|---|---|---|
| W-7 | Media (largo plazo) | Sin protección post-cuántica: todo es X25519 y el PeerID *es* la clave pública, así que el tráfico grabado hoy sería descifrable de principio a fin por un adversario cuántico futuro | Abierto; diseñado (`DISENO-postcuantico.md`), sin implementar a propósito | Ratchet PQ lento híbrido tras la revisión externa |
| W-11 | Media | El `.krbk` solo lo protege la frase (PBKDF2, 310 000 iteraciones): quien lo obtenga puede probar frases sin límite. Es además la vía natural hacia H-5 (robar la identidad propia) | Abierto, documentado | KDF resistente a memoria (p. ej. Argon2id) o factor adicional |
| W-9 | Informativa | `PN` se transmite y no se usa | Abierto | Aclarar en la revisión si falta una comprobación |

**2. W-13 está incompleto.** Proponemos esta descripción: «El nodo ve el grafo diario de parejas en
la DHT, la presencia (wake), quién habla con quién en vivo (relay) y, por identify, qué protocolos
admite cada teléfono». El grafo de parejas es el metadato de más peso y ahora no aparece.

**3. Redacción de W-8.** «Noise/TLS rechaza duplicación, reordenamiento y reflexión dentro del
circuito» describe una prueba que no se ejecutó tal cual. Proponemos: «Dos pruebas de Go lo acotan:
un intermediario que duplica, reordena o refleja bytes en una conexión TCP directa provoca
`tls: bad record MAC` y la caída de la conexión sin entregar nada repetido, y un relay con un TLS
espía no ve lo que viaja por el circuito, cifrado de extremo a extremo. Se negoció TLS 1.3. La vía
QUIC directa no tiene test (por inspección, no usa 0-RTT). Sigue sin haber contador de aplicación
frente a extremos comprometidos».

**4. Evidencia.**

- **H-6:** en lugar de «suite JVM», el test concreto `ChatServiceTest` → `volver a anadir a un
  contacto bloqueado no lo desbloquea ni olvida su version`.
- **Limitaciones abiertas:** proponemos añadir la columna de evidencia, porque tres están fijadas en
  tests y conviene distinguirlas de las que solo están declaradas:
  - H-4: `RatchetTest` → `con el secreto compartido un linaje forjado secuestra la sesion y no hay
    vuelta atras`;
  - H-5: `ChatServiceTest` → `con tu identidad robada te pueden escribir como cualquier contacto por
    el buzon ciego`, que distingue el buzón ciego, el buzón con PeerID y un nodo honrado, y el nodo
    que miente;
  - W-6: `RatchetTest` → `tras perder el estado con el reloj atrasado un sentido queda roto y no se
    arregla solo`.

**5. Coste de algunas recomendaciones.** Pedimos que el informe lo recoja, para que no parezcan
ajustes menores:

- **W-12:** separar las claves cambia el formato de red y todos los secretos compartidos, porque `S`
  se deriva convirtiendo la identidad Ed25519.
- **W-6:** un linaje monótono duradero no cubre la importación de un `.krbk` en un móvil nuevo, salvo
  que el último linaje viaje en el respaldo.
- **H-4, H-5, W-6 y W-12** tocan el formato de red o la regla del linaje, que están congelados hasta
  el informe externo (`REVISION-protocolo-2026-09-14.md` §5.4).

**6. H-7.** «Corregido parcialmente» nos parece defendible. Nosotros lo registramos como corregido y
declaramos lo restante como W-14 aceptada. Lo único importante es que W-14 figure, y figura.

**7. Nota informativa sugerida.** Como el registro se verifica hasta `ebf2d43`: ese commit eliminó
tres bytes NUL literales que `fe21111` había dejado en `CallService.kt` y `ChatService.kt`. El
comportamiento no cambiaba, pero `grep` trataba ambos archivos como binarios y no encontraba nada en
ellos, lo que afectaba a la revisión de `ChatService.kt`. Un test fija ahora el identificador
afectado.

Estamos de acuerdo, sin reparos, con: C-001 absorbido por H-7 y W-14, las severidades de H-4 y H-5,
la separación entre la procedencia del AAR de `revision-externa-1` y la reproducibilidad posterior, y
el criterio de cierre.
