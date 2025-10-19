# Remediation Plan (Prioritized)

| ID    | Tarea                                                                               | Prioridad | Costo (días) | Riesgos                                                  | Dependencias                                                           | Impacto esperado                                   |
| ----- | ----------------------------------------------------------------------------------- | --------- | ------------ | -------------------------------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------------- |
| RP-01 | Fail-fast de envs en build y validador central runtime (server y `NEXT_PUBLIC_*`)   | P0        | 1-2          | Falsos positivos si variables opcionales no distinguidas | `docs-audit/09-security-env/`                                          | Reduce fallas en prod por env faltantes; seguridad |
| RP-02 | Endurecer CSP con nonce (remover `'unsafe-inline'/'unsafe-eval'` donde sea posible) | P0        | 2-4          | Posibles roturas de scripts inline                       | `docs-audit/06-frontend-middleware/`, `09-security-env/`               | Disminuye superficie XSS                           |
| RP-03 | Implementar rate limiting en `/api` y webhooks (IP/UA)                              | P0        | 1-2          | Bloqueo a usuarios legítimos si umbrales mal calibrados  | `docs-audit/09-security-env/`                                          | Mitiga abuso/DDoS y consumo excesivo               |
| RP-04 | CORS restrictivo en `/api` (sin `*` en prod)                                        | P0        | 0.5          | Integraciones externas rotas si orígenes no incluidos    | `docs-audit/09-security-env/`                                          | Reduce riesgos de exfiltración                     |
| RP-05 | Página 403 y RBAC middleware con flujo diferenciado (403 vs login)                  | P1        | 1            | Cambios de UX mínimos                                    | `docs-audit/06-frontend-middleware/`                                   | Mejora control de acceso                           |
| RP-06 | Idempotencia en `orders/[id]/retry-payment` (bloqueo/registro)                      | P1        | 1-2          | Complejidad con reintentos cliente                       | `docs-audit/07-integrations/`, `04-api/`                               | Evita pagos duplicados                             |
| RP-07 | Healthchecks `/health` y `/ready` + checks Mongo/Redis                              | P1        | 1            | Ninguno significativo                                    | `docs-audit/11-devops-deploy/`                                         | Mejor despliegue/auto-reinicio                     |
| RP-08 | CI pipeline: lint, type-check, test:coverage con gates y reportes                   | P1        | 1-2          | Tiempo de CI mayor                                       | `docs-audit/10-quality-testing/`, `11-devops-deploy/`                  | Calidad y regresiones controladas                  |
| RP-09 | A11y: aria-label en icon-only, progress ARIA, aria-live en chat/notifs              | P1        | 1            | Bajo                                                     | `docs-audit/08-ui-state/`                                              | Accesibilidad mejorada                             |
| RP-10 | Validaciones de archivo (tipo/tamaño) previas a Cloudinary                          | P1        | 1            | Rechazo de archivos borderline                           | `docs-audit/07-integrations/`                                          | Seguridad/perf                                     |
| RP-11 | Matriz de entornos y secreto en Secret Manager (no `.env` en scripts)               | P1        | 1            | Coordinación DevOps                                      | `docs-audit/11-devops-deploy/`                                         | Menor riesgo de filtraciones                       |
| RP-12 | Tests críticos: MP webhook (firma/idempotencia), middleware RBAC                    | P2        | 2-3          | Mocks complejos                                          | `docs-audit/10-quality-testing/`, `04-api/`, `06-frontend-middleware/` | Confianza en flujos críticos                       |
| RP-13 | Cobertura progresiva 70→80→90% (módulos críticos ≥85%)                              | P2        | 2-4          | Esfuerzo sostenido                                       | `docs-audit/10-quality-testing/`                                       | Calidad continua                                   |
| RP-14 | Observabilidad: muestreo de logs y anonimización IP/UA                              | P2        | 1-2          | Menor detalle en diagnósticos                            | `docs-audit/06-frontend-middleware/`, `09-security-env/`               | Compliance y costo logs                            |

## Secuencia sugerida

- P0 primero (seguridad/estabilidad), luego P1 (operativo/calidad), por último P2 (madurez).

## Notas

- Mantener una rama `audit-docs-only` y PRs por módulo.
- No mezclar cambios de código con documentación.
