# Criterios de aceptación (07-integrations)

- **[MercadoPago]**
  - Idempotencia aplicada también en reintentos de pago; webhook con validación de firma y registro de eventos.
  - Claves configuradas y segregadas (server vs client); timeouts y reintentos definidos.

- **[Cloudinary]**
  - Validación previa de archivos; secretos no expuestos al cliente; presets/transformaciones bajo control.

- **[Resend]**
  - Claves presentes (`RESEND_API_KEY`, `RESEND_FROM`); webhooks firmados si se usan.

- **[Redis/Socket.IO]**
  - CORS restringido a origen válido; reconexión/backoff y fallback definidos.

- **[Google Maps]**
  - API key restringida por dominio y CSP compatible.

- **[env-matrix.csv]**
  - Matriz actualizada con todas las claves por integración y referencias de uso.
