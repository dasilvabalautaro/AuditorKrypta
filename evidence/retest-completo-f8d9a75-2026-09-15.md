# Retest completo del estado `f8d9a75`

**Fecha:** 15 de septiembre de 2026  
**Estado verificado:** `f8d9a75d76043185ddfbabfc0e645fcf32fb95a2`.

## Ejecuciones

- Gradle `testDebugUnitTest` en la copia aislada: `BUILD SUCCESSFUL`, 127 tareas, 0 fallos.
- Go del puente `go test -race -count=1 ./...`: pasa (`ok chat.neto.krypta/nativego`).
- Las pruebas dirigidas de `RatchetTest`, `CallServiceTest` y `ChatServiceTest` ya habían pasado en la misma copia.

## Alcance del cambio revisado

`f119237e…` solo modifica documentación, especificación, modelo de seguridad y pruebas de ratchet; `f8d9a75…` modifica ayuda/manual y su test. No cambia primitivas, formato de red ni esquema de base de datos.

## Conclusión

El estado más reciente mantiene las regresiones funcionales en verde. W-6 y W-14 deben permanecer en el informe como limitaciones explícitas; el éxito de la suite no constituye una prueba formal de seguridad ni elimina H-4/H-5.
