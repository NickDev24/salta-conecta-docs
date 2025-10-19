# Checklist (11-devops-deploy)

- **[Pipelines/CI]**
  - Pasos: `install` → `lint` → `type-check` → `test:coverage` → `build`.
  - Publicar cobertura (`lcov`, `text-summary`). Gates con umbral ≥70% (progresivo a 80/90).
  - Variables de entorno por entorno (secrets store del CI), sin secretos en logs.

- **[Estrategias]**
  - Vercel: validar `maxDuration` y cold starts; functions críticas optimizadas; uso de colas para tareas >30s.
  - Docker: healthchecks para `app`, `mongo`, `redis`. No exponer puertos a Internet sin firewall.
  - Servidor propio: PM2 + reverse-proxy (Apache/Nginx) con SSL y headers; rotación de logs.

- **[Healthchecks]**
  - Endpoint `/health` y `/ready` con checks de DB/Redis y versión.
  - Integrar healthchecks en orquestador (Docker Compose/Swarm/K8s/Vercel monitors).

- **[Config/Env]**
  - Fail-fast de envs en build y validador central runtime.
  - Matriz de entornos con variables y orígenes (secrets manager).

- **[Observabilidad]**
  - Logs estructurados, métricas básicas (latencia, error rate), alertas en 5xx y picos de latencia.
