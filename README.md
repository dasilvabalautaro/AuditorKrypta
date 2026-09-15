# AuditorKrypta

Repositorio de trabajo para la revisión independiente del diseño criptográfico y del camino criptográfico de [Krypta](https://github.com/dasilvabalautaro/Krypta).

La auditoría está fijada inicialmente al commit de Krypta `a97cbabd95e3787deca872e2157a64473073af6c`. Cubre el ratchet por épocas, la negociación de capacidades, las claves de llamada, las etiquetas del buzón ciego y su implementación en Kotlin y Go.

## Documentación

- [Plan de auditoría](docs/00-plan-auditoria.md)
- [Manifiesto y entorno](docs/01-manifiesto-y-entorno.md)
- [Diario de auditoría](docs/diario-auditoria.md)
- [Evidencia de la línea base de pruebas](evidence/linea-base-pruebas-2026-09-14.md)
- [Revisión de la actualización AAR/APK](evidence/revision-aar-apk-2026-09-14.md)
- [Reconstrucción reproducible del AAR posterior](evidence/revision-aar-reproducible-8d-2026-09-14.md)
- [Especificación criptográfica normalizada (Fase 1)](evidence/especificacion-normalizada-fase1-2026-09-14.md)
- [Verificación dinámica inicial](docs/08-verificacion-dinamica.md)
- [Verificación dinámica del commit `fe21111f`](evidence/revision-dinamica-fe21111-2026-09-15.md)

Los resultados se incorporarán progresivamente y distinguirán entre afirmaciones del diseño, comportamiento del código y propiedades demostradas por pruebas.

## Divulgación responsable

Este repositorio contendrá únicamente material apto para publicación. Credenciales, notas locales y detalles explotables de vulnerabilidades todavía abiertas se mantienen fuera de Git mediante las reglas de `.gitignore`.

## Licencia

Apache License 2.0. Véase [LICENSE](LICENSE).
