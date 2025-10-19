# Checklist de verificación (04-api)

- **[Auth]** Cada endpoint sensible requiere sesión: `getServerSession(...)` o middleware (`requireAdmin/requireRole`).
- **[Roles]** Controles explícitos por rol (admin/driver/supplier/client) con helpers centralizados.
- **[Validación]** Entradas validadas con `zod` (`safeParse`) y respuesta 400 con issues; sin validación manual dispersa.
- **[Errores]** `try/catch` consistente, mensajes neutrales y `status` correctos (401/403/404/400/500).
- **[Idempotencia]** Webhooks/pagos con locks y logs; evitar duplicados (e.g., reintentos de `retry-payment`).
- **[Servicios externos]** Manejo de timeouts/reintentos y errores para MP/Resend/Cloudinary/Redis.
- **[Auditoría]** Registrar eventos sensibles (cambios de estado, entregas) en `audit`/logs.
- **[Rate limiting]** Endpoints públicos/webhooks con protección de tasa.
- **[CORS]** Rutas con CORS adecuado; sin `*` en prod.
- **[Security headers]** Confirmar headers en respuestas sensibles si aplican.
