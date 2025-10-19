# Implementation Plan — SaltaConecta

## Resumen Ejecutivo

- **Estado actual**: La base técnica es sólida (Next.js 15 + TS, modelos e índices Mongo maduros, middleware con RBAC y headers de seguridad, integraciones principales operativas). La auditoría detecta brechas de seguridad de configuración (CSP permisiva, CORS flexible, build sin fail-fast), faltante de controles anti-abuso (rate limit) y puntos de idempotencia/QA por cubrir.
- **Riesgos críticos**: CSP con `'unsafe-inline'/'unsafe-eval'` y build que no falla con envs faltantes (`next.config.js`), ausencia de rate limiting, endpoints sensibles sin auth uniforme (`drivers/connect`), secretos gestionados en texto por `deploy.sh`, falta de healthchecks estándar.
- **Potencial**: Al completar F1–F4, el sistema queda listo para producción con mayor resiliencia, seguridad y gobernanza operativa: menor superficie XSS, menos fallas por configuración, pagos sin duplicados, despliegues predecibles, métricas y calidad bajo CI.

---

## Fase 1 – Seguridad y Entorno (P0)

- **Objetivos**
  - Fail-fast de variables en build + validador de entorno en runtime.
  - Endurecer CSP con `nonce` y reducir dominios/`'unsafe-inline'/'unsafe-eval'`.
  - CORS restrictivo por entorno y rate limiting en `/api` y webhooks.
- **Módulos afectados**
  - `03-config-build`, `09-security-env`, `06-frontend-middleware`, `07-integrations`.
- **Tareas**
  - Fail-fast en `next.config.js` o prebuild script para envs críticas. Validador central (server y `NEXT_PUBLIC_*`).
  - Ajuste CSP: quitar `'unsafe-inline'/'unsafe-eval'` cuando sea viable; usar `x-nonce` del `middleware.ts`. Limitar `script-src`, `connect-src`, `frame-src` a dominios mínimos (Google Maps, Mercado Pago, Resend, Cloudinary).
  - CORS: `ALLOWED_ORIGINS` por env, sin `*` en producción.
  - Rate limit: `rate-limiter-flexible` o equivalente para `/api/**` y webhooks de MP/Resend.
- **Riesgos y dependencias**
  - Riesgo de romper SDKs que dependan de inline scripts (MP/Maps). Coordinar con Frontend.
  - Dependencia de matrices de entorno (`02-inventory/env-matrix.csv`).
- **Esfuerzo estimado**: Alto para CSP (ajustes y pruebas); Bajo–Medio para fail-fast/CORS/ratelimit.
- **Criterios de finalización**
  - Build falla sin envs obligatorias; validador runtime con mensajes claros.
  - CSP sin `'unsafe-inline'/'unsafe-eval'` o con `nonce` aplicado y lista de dominios estricta.
  - CORS sin comodines en prod; rate limiting activo con métricas.
- **Beneficios esperados**
  - Reducción de XSS y fallas por configuración. Mitigación de abuso/DDoS.

---

## Fase 2 – Backend y Modelos (P1)

- **Objetivos**
  - Idempotencia en reintentos de pago y auth/roles estandarizados.
  - Resiliencia conexión Mongo y housekeeping (TTL).
- **Módulos afectados**
  - `04-api`, `05-data-models`, `07-integrations`.
- **Tareas**
  - `orders/[id]/retry-payment`: bloquear multi-click, registrar intentos, validar estado previo.
  - Helper `requireRole()` para roles y uso uniforme de `zod` en endpoints con payload.
  - Mongo: `serverSelectionTimeoutMS`, `socketTimeoutMS`, pool y `retryWrites`; `autoIndex=false` en prod + TTL en `PaymentLog` (~90 días).
- **Riesgos y dependencias**
  - Cambios en flujos de pago y UX de reintentos. Coordinar QA.
  - Dependencia de F1 (rate limit/CSP) para coherencia de seguridad.
- **Esfuerzo estimado**: Medio.
- **Criterios de finalización**
  - Reintentos no generan preferencias duplicadas; logs de idempotencia.
  - Endpoints con `zod` y roles centralizados; auditoría mínima de acciones.
  - Conexión Mongo con timeouts/pooling configurados; TTL activo.
- **Beneficios esperados**
  - Pagos confiables, menor deuda técnica y menos fallas por red/DB.

---

## Fase 3 – Frontend y Experiencia de Usuario (P1)

- **Objetivos**
  - RBAC UX (403 dedicado) y accesibilidad esencial (A11y).
  - SSR-safety y performance ligera en contextos/tiendas.
- **Módulos afectados**
  - `06-frontend-middleware`, `08-ui-state`.
- **Tareas**
  - Página 403 (acceso denegado) + diferenciación 401/403 desde `middleware.ts`.
  - A11y: `aria-label` en icon-only, `role="progressbar"` en barras, `aria-live` en chat/notifs.
  - Documentar consumo de stores `zustand` en Client Components; memoizar derivados O(n) si listas >50.
- **Riesgos y dependencias**
  - Coordinación con diseño/UI. Sin dependencia crítica de F2.
- **Esfuerzo estimado**: Bajo–Medio.
- **Criterios de finalización**
  - Flujos de acceso muestran 403 donde corresponde.
  - Lint de A11y pasa en componentes auditados; mejoras aplicadas.
- **Beneficios esperados**
  - Mejor UX, accesibilidad y claridad de permisos.

---

## Fase 4 – DevOps y Optimización Final (P1–P2)

