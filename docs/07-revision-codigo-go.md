# Revisión de código Go — transporte y frontera criptográfica

**Commit de verificación:** `fe21111f5dfe4d9ee80e8b8e5972152ee57b53b2`.

## Observaciones

- `CallStream.ReadFrame` y `VideoStream.ReadFrame` implementan framing de longitud (`uint16` y `uint32` respectivamente). El framing no aporta autenticidad; la autenticidad depende de Noise/TLS y del AES-GCM de Kotlin.
- `TestRelayNoVeLoQueViajaPorElCircuito` instrumenta el transporte del relay. El relay ve sus propios registros y los protocolos anunciados por identify, pero no ve el frame ni la negociación dentro del circuito.
- Las pruebas de tampering/replay del transporte obtienen `tls: bad record MAC` ante duplicación, reordenamiento o reflexión de bytes y no entregan un frame repetido al consumidor.
- La suite completa del puente (`go test -count=1 ./...`) pasa en la copia aislada.

## Conclusión

Para el modelo de relay relayed probado, Noise/TLS proporciona el límite que faltaba para W-8: el operador no puede repetir o alterar frames dentro del circuito. No debe extrapolarse a un endpoint comprometido ni a una API que entregue el mismo frame dos veces después de descifrarlo.

La frontera Kotlin–Go queda correctamente separada: Go transporta bytes y Kotlin mantiene las claves y el descifrado de medios. No se observó una derivación de claves de llamada en Go.
