# Hallazgos API (`app/api/**/route.ts`)

- **[Autenticación y roles]**
  - **Admin protegido**: `app/api/admin/analytics/route.ts` usa `requireAdmin()` (líneas 12-16) y consulta múltiples modelos.
  - **Cliente autenticado**: `app/api/account/addresses/route.ts` usa `getServerSession(authOptions)` (líneas 9-16, 43-51) y verifica rol `client`.
  - **Sin auth explícita**: `app/api/drivers/connect/route.ts` no usa sesión ni middleware; permite cambiar `availabilityStatus` y asignar rutas (líneas 21-37, 52-73).
  - **Autorización contextual**: `app/api/orders/[id]/confirm-delivery/route.ts` valida sesión y roles (`driver`/`supplier`) antes de actualizar orden (líneas 15-40).

- **[Validación (Zod)]**
  - **Presente**: `orders/[id]/retry-payment` define `RetryPaymentSchema` con `zod` y valida opcionalmente payload (líneas 15-25).
  - **Presente**: `orders/[id]/confirm-delivery` usa `ConfirmDeliverySchema` (líneas 7, 18-21).
  - **Presente**: `drivers/connect` usa `DriverConnectSchema` (líneas 8, 24-27).
  - **Ausente/Manual**: `account/addresses` valida campos manualmente (líneas 55-61), sin `zod`.

- **[Manejo de errores]**
  - Patrón general de `try/catch` y `NextResponse.json` con `status` apropiado en los endpoints auditados (
    - `admin/analytics` líneas 12-16, 291-297;
    - `account/addresses` líneas 32-38, 106-113;
    - `orders/[id]/retry-payment` líneas 91-95;
    - `orders/[id]/confirm-delivery` líneas 49-53;
    - `drivers/connect` líneas 76-81).

- **[Idempotencia]**
  - **Aplicada**: `mercadopago/webhook` implementa locks (`acquireLock`) y verifica duplicados con `PaymentLog` (líneas 117-141) y locks específicos por contexto (líneas 168-175, 214-218, 278-283). Considerado robusto.
  - **No aplicada**: `orders/[id]/retry-payment` crea nueva preferencia MP sin chequear reintentos repetidos (potenciales duplicados si el cliente reintenta rápidamente).

- **[Dependencias externas]**
  - **DB (Mongo/Mongoose)**: Todos los auditados usan `connectToDatabase`/modelos (`Order`, `Driver`, `Route`, `User`).
  - **Mercado Pago**: `orders/[id]/retry-payment` usa `createMercadoPagoPreference`; `mercadopago/webhook` consume API MP y valida firma.
  - **Resend/WebSocket**: `mercadopago/webhook` puede enviar emails y emitir eventos Socket.IO.
  - **Redis/Socket.IO**: `pages/api/socket.ts` integra `@socket.io/redis-adapter` condicionalmente.

- **[Observaciones de seguridad]**
  - Endpoints sin auth (ej. `drivers/connect`) modifican estado sensible; debería al menos requerir sesión de `driver` o token de servicio.
  - Validaciones con `zod` no son uniformes; algunos endpoints críticos usan validación manual.
  - Roles verificados directamente con strings; recomendable estandarizar con helpers (`requireRole('driver'|'supplier'|...)`).
