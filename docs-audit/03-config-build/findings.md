# Hallazgos de configuración y build

- **[CSP permisiva con inline/eval]** en `next.config.js` líneas 36-56: `script-src` incluye `'unsafe-inline' 'unsafe-eval'` y dominios amplios (Google/Mercado Pago). Riesgo de XSS aumentado.
- **[Validación de variables en build solo advierte]** en `next.config.js` líneas 17-25: si faltan obligatorias en producción, solo hace `console.warn` y continúa el build.
- **[CORS amplio]** en `next.config.js` líneas 60-65 y 119-133: en dev `globalCorsOrigin='*'`; para `/api` usa `NEXT_PUBLIC_*` o `*`. Riesgo de exposición si mal configurado.
- **[Headers de seguridad presentes]** en `next.config.js` líneas 27-58: añade `X-Frame-Options`, `X-Content-Type-Options`, `X-XSS-Protection`, `Referrer-Policy`, `Permissions-Policy` y CSP. Buen baseline.
- **[Exposición de env al cliente]** en `next.config.js` líneas 99-107: reexpone `NEXT_PUBLIC_*` hacia el browser. Correcto, pero requiere control de qué claves son públicas.
- **[Webpack ignore en cliente]** en `next.config.js` líneas 138-159: ignora módulos de server en cliente (e.g., `mongodb`, `ioredis`). Previene bundles rotos.
- **[Experimental y serverExternalPackages]** en `next.config.js` líneas 161-175: varias flags; se externalizan `mongoose`, `ioredis`, `winston` en server. Alinear con plataforma de despliegue.
- **[tsconfig estricto pero con allowJs y skipLibCheck]** en `tsconfig.json` líneas 9-13: `allowJs: true`, `skipLibCheck: true`. Estricto global activo (`strict: true`). `moduleResolution: bundler` puede afectar tooling.
- **[Aliases amplios]** en `tsconfig.json` líneas 25-71: múltiples `paths` (`@/*` y específicos). Requiere consistencia con Jest y TS path mapper.
- **[ESLint baseline Next]** en `.eslintrc.json` líneas 2-12: extiende `next/core-web-vitals`; `no-console` desactivado, `no-debugger` en `warn`. Puede ocultar problemas en prod.
- **[Jest configurado con cobertura 70%]** en `jest.config.js` líneas 41-47. Ignora utilidades específicas en `__tests__/utils` (líneas 24-29). `testTimeout` 10s (línea 50).
- **[Docker Compose producción simplificado]** en `docker-compose.yml` líneas 8-14: define secretos en texto plano (MP access token, NEXTAUTH_SECRET). Usa `NODE_ENV=production`. Sin límites de recursos ni healthchecks.
- **[Servicios Mongo/Redis con volúmenes]** en `docker-compose.yml` líneas 22-34: persiste datos; expone puertos 27017/6379 a host.
- **[Vercel funciones with maxDuration]** en `vercel.json` líneas 3-5: `app/**/route.{ts,js}` con `maxDuration: 30`. Sin env/overrides adicionales.
- **[Variables env dispersas]** en `.env.example` y `env.example`: claves duplicadas/alias (e.g., `MERCADOPAGO_ACCESS_TOKEN` y `MP_ACCESS_TOKEN`, `MERCADOPAGO_APPLICATION_ID` y `MP_APPLICATION_ID`). Necesario consolidar.
