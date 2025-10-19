# Criterios de aceptación (04-api)

- **[Auth y roles]**
  - Todos los endpoints que cambian estado requieren sesión válida.
  - Roles verificados mediante helper central (`requireRole/requireAdmin`) con respuestas 401/403 consistentes.

- **[Validación Zod]**
  - Entrada validada con `zod` (body/params/query). Errores 400 incluyen `issues`.

- **[Idempotencia]**
  - Webhooks y pagos implementan locks/logs anti-duplicado. Reintentos de pago no generan preferencias duplicadas.

- **[Errores estándar]**
  - Respuestas con estructura establecida y códigos correctos. Sin filtración de errores internos.

- **[Servicios externos]**
  - Timeouts y reintentos definidos. Errores externos manejados con degradación controlada.

- **[Observabilidad]**
  - Acciones sensibles registradas (audit trail). Métricas básicas disponibles.

- **[Rate limiting]**
  - Endpoints públicos/webhooks protegidos contra abuso.

- **[CSV actualizado]**
  - `docs-audit/02-inventory/endpoints-matrix.csv` contiene filas para endpoints críticos y está listo para ampliación total.
