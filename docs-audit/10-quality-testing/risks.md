# Matriz de riesgos (10-quality-testing)

| ID  | Riesgo                                                 | Evidencia                                                                   | Severidad | Impacto | Esfuerzo |
| --- | ------------------------------------------------------ | --------------------------------------------------------------------------- | --------- | ------- | -------- |
| T1  | Cobertura real < umbral deseado                        | `jest.config.js` coverageThreshold=70%                                      | Medium    | Medio   | Bajo     |
| T2  | Falta de reporter cobertura para CI                    | `package.json` (sin jest reporters configurados)                            | Low       | Bajo    | Bajo     |
| T3  | Tests de rutas críticas insuficientes (webhooks/pagos) | Estructura `__tests__/` no incluye MP webhook                               | High      | Alto    | Medio    |
| T4  | Flaky en integración por dependencias externas         | `scheduler.integration.test.ts` usa memory server; OK pero sin aislar redes | Low       | Medio   | Medio    |
| T5  | ESLint no falla build si `lint` no corre en CI         | Scripts disponibles, falta verificación en pipeline                         | Medium    | Medio   | Bajo     |
