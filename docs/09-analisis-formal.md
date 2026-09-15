# Análisis formal focalizado

## Estado de herramientas

En el entorno de auditoría no están instalados Tamarin ni ProVerif. Por ello no se presentan resultados de un verificador automático. El modelo abstracto y las consultas previstas están en [`models/ratchet-core.md`](../models/ratchet-core.md).

## Resultado preliminar por inspección

El modelo reproduce tres contraejemplos ya fijados por pruebas JVM: secuestro por linaje (H-4), pérdida de convergencia con reloj atrasado (W-6) y suplantación KCI al nivel de `S` (H-5). Esto valida que el modelo contiene las transiciones relevantes, pero no demuestra las propiedades positivas de confidencialidad, PFS o recuperación.

## Trabajo pendiente para cerrar la fase formal

1. Traducir las reglas a Tamarin o ProVerif en un entorno reproducible.
2. Modelar compromiso de `S`, compromiso de estado y borrado de privadas efímeras.
3. Añadir persistencia/deduplicación como sistema de eventos separado.
4. Comparar trazas del modelo con `RatchetTest`, `RatchetPropertyTest` y `RatchetSessionsTest`.
5. Registrar consultas, versión de la herramienta, tiempos y límites de búsqueda.
