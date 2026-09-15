# Modelo abstracto del núcleo del ratchet

**Estado:** especificación para futura ejecución en Tamarin/ProVerif; no ejecutada porque ninguna de las herramientas está instalada.

## Sorts y funciones

```text
S                : secret compartido estático
L, E, N          : linaje, época y contador
dh(x,y)          : X25519
root(S,L)        : HKDF(S, vacío, "krypta-rtc-root:0:" || L)
root_dh(dh,RK,E) : HKDF(dh, RK, "krypta-rtc-root:" || E)
chain(RK,E,D)    : HKDF(RK, vacío, "krypta-rtc-chain:" || E || ":" || D)
mk(CK,N)         : HMAC(CK_N, 0x01)
nonce(mk)        : HKDF(mk, vacío, "krypta-rtc-msg", 44)[32..43]
seal(mk,h,p)     : AEAD(mk[0..31], nonce(mk), p, h)
```

## Reglas mínimas

1. Cada lado crea `root(S,L)` y dos cadenas direccionales en época 0.
2. Un mensaje consume `mk(CK,N)` y avanza la cadena; el header autenticado contiene `(L,E,N,PN,cur_pub,next_pub)`.
3. Si un header válido tiene `L > L_local`, el receptor ejecuta `reset` y adopta el linaje.
4. Si tiene la propuesta pública de la época siguiente, ambos calculan `root_dh` y avanzan.
5. El receptor puede abrir época 0 de cualquier linaje porque conoce `S`.
6. La deduplicación por hash del wire ocurre fuera del modelo criptográfico.

## Consultas objetivo

- **Secrecy:** un observador sin `S` no obtiene `p` de un `seal` válido.
- **Header integrity:** modificar `(L,E,N,PN,pub)` invalida `seal`.
- **Key uniqueness:** no existen dos emisiones del mismo emisor con `(L,E,N)` igual bajo linajes monotónicos.
- **Forward secrecy:** tras borrar `priv_e`, un compromiso posterior no revela mensajes de épocas anteriores.
- **Post-compromise recovery:** tras un DH no observado por el atacante, los mensajes posteriores vuelven a ser secretos.
- **No downgrade/KCI:** conocer `S` no debería permitir hacerse pasar por la identidad concreta (propiedad actualmente no satisfecha).

## Contraejemplos ya reproducidos

- **H-4:** un adversario que conoce `S` elige `L = Long.MAX_VALUE/2`; `reset` lo adopta y el extremo legítimo queda fuera.
- **W-6:** tras perder el estado, un `L` menor permite enviar época 0, pero impide abrir las épocas avanzadas del otro sentido.
- **H-5:** conocer la identidad privada y el PeerID del contacto permite reconstruir `S` y fabricar sobres/etiquetas indistinguibles de los legítimos.

## Diferencias implementación–modelo

- El modelo trata HKDF/X25519/AEAD como funciones ideales; no cubre JCA, gomobile, SQLCipher ni errores de memoria.
- El modelo no representa mutex, transacciones, poda de `seen`, reinicios ni el transporte Noise; esas capas requieren lemas separados.
- La época 0 re-derivable y la adopción de linaje son decisiones explícitas del diseño, no supuestos ocultos.
