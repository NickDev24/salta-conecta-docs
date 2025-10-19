# Checklist (07-integrations)

- **[MercadoPago]**
  - Acceso server: `MERCADOPAGO_ACCESS_TOKEN|MP_ACCESS_TOKEN` presentes y seguros.
  - Webhook: firma con `NEXT_WEBHOOK_MERCADOPAGO_KEY_SECRET|MERCADOPAGO_WEBHOOK_SECRET`; idempotencia activa.
  - `retry-payment`: evitar preferencias duplicadas (bloqueo/registro).
  - `NEXT_PUBLIC_MERCADOPAGO_PUBLIC_KEY` solo en cliente.

- **[Cloudinary]**
  - `CLOUDINARY_*` configuradas; validar tipo/tamaño de archivo antes de subir.
  - `NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME` expuesta; jamás `API_SECRET`.

- **[Resend]**
  - `RESEND_API_KEY`/`RESEND_FROM` configuradas; TLS y DMARC/SPF en dominio.
  - Webhooks (si se usan): `RESEND_WEBHOOK_SECRET` y verificación.

- **[Redis/Socket.IO]**
  - `REDIS_URL` definido; reconexión/backoff adecuados.
  - CORS de Socket.IO restringido a `NEXT_PUBLIC_APP_URL`.

- **[Google Maps]**
  - `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` restringida por dominio/quotas.
  - CSP compatible con Google Maps SDK.
