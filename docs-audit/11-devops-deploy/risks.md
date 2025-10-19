# Matriz de riesgos (11-devops-deploy)

| ID  | Riesgo                                                     | Evidencia                              | Severidad | Impacto | Esfuerzo |
| --- | ---------------------------------------------------------- | -------------------------------------- | --------- | ------- | -------- |
| D1  | Secretos en texto en `.env.production` generado por script | `deploy.sh` 118-242                    | High      | Alto    | Bajo     |
| D2  | Falta de healthcheck `/health`/`/ready`                    | Proyecto carece de endpoints dedicados | Medium    | Alto    | Bajo     |
| D3  | `maxDuration` 30s limita tareas largas en Vercel           | `vercel.json` 4-5                      | Medium    | Medio   | Medio    |
| D4  | `docker-compose.yml` sin healthcheck ni reinicios          | `docker-compose.yml`                   | Medium    | Medio   | Bajo     |
| D5  | Build no falla con envs faltantes                          | `next.config.js` 17-25                 | High      | Alto    | Bajo     |
