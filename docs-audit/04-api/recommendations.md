# Recomendaciones (04-api)

- **[Autenticación obligatoria]**
  - Requerir sesión en endpoints que modifican estado (`drivers/connect`, otros similares). Usar `getServerSession` o middleware central.

- **[Autorización por helper]**
  - Estandarizar `requireRole('admin'|'driver'|'supplier'|'client')` y usarlo en rutas con control de acceso. Evitar checks ad-hoc por `session.user.role === '...'.`

- **[Validación con Zod]**
  - Adoptar `zod` y `safeParse` en todos los endpoints que aceptan body/params/query. Responder 400 con `issues` formateados.

- **[Idempotencia pagos]**
  - Extender estrategia de `mercadopago/webhook` a `orders/[id]/retry-payment`: registrar intentos, bloquear multi-click, y validar estado previo para evitar preferencias duplicadas.

- **[Errores estándar]**
  - Respuestas JSON con estructura `{ error: code, message?: string }`, códigos correctos y sin detalles internos. Loggear internamente.

- **[Rate limiting / Anti-abuso]**
  - Añadir rate limiting en endpoints públicos y webhooks mediante `rate-limiter-flexible`.

- **[Observabilidad]**
  - Registrar auditoría y métricas en cambios de estado (entrega, asignación de rutas). Estandarizar `audit.push({ action, actorId, actorRole, at })`.

- **[Timeouts/retries]**
  - Para MP/Resend/Cloudinary usar `withRetry`/timeouts. Centralizar en `lib/*` y manejar errores de red.
