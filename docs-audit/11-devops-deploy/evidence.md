# Evidencias (11-devops-deploy)

- **Vercel – `vercel.json`**

```json
{
  "version": 2,
  "functions": {
    "app/**/route.ts": { "maxDuration": 30 },
    "app/**/route.js": { "maxDuration": 30 }
  }
}
```

- **Docker Compose – `docker-compose.yml`**

```yaml
services:
  app:
    build: .
    ports:
      - '3000:3000'
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongo:27017/saltaconecta
      - REDIS_URL=redis://redis:6379
---
mongo:
  image: mongo:7.0
redis:
  image: redis:7.2-alpine
```

- **Script despliegue propio – `deploy.sh`**

```bash
118 cat > .env.production <<'ENV'
# ...
# Múltiples secretos de MP/Resend/Cloudinary en texto (para entorno local/prod)
# ...
284 npm run build
288 pm2 start ecosystem.config.js --env production
```

- **Servidor Node – `server.js`**

```js
11 const useHttps = process.env.USE_HTTPS !== 'false';
31 if (useHttps) { /* HTTPS server con certs locales */ } else { /* HTTP */ }
62 process.on('SIGTERM', () => { server.close(...) })
```

- **Headers/CSP – `next.config.js`**

```js
27 const securityHeaders = [ /* CSP y headers */ ];
112 async headers() { return [{ source: '/(.*)', headers: [...securityHeaders, { 'Access-Control-Allow-Origin': process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000' }] }, { source: '/api/:path*', headers: [{ 'Access-Control-Allow-Origin': process.env.NEXT_PUBLIC_BASE_URL || process.env.NEXT_PUBLIC_APP_URL || '*' }] }] }
```
