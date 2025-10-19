# Integrations Matrix (flujos y riesgos)

## MercadoPago

- **Claves**: `MERCADOPAGO_ACCESS_TOKEN|MP_ACCESS_TOKEN`, `NEXT_PUBLIC_MERCADOPAGO_PUBLIC_KEY`, `MERCADOPAGO_APPLICATION_ID|MP_APPLICATION_ID`, `NEXT_WEBHOOK_MERCADOPAGO_KEY_SECRET|MERCADOPAGO_WEBHOOK_SECRET`, `MP_CLIENT_ID|MP_CLIENT_SECRET|MP_REDIRECT_URI`.
- **Flujo**:
  - Cliente inicia pago → preferencia vía `createMercadoPagoPreference()` (`lib/mercadopago.ts`), `back_urls` y `notification_url` con `NEXT_PUBLIC_APP_URL`.
  - MP llama webhook → `app/api/mercadopago/webhook/route.ts` valida firma, aplica idempotencia (locks + `PaymentLog`), actualiza `Order`, dispara notificaciones.
- **Riesgos**:
  - Duplicidad de preferencias si no hay idempotencia en `retry-payment`.
  - CSP para SDK MP y exposición de `NEXT_PUBLIC_*` correctamente limitadas.

## Cloudinary

- **Claves**: `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`, `NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_UPLOAD_PRESET`.
- **Flujo**:
  - Subidas desde server (upload_stream) y generación de URLs optimizadas.
- **Riesgos**:
  - Validación de tipo/tamaño de archivo fuera del helper.
  - Exposición de `NEXT_PUBLIC_*` solo para cloud name (no secrets).

## Resend

- **Claves**: `RESEND_API_KEY`, `RESEND_FROM`, `RESEND_WEBHOOK_SECRET` (si se usan webhooks).
- **Flujo**:
  - Envío de emails transaccionales con `sendEmail()` y helpers de plantillas. Enlaces referencian `NEXT_PUBLIC_APP_URL`.
- **Riesgos**:
  - Gestión de rebotes/webhooks a validar.
  - Evitar exponer `RESEND_API_KEY` al cliente.

## Redis/Socket.IO

- **Claves**: `REDIS_URL`.
- **Flujo**:
  - `pages/api/socket.ts` inicia servidor Socket.IO y setea adapter Redis si hay `REDIS_URL`.
- **Riesgos**:
  - CORS excesivo si `NEXT_PUBLIC_APP_URL` no está bien configurado.
  - Reconexión/backoff de Redis a evaluar para resiliencia.

## Google Maps

- **Claves**: `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY`.
- **Flujo**:
  - Uso con `@react-google-maps/api` y configuración en `config/maps.ts`.
- **Riesgos**:
  - Restricciones de API key (dominios/quotas) y CSP.
