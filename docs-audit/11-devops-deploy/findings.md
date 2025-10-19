# Hallazgos (11-devops-deploy)

- **[Estrategias de deploy soportadas]**
  - Vercel: `vercel.json` limita `maxDuration: 30s` para `app/**/route.(ts|js)` (edge/serverless) → considerar timeouts para integraciones externas.
  - Docker: `docker-compose.yml` define `app`, `mongo`, `redis` para despliegue local/prod sencillo (puerto 3000, Mongo/Redis expuestos).
  - Servidor propio: `deploy.sh` automatiza provisión (Node, PM2, Apache como reverse-proxy, Certbot opcional), genera `.env.production` y corre `next build`/PM2.

- **[Riesgos en deploy.sh]**
  - Es interactivo y escribe `.env.production` con secretos explícitos (tokens y claves de MP, Cloudinary, Resend). Requiere almacenar fuera del repo y rotación.
  - Configura Apache con HTTP→HTTPS y headers básicos; sin healthchecks explícitos.

- **[Healthchecks]**
  - No se observa endpoint `/health`/`/ready` estándar. `server.js` inicia Next con HTTP/HTTPS local, maneja shutdown con SIGTERM/SIGINT, pero sin healthcheck.

- **[Build/Runtime]**
  - `next.config.js` valida envs en build con `console.warn` (no fail-fast). CORS/headers definidos. Imagen/optimizaciones.

- **[Tiempo máx ejecución]**
  - En Vercel, las rutas `app/**/route.ts` tienen `maxDuration: 30`. Operaciones largas deben desplazarse a colas/background o usar server pers.
