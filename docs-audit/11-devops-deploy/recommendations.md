# Recomendaciones (11-devops-deploy)

- **[Pipelines/CI]**
  - Workflow con jobs: install → lint → type-check → test:coverage → build → (optional) deploy.
  - Publicar cobertura (LCOV) y tiempos por test. Gates con umbral progresivo (70%→80%→90%).
  - Inyectar envs desde secret manager del CI; nunca escribir secretos en repo.

- **[Estrategia Vercel]**
  - Revisar rutas que pueden exceder `30s`; mover a colas/tasks o Serverless Functions con streaming si aplica.
  - Configurar observabilidad (logs/alerts) y variables por entorno (Preview/Prod).

- **[Estrategia Docker/On-prem]**
  - Añadir healthchecks y política de reinicio (`restart: unless-stopped`).
  - Reverse-proxy con Nginx (o mantener Apache) y certificados gestionados automáticamente.
  - Externalizar `.env.production` a variables de entorno/secret store. Rotar claves expuestas por `deploy.sh`.

- **[Healthchecks]**
  - Implementar `/health` (liveness) y `/ready` (readiness) que verifiquen Mongo/Redis y devuelvan versión.
  - Integrar healthchecks a `docker-compose.yml` y a monitores (Vercel/uptime).

- **[Config/Env]**
  - Fail-fast de envs en build + validador runtime central.
  - Matriz de entornos completa con variables obligatorias/opcionales y orígenes (secret manager/CI/CD).

- **[Seguridad/Compliance]**
  - Rotación de secretos mencionados en `deploy.sh`. Auditoría de acceso a servidores y registros.
