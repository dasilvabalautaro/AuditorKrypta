# Análisis formal focalizado

## Estado de herramientas

ProVerif 2.05 está instalado en el entorno OPAM y se ejecutó con el entorno reproducible de la auditoría. Tamarin no forma parte de esta repetición. El modelo abstracto y sus límites están en [`models/ratchet-core.md`](../models/ratchet-core.md); los modelos ejecutables son [`models/ratchet-secrecy.pv`](../models/ratchet-secrecy.pv) y [`models/ratchet-kci.pv`](../models/ratchet-kci.pv).

## Resultado ejecutado

En `ratchet-secrecy.pv`, ProVerif devuelve `true` para `not attacker(message)` y para la correspondencia `opened(message) ==> sent(message)`. Esto demuestra confidencialidad e integridad de cabecera solo en el modelo idealizado.

En `ratchet-kci.pv`, ProVerif devuelve `false` para la correspondencia entre aceptación como par y envío legítimo, y genera una traza en la que el atacante conoce `S`, fabrica el sobre y el receptor lo acepta. Esto reproduce formalmente H-5/KCI bajo el modelo actual.

## Resultado preliminar por inspección

El modelo reproduce tres contraejemplos ya fijados por pruebas JVM: secuestro por linaje (H-4), pérdida de convergencia con reloj atrasado (W-6) y suplantación KCI al nivel de `S` (H-5). Esto valida que el modelo contiene las transiciones relevantes, pero no demuestra las propiedades positivas de confidencialidad, PFS o recuperación.

## Trabajo pendiente para cerrar la fase formal

1. Traducir persistencia, deduplicación, concurrencia y transporte a modelos separados.
2. Modelar compromiso de estado, borrado de privadas efímeras y recuperación poscompromiso.
3. Comparar trazas del modelo con `RatchetTest`, `RatchetPropertyTest` y `RatchetSessionsTest`.
4. Ejecutar Tamarin si se requiere una segunda formalización independiente.
