# Matriz de riesgos (06-frontend-middleware)

| ID  | Riesgo                                               | Evidencia                                   | Severidad | Impacto | Esfuerzo |
| --- | ---------------------------------------------------- | ------------------------------------------- | --------- | ------- | -------- |
| F1  | RBAC por prefijo puede dejar huecos si cambian rutas | `middleware.ts` 86-113                      | Medium    | Alto    | Bajo     |
| F2  | Bloqueo de bots solo por UA es evadible              | `middleware.ts` 70-82                       | Medium    | Medio   | Bajo     |
| F3  | Redirección HTTPS depende de `x-forwarded-proto`     | `middleware.ts` 64-68                       | Medium    | Alto    | Bajo     |
| F4  | `X-XSS-Protection` obsoleto, foco debe ser CSP       | `middleware.ts` 44; CSP en `next.config.js` | Low       | Medio   | Bajo     |
| F5  | Log de cada request puede generar alto volumen/PII   | `middleware.ts` 28-35                       | Low       | Medio   | Bajo     |
