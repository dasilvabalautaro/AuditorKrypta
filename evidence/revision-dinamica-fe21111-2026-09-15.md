# Verificación dinámica del commit remoto `fe21111f`

**Fecha:** 15 de septiembre de 2026  
**Commit revisado:** `fe21111f5dfe4d9ee80e8b8e5972152ee57b53b2`  
**Padre:** `ea088b1f7bc8d029f708b73040d09d42c7e02536`  
**Alcance:** corrección de replay de señales de llamada (H-7) y verificación del transporte relayed (W-8).

## Procedencia y preservación

El commit se confirmó mediante la API pública de GitHub y una clonación temporal bajo `.audit-work/krypta-fe`. El clon se fijó en `fe21111f…`; el proyecto Krypta original no se usó para compilar ni probar. Su estado local se consultó en modo lectura.

## Resultados

### H-7 / señales v1

El cambio añade deduplicación en memoria por `(contacto, callId)` durante la ventana de la llamada y una ventana adicional de diez minutos. También rechaza timestamps más de diez minutos en el futuro y hace determinista la fila de llamada perdida para no avisar dos veces por el mismo identificador.

La prueba JVM de `CallServiceTest` y `ChatServiceTest` terminó con `BUILD SUCCESSFUL`. Esto confirma la regresión cubierta por el autor y la compatibilidad del cambio con el camino de señalización. La deduplicación es volátil: se pierde al reiniciar el proceso, por lo que no equivale a una deduplicación persistente del buzón.

### W-8 / transporte relayed

La prueba Go `TestRelayNoVeLoQueViajaPorElCircuito` pasó, así como las pruebas de tampering/replay del transporte seleccionadas y la suite completa de `native-bridge/libp2p` (`go test -count=1 ./...`). La sonda demuestra que el relay puede observar sus propios registros y nombres de protocolo anunciados por identify, pero no el frame ni la negociación dentro del circuito. Un proxy que duplica, reordena o refleja bytes obtiene `tls: bad record MAC` y no entrega un frame repetido.

Esto acota W-8: para el adversario relay definido en el modelo, la capa de transporte impide que el relay modifique o reproduzca frames dentro del circuito. La prueba unitaria anterior que descifraba dos veces el mismo AES-GCM sigue siendo cierta como propiedad de la primitiva, pero no representa un ataque atravesando Noise/TLS relayed. El riesgo residual sería un endpoint comprometido o una ruta distinta al circuito, fuera de esta conclusión.

## Conteo de pruebas

- Go puente: suite completa, pasa.
- Go relay/transporte: pasa.
- JVM `CallServiceTest` + `ChatServiceTest`: pasa.
- El commit remoto declara 288 pruebas JVM totales; esta ejecución se concentró en los módulos afectados.

## Estado provisional

- H-7: corregido en `fe21111f…`, sujeto a la limitación de deduplicación en RAM.
- W-8: severidad baja/informativa para el relay bajo transporte relayed verificado; no debe describirse como replay explotable por el relay sin especificar otro camino.
- C-001: reformular como replay de señales v1 sin deduplicación persistente por `callId`; la fecha futura es una condición secundaria y ahora tiene límite superior.

Este documento está en el árbol de trabajo de AuditorKrypta y no se ha incorporado a ningún commit.
