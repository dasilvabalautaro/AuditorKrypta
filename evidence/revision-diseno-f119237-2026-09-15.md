# Revisión de diseño actualizada: `f119237e`

**Commit revisado:** `f119237e67affe62a935926cedeba2950ed82955`  
**Estado posterior verificado:** `f8d9a75d76043185ddfbabfc0e645fcf32fb95a2`.

## Cambios relevantes

- W-6 se reformula con su efecto real: tras perder el estado con el reloj atrasado, los mensajes salientes pueden llegar en época 0 mientras los entrantes se pierden; corregir el reloj no repara automáticamente la sesión.
- Se añade W-14: la deduplicación de invites vive en RAM, por lo que un reinicio dentro de la ventana puede permitir un timbre adicional.
- Krypta deja de presentar el secreto hacia adelante como garantía general y elimina promesas de resistencia KCI o negación plausible.

## Verificación independiente

Se creó un clon aislado del estado `f8d9a75…` y se ejecutaron `RatchetTest`, `CallServiceTest` y `ChatServiceTest`: `BUILD SUCCESSFUL`. La revisión de `f119237e…` no encontró cambios de producción criptográfica, formato de red ni esquema. La actualización `f8d9a75…` de ayuda es coherente con W-6.

## Evaluación

La documentación mejora la precisión y hace explícitos W-6/W-14 y los límites de H-4/H-5; no los elimina ni constituye una solución criptográfica. La prueba JVM confirma regresión funcional, no una prueba formal de seguridad.
