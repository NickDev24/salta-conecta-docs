# Hallazgos (09-security-env)

- **[Validación de envs en build débil]**
  - `next.config.js` verifica variables requeridas en producción pero solo hace `console.warn` y continúa el build (líneas 17-25). Riesgo de fallar en runtime.

- **[Exposición controlada de envs al cliente]**
  - `next.config.js` reexpone únicamente claves `NEXT_PUBLIC_*` (líneas 99-107). Correcto, pero depende de configuración segura de `.env`.

- **[Headers de seguridad presentes]**
  - `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, `X-DNS-Prefetch-Control` y CSP configurados globalmente (líneas 27-58; 112-135). Middleware añade adicionales y `x-nonce` (ver `middleware.ts`).

- **[CSP permisiva]**
  - `script-src` incluye `'unsafe-inline' 'unsafe-eval'` y dominios amplios (Google/Mercado Pago) (líneas 36-56). Incrementa superficie de XSS; `nonce` está disponible en middleware.

- **[CORS amplio en dev y configurable en prod]**
  - Para `/api` usa `NEXT_PUBLIC_BASE_URL || NEXT_PUBLIC_APP_URL || '*'` (líneas 126-133). En rutas generales, `Access-Control-Allow-Origin` toma `NEXT_PUBLIC_APP_URL || localhost` (líneas 118-123).

- **[HTTPS/HSTS y mitigación en edge]**
  - `middleware.ts` fuerza HTTPS en producción y aplica HSTS si corresponde; redirige cuando `x-forwarded-proto !== 'https'`.

- **[Rate limiting ausente]**
  - No se encontró utilidad de rate limiting en el proyecto (grep sin resultados). Riesgo para endpoints públicos y webhooks.

- **[Secretos server-only vs client]**
  - Secretos críticos se usan solo en server (p. ej., `MONGODB_URI` en `lib/mongodb.ts`, `MERCADOPAGO_ACCESS_TOKEN` en `lib/mercadopago.ts`, `RESEND_API_KEY` en `lib/resend.ts`, `CLOUDINARY_API_SECRET` en `lib/cloudinary.ts`). Correcto.

- **[Fail-fast de runtime parcial]**
  - `lib/mongodb.ts` lanza si falta `MONGODB_URI`. Otras integraciones validan en helpers (`isEmailEnabled`, `isCloudinaryConfigured`). Falta un validador central de runtime.
