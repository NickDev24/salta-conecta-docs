# Recomendaciones (05-data-models)

- **[Conexión Mongo resiliente]**
  - Establecer opciones de conexión: `serverSelectionTimeoutMS=5000-10000`, `socketTimeoutMS=20000-30000`, `maxPoolSize=20-50`, `minPoolSize=2-5`, `retryWrites=true`, `w=majority`.
  - Considerar `readPreference=primaryPreferred` para lecturas en rutas no críticas.
  - Configurar `autoIndex=false` en producción y construir índices vía migraciones/deploy.

- **[TTL y housekeeping]**
  - Activar TTL en `PaymentLog` (ej. 90 días) y evaluar TTL para tracking antiguo (`DriverLocationHistory`) si crece rápidamente.
  - Asegurar `expiresAt` donde sea necesario (notificaciones, tokens temporales si existieran).

- **[Índices compuestos]**
  - Revisar planes de consulta (profiler) y añadir índices compuestos adicionales: `Order(status, createdAt)`, `Order(supplier, paymentStatus, createdAt)`, `Order(assignedDriverId, status)` ya existe.
  - Validar `sparse/partialFilterExpression` para reducir tamaño donde aplique.

- **[Esquemas estrictos]**
  - Reemplazar `Schema.Types.Mixed` por subesquemas tipados donde sea posible (`paymentDetails`, `metadata`, `request/response`).
  - Añadir validaciones y límites (`maxlength`, `min`, `enum`).

- **[Uso de lean()]**
  - Documentar cuándo usar `.lean()` y evitarlo cuando se necesitan métodos del documento o virtuals.

- **[Migraciones/seed]**
  - Mantener scripts de migración de índices y datos alineados con cambios de esquema.
