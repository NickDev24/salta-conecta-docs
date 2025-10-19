# Recomendaciones (07-integrations)

- **[MercadoPago]**
  - Idempotencia en reintentos: bloquear `orders/[id]/retry-payment` por ventana corta y registrar intentos; evitar preferencias duplicadas.
  - Asegurar `NEXT_WEBHOOK_MERCADOPAGO_KEY_SECRET` configurado y rotación periódica.
  - Timeouts y reintentos a nivel de requests MP externos homogéneos (`withRetry`).

- **[Cloudinary]**
  - Validación previa de tamaño/MIME; antivirus opcional.
  - Uso de `upload_preset` firmado cuando corresponda; limitar transformaciones desde cliente.

- **[Resend]**
  - Configurar `RESEND_WEBHOOK_SECRET` si se consumen webhooks y validar firma.
  - Trazabilidad (tags) estandarizada y manejo de errores/bounces.

- **[Redis/Socket.IO]**
  - CORS estrictos; manejar reconexiones y degradar si Redis cae (usar in-memory fallback si es aceptable).
  - Monitorear latencia/adaptador Redis.

- **[Google Maps]**
  - Restringir API key por dominio y por API; cuotas y alertas.
  - Asegurar CSP compatible con dominios de Google Maps.
