# Hallazgos de Data Models (models/**, schemas/**, MongoDB)

- **[Conexión Mongo estable pero sin retries ni timeouts]** en `lib/mongodb.ts` líneas 11-19 y 15-18:
  - Usa cache global de `mongoose` (promesa) y `mongoose.connect(MONGODB_URI, { dbName })`.
  - No define `serverSelectionTimeoutMS`, `socketTimeoutMS`, `maxPoolSize`, `retryWrites`, ni `autoIndex` explícitos.
  - Si `MONGODB_URI` falta, lanza error temprano (línea 13).

- **[Índices bien trabajados en modelos principales]**
  - `models/Order.ts`: múltiples índices compuestos para consultas comunes (líneas 173-180), `timestamps: true` (línea 171).
  - `models/Notification.ts`: índices por usuario/estado/prioridad (líneas 96-101) y **TTL** sobre `expiresAt` (línea 100).
  - `models/DriverLocationHistory.ts`: índices por `driverId/userId` + `recordedAt` (líneas 33-36), `timestamps: true` (línea 30).
  - `models/PaymentLog.ts`: índices compuestos (líneas 65-69), TTL comentado (línea 71).

- **[TTL (time-to-live) parcialmente aplicado]**
  - Activo en `Notification` sobre `expiresAt` (`expireAfterSeconds: 0`).
  - Inactivo en `PaymentLog` (comentado → oportunidad para limpieza automática de logs antiguos).

- **[Uso de `lean()` extendido]**
  - Se detectaron >100 usos de `lean()` en consultas (e.g., admin dashboards, analytics). Buen patrón para lecturas.
  - Asegurar coherencia: `lean()` sólo en rutas/métodos que no dependan de getters/middlewares de documentos.

- **[Buenas prácticas de schema]**
  - Validaciones `min/max/enum` en campos numéricos y string (e.g., `Order`, `DriverLocationHistory`).
  - Subdocumentos: `OrderItemSchema`, `ShippingAddressSchema` en `Order` con tipos explícitos.
  - `timestamps: true` usado en varios esquemas (auditoría básica de creación/actualización).

- **[Lógica de negocio en middleware/modelo]**
  - `OrderSchema.pre('save')` recalcula totales y asegura no-negatividad (líneas 182-197). Correcto para consistencia.
  - Métodos estáticos en `Order` y `Notification` para consultas comunes (p. ej., `findByCustomer`, `getStats`, `findByUser`).

- **[Oportunidades]**
  - Uniformar TTLs para colecciones con datos efímeros (logs, tracking, notificaciones caducadas).
  - Revisar índices compuestos adicionales (e.g., `Order` por `status+createdAt`, `supplier+paymentStatus`).
  - Definir opciones de conexión (pooling, timeouts, retries) para resiliencia en prod.
