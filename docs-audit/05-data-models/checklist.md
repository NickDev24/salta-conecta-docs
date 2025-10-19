# Checklist (05-data-models)

- **[Conexión Mongo]**
  - Definir `serverSelectionTimeoutMS`, `socketTimeoutMS`, `maxPoolSize`, `minPoolSize`, `retryWrites`, `w`, `readPreference` según entorno.
  - Validar `MONGODB_URI`/`MONGODB_DB` y fallback seguro.
  - Habilitar/deshabilitar `autoIndex` por entorno (off en prod si aplica).

- **[Índices]**
  - Verificar índices compuestos para queries críticas (`Order.status+paymentStatus+createdAt`, `supplier+status`, `assignedDriverId+status`).
  - Confirmar `sparse/unique` donde corresponda (`orderNumber` unique ya presente).
  - Revisar índices en subdocumentos usados en filtros (e.g., `shippingAddress.city`).

- **[TTL]**
  - Activar TTL en colecciones efímeras: `PaymentLog` (90d), tracking de ubicaciones antiguo, notificaciones expiradas (ya activo).
  - Asegurar `expiresAt` populado cuando se requiera TTL dinámico.

- **[Lean()]**
  - Usar `.lean()` en lecturas sin mutación ni hooks; evitar en operaciones que dependan de métodos virtuales o middlewares.

- **[Validación de esquema]**
  - Enumeraciones/rangos en campos numéricos/strings críticos.
  - Normalizar tipos `Schema.Types.Mixed` con subesquemas cuando sea posible.

- **[Resiliencia]**
  - Retries/backoff para operaciones críticas (pagos/logs) a nivel de aplicación.
  - Manejo de errores y logs en operaciones de escritura (create/update) sensibles.
