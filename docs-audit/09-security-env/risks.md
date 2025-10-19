# Matriz de riesgos (09-security-env)

| ID  | Riesgo                                            | Evidencia                | Severidad | Impacto | Esfuerzo |
| --- | ------------------------------------------------- | ------------------------ | --------- | ------- | -------- |
| S1  | Build pasa con envs faltantes                     | `next.config.js` 17-25   | High      | Alto    | Bajo     |
| S2  | CSP permisiva con `'unsafe-inline'/'unsafe-eval'` | `next.config.js` 36-56   | High      | Alto    | Medio    |
| S3  | CORS potencialmente amplio en `/api`              | `next.config.js` 126-133 | Medium    | Alto    | Bajo     |
| S4  | Rate limiting ausente                             | Búsqueda sin resultados  | High      | Alto    | Medio    |
| S5  | Logs de request en middleware con PII             | `middleware.ts` 28-35    | Low       | Medio   | Bajo     |
