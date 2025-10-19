# Auditoría Salta Conecta – Overview Final

## Alcance y estado

- **Módulos cubiertos**: 01–11 completos.
- **Entregables por módulo**: findings, evidence, checklist, risks, recommendations, acceptance-criteria. Extras según módulo (matrices, umbrales, flujos).
- **Ubicación**: `docs-audit/` (sin cambios fuera de documentación).

## Síntesis ejecutiva

- **Seguridad**: CSP permisiva y validación de envs en build débil; sin rate limiting. RBAC en middleware por prefijo correcto pero mejorable (403 vs login).
- **APIs/Integraciones**: Webhook MP con idempotencia robusta; riesgo de preferencias duplicadas en `retry-payment`. Secretos segregados server-side.
- **Datos**: Modelos con índices relevantes; TTL opcional en logs. Conexión Mongo clara.
- **Frontend/Middleware**: Headers y HSTS; bloqueo básico de bots; nonce disponible. A11y con oportunidades en icon-only y progreso.
- **Calidad/Testing**: Jest + Testing Library; umbral 70% global; integración con Mongo en memoria; falta suite para webhooks/RBAC.
- **DevOps**: Estrategias Vercel/Docker/On-prem; script `deploy.sh` escribe `.env.production` con secretos (rotar y externalizar). Faltan healthchecks estándar.

## Próximos pasos (priorizado)

- **P0**
  - Fail-fast envs (build) + validador runtime central (server y `NEXT_PUBLIC_*`).
  - Endurecer CSP con nonce (eliminar `'unsafe-inline'/'unsafe-eval'` donde sea viable).
  - Rate limiting en `/api` y webhooks; CORS restrictivo.
- **P1**
  - Healthchecks `/health` y `/ready` (Mongo/Redis).
  - Idempotencia en `orders/[id]/retry-payment` y página 403 para RBAC.
  - CI: `lint` + `type-check` + `test:coverage` con gates; publicar cobertura.
- **P2**
  - Cobertura progresiva 80%→90%; tests críticos (MP webhook, middleware RBAC).
  - Observabilidad: muestreo de logs, anonimización IP/UA.

## Trazabilidad

- Índice de hallazgos y evidencias: `docs-audit/99-roadmap/change-log.md`.
- Plan de remediación: `docs-audit/99-roadmap/remediation-plan.md`.

## Reglas operativas

- Trabajar en rama `audit-docs-only`.
- 1 PR por módulo, solo archivos dentro de `docs-audit/`.
- (Opcional) pre-commit local que bloquee cambios fuera de `docs-audit/`.
