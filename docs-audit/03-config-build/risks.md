# Matriz de riesgos (severidad × impacto × esfuerzo)

| ID  | Riesgo                                                              | Evidencia                         | Severidad | Impacto | Esfuerzo |
| --- | ------------------------------------------------------------------- | --------------------------------- | --------- | ------- | -------- |
| R1  | CSP permite `'unsafe-inline'`/`'unsafe-eval'`                       | `next.config.js` 36-44            | Critical  | Alto    | Medio    |
| R2  | Build no falla si faltan env críticas                               | `next.config.js` 17-25            | High      | Alto    | Bajo     |
| R3  | CORS amplio (`*` en dev / configurable en prod)                     | `next.config.js` 60-65, 119-133   | High      | Alto    | Bajo     |
| R4  | Secretos en texto plano en Compose                                  | `docker-compose.yml` 8-14         | High      | Alto    | Bajo     |
| R5  | Puertos DB/Cache expuestos al host                                  | `docker-compose.yml` 20-29        | Medium    | Medio   | Bajo     |
| R6  | `allowJs` y `skipLibCheck` relajan chequeos                         | `tsconfig.json` 9-13              | Medium    | Medio   | Bajo     |
| R7  | Consola habilitada en ESLint                                        | `.eslintrc.json` 8                | Low       | Medio   | Bajo     |
| R8  | Alias de env duplicados (MP)                                        | `env.example` 36-43               | Low       | Bajo    | Bajo     |
| R9  | `maxDuration` único para todas las rutas                            | `vercel.json` 3-5                 | Medium    | Medio   | Bajo     |
| R10 | `serverExternalPackages`/ignore webpack desalineados con plataforma | `next.config.js` 138-159, 173-175 | Medium    | Medio   | Medio    |
| R11 | `moduleResolution: bundler` puede afectar tooling/IDE               | `tsconfig.json` 15                | Medium    | Medio   | Medio    |
| R12 | Exposición de `NEXT_PUBLIC_*` no auditada                           | `next.config.js` 99-107           | Medium    | Medio   | Bajo     |