- **Objetivos**
  - CI/CD con gates de calidad, healthchecks y gestión segura de secretos.
  - Optimización de despliegues (Vercel/Docker) y observabilidad.
- **Módulos afectados**
  - `10-quality-testing`, `11-devops-deploy`, `03-config-build`.
- **Tareas**
  - CI: jobs `lint` + `type-check` + `test:coverage` con reporters `lcov`/`text-summary`; gates 70%→80%→90%.
  - Healthchecks `/health` y `/ready` (Mongo/Redis) y wiring en Docker (`healthcheck`, `restart: unless-stopped`).
  - Secretos: externalizar `.env.production` a Secret Manager del CI y del entorno; rotar claves expuestas por `deploy.sh`.
  - Vercel: revisar rutas >30s y mover a background/colas.
  - Observabilidad: muestreo y anonimización de IP/UA en logs; alertas básicas.
- **Riesgos y dependencias**
  - Cambios en pipelines y permisos del repositorio/plataforma.
- **Esfuerzo estimado**: Medio.
- **Criterios de finalización**
  - Pipeline verde con gates activos; reporters publicados.
  - Healthchecks operativos y monitoreados; secretos fuera de scripts.
  - Sin timeouts en rutas críticas en Vercel.
- **Beneficios esperados**
  - Despliegues previsibles, menor MTTR y cumplimiento de seguridad operativa.

---

## Cronograma sugerido (sprints)

| Fase                     | Duración    | Responsable             | Dependencias | Resultado                                                             | Impacto |
| ------------------------ | ----------- | ----------------------- | ------------ | --------------------------------------------------------------------- | ------- |
| F1 – Seguridad y Entorno | 2–3 sprints | Tech Lead + Sec + FE/BE | Ninguna      | CSP endurecida, fail-fast env, CORS estricto, rate limit              | Alto    |
| F2 – Backend y Modelos   | 2 sprints   | BE Lead + QA            | F1           | Idempotencia pagos, roles/validación estandarizados, Mongo resiliente | Alto    |
| F3 – Frontend y UX       | 1 sprint    | FE Lead + Diseño        | F1 (parcial) | 403 UX, A11y esencial, SSR-safety documentada                         | Medio   |
| F4 – DevOps/Optimización | 2 sprints   | DevOps + QA             | F1           | CI con gates, healthchecks, secretos gestionados, observabilidad      | Alto    |

- **Paralelización**
  - F3 puede iniciar en paralelo a F2 tras completar ajustes base de F1 (CSP/CORS/ratelimit) que afecten UI.
  - Parte de F4 (pipeline CI) puede arrancar en el último sprint de F1.

---

## Criterios de Aceptación por Fase

- **F1**
  - PRs con cambios de `next.config.js` y/o scripts de validación. Reporte de dominios CSP efectivos y verificación con `Report-Only` previo. Pruebas de CORS y ratelimit con casos límite.
- **F2**
  - Suite de pruebas para `retry-payment` con reintentos rápidos; verificación de no duplicación. Endpoints auditados usan `zod`/`requireRole`.
- **F3**
  - Acceso denegado retorna 403; A11y validada (axe básico) en componentes modificados.
- **F4**
  - CI falla si no se cumple cobertura y lint/type-check. `/health` y `/ready` integrados en Docker y monitoreo. Secretos fuera de scripts y rotados.

---

## Mapa de Dependencias (alto nivel)

- `F1` → habilita seguridad base que afecta UI y APIs (CSP/CORS/ratelimit).
- `F2` → depende de `F1` para coherencia de seguridad; toca pagos e integraciones.
- `F3` → puede correr en paralelo con `F2` tras base de `F1`.
- `F4` → puede iniciar con CI tras `F1` y consolidarse al final.

---

## Anexo de Riesgos Estratégicos y Mitigación

- **CSP permisiva** (`03-config-build`, `09-security-env`): endurecer con `nonce`, reducir dominios. Validar SDKs (MP/Maps) y usar `strict-dynamic` si aplica.
- **Build sin fail-fast/validador** (`03-config-build`, `09-security-env`): fail temprano + validador runtime con Zod/envsafe.
- **Rate limiting ausente** (`09-security-env`): implementar en `/api` y webhooks; monitorear rechazos y ajustar umbrales.
- **Endpoint sin auth** (`04-api` `drivers/connect`): exigir sesión y rol `driver` o token de servicio; auditar cambios de estado.
- **Idempotencia pagos incompleta** (`04-api`, `07-integrations`): bloquear reintentos y registrar; extender patrón del webhook.
- **Conexión Mongo sin timeouts** (`05-data-models`): configurar timeouts/pool/retries; `autoIndex=false` en prod.
- **Secretos en scripts** (`11-devops-deploy`): externalizar secretos a Secret Manager; rotación inmediata.
- **Falta de healthchecks** (`11-devops-deploy`): implementar `/health` y `/ready` (Mongo/Redis) y configurar en Docker/monitoring.
- **CORS flexible** (`03-config-build`): orígenes explícitos por entorno; evitar `*` en prod.
- **Volumen de logs/PII** (`06-frontend-middleware`, `09-security-env`): muestreo y anonimización de IP/UA; alertas.

---

## Referencias y Trazabilidad

- Resumen final: `docs-audit/99-roadmap/00-overview.md`.
- Plan de remediación detallado: `docs-audit/99-roadmap/remediation-plan.md`.
- Matrices de inventario: `docs-audit/02-inventory/`.
- Evidencias por módulo: `docs-audit/03–11/*` (`findings.md`, `risks.md`, `recommendations.md`).
