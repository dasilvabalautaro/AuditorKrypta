# AuditorKrypta

Repositorio de trabajo para la revisión independiente del diseño criptográfico y del camino criptográfico de [Krypta](https://github.com/dasilvabalautaro/Krypta).

La auditoría está fijada inicialmente al commit de Krypta `a97cbabd95e3787deca872e2157a64473073af6c`. Cubre el ratchet por épocas, la negociación de capacidades, las claves de llamada, las etiquetas del buzón ciego y su implementación en Kotlin y Go.

## Documentación

- [Plan de auditoría](docs/00-plan-auditoria.md)

Los resultados se incorporarán progresivamente y distinguirán entre afirmaciones del diseño, comportamiento del código y propiedades demostradas por pruebas.

## Divulgación responsable

Este repositorio contendrá únicamente material apto para publicación. Credenciales, notas locales y detalles explotables de vulnerabilidades todavía abiertas se mantienen fuera de Git mediante las reglas de `.gitignore`.

## Licencia

Apache License 2.0. Véase [LICENSE](LICENSE).
