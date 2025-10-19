# Criterios de aceptación (10-quality-testing)

- **[Cobertura]**
  - Umbral global inicial ≥70% en CI con reporte `text-summary` y `lcov` publicado como artefacto.
  - Plan acordado para elevar a 80% (P1) y 90% (P2) con módulos críticos ≥85% líneas/branches.

- **[Suite]**
  - Tests para rutas críticas: `app/api/mercadopago/webhook` (firma/duplicados), RBAC middleware (admin/provider), y estados de error en stores/contexts.
  - Integración con `mongodb-memory-server` establecida y documentada; unit tests sin dependencias externas.

- **[Calidad]**
  - `npm run lint` y `npm run type-check` ejecutados en CI y bloquean merges si fallan.
  - Tiempos: unitarios <10s, integración con timeouts configurados y sin sleeps arbitrarios.
