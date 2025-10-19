# Criterios de aceptación (11-devops-deploy)

- **[Pipelines/CI]**
  - Workflow CI con pasos `lint`, `type-check`, `test:coverage` y `build`; gates de cobertura en 70% inicial.
  - Reportes publicados (LCOV, text-summary) y fallos bloquean merge.

- **[Estrategias]**
  - Definida estrategia de despliegue para Vercel (30s) y Docker/On-prem; tareas largas externalizadas a colas.

- **[Healthchecks]**
  - Endpoints `/health` y `/ready` documentados e integrados con orquestador.

- **[Config/Env]**
  - Fail-fast de envs en build + validador central runtime. Matriz de entornos completa.

- **[Seguridad]**
  - Secretos gestionados fuera del repo; rotación definida para valores expuestos por `deploy.sh`.
