# Hallazgos (07-integrations)

- **[MercadoPago]** `lib/mercadopago.ts`
  - Usa `MERCADOPAGO_ACCESS_TOKEN|MP_ACCESS_TOKEN` (línea 5) y `NEXT_PUBLIC_APP_URL` para `back_urls` y `notification_url` (líneas 121-137).
  - Timeout explícito (5000ms) en config del SDK; sin reintentos automáticos (línea 7).
  - Firma de webhook validada con `validateWebhookSignature` y secreto `NEXT_WEBHOOK_MERCADOPAGO_KEY_SECRET|MERCADOPAGO_WEBHOOK_SECRET` (ver `app/api/mercadopago/webhook/route.ts`).
  - Flujo de reembolsos y obtención de pagos presente; idempotencia principal implementada en webhook, no en creación de preferencias.

- **[Cloudinary]** `lib/cloudinary.ts`
  - Requiere `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` (líneas 5-9). Usa `upload_stream` y helpers de URL.
  - Valida configuración antes de operar (`isCloudinaryConfigured()`).
  - Sin control de tamaño/tipo de archivo en este helper (asumido en validaciones previas).

- **[Resend]** `lib/resend.ts`
  - Requiere `RESEND_API_KEY` y `RESEND_FROM` (líneas 3-7, 35-37, 48-56).
  - Plantillas HTML con botones que referencian `NEXT_PUBLIC_APP_URL` para enlaces (líneas 103-106, 181-183, 266-291).
  - No se observa manejo de webhooks aquí; la validación de webhooks aparece en `.env.example` (`RESEND_WEBHOOK_SECRET`).

- **[Redis/Socket.IO]** `lib/redis.ts`, `pages/api/socket.ts`
  - Redis ioredis via `REDIS_URL` (líneas 7-13) con `maxRetriesPerRequest: 2`.
  - `pages/api/socket.ts` crea servidor Socket.IO, con CORS desde `NEXT_PUBLIC_APP_URL` y usa adapter Redis si hay cliente (líneas 31-47).
  - No hay claves específicas de Socket.IO; depende de `REDIS_URL` y orígenes públicos.

- **[Google Maps]** `config/maps.ts`
  - Config UI; la clave cliente es `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` (expuesta en `next.config.js`).

- **[Notas de seguridad]**
  - Aislar secretos en server: `MERCADOPAGO_ACCESS_TOKEN`, `RESEND_API_KEY`, `CLOUDINARY_API_SECRET`, `REDIS_URL` nunca deben exponerse al cliente.
  - Endurecer CSP si se usan SDKs de terceros (MP/Maps) y utilizar `nonce` para inline script.
  - Validar tamaños de archivo y tipos en rutas de subida que llamen a Cloudinary.
