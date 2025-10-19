# Hallazgos (06-frontend-middleware)

- **[RBAC en middleware]** `middleware.ts` protege rutas por prefijo: `'/admin'` y `'/provider'` mediante `next-auth/jwt.getToken()` y roles de `USER_ROLES` (líneas 86-113). Redirige a `/auth/login` si no autorizado.
- **[HTTPS y HSTS]** En producción fuerza HTTPS y aplica HSTS cuando el host no es local/privado (líneas 53-68 y 60-62). Usa `x-forwarded-proto` para detectar esquema tras proxy/CDN.
- **[Cabeceras de seguridad]** Establece `X-Content-Type-Options`, `X-Frame-Options`, `X-XSS-Protection`, `Referrer-Policy`, `Permissions-Policy`, `X-DNS-Prefetch-Control` (líneas 41-49). Nonce se agrega como `x-nonce` (líneas 38-51) para uso en páginas.
- **[Bloqueo de bots]** Filtra user-agents conocidos (Ahrefs/Semrush/Yandex/etc.) y devuelve 403 (líneas 70-82). Registra eventos con `logger` (líneas 28-35, 80-81).
- **[Scope de middleware]** Excluye `api`, `_next/static`, `_next/image`, `favicon.ico`, `public/` (líneas 6-17). Abarca la app entera (SSR/CSR) fuera de esos paths.
- **[SSR/CSR y caché]** No se observan políticas de caché específicas en middleware. La CSP y headers generales están en `next.config.js` (seguridad y CORS). Caché de páginas depende de la configuración por ruta y uso de `fetch`/`revalidate` (no centralizado en middleware).
- **[Compatibilidad Edge]** Usa `next-auth/jwt` en middleware (Edge-compatible) con secreto `NEXTAUTH_SECRET|AUTH_SECRET` (líneas 92-99), correcto para evitar dependencias de Node APIs.
- **[Observabilidad]** `logger.info` de cada request (método, path, IP, UA, host). Importante revisar impacto de performance/log volume.
- **[Riesgos menores]**
  - Bloqueo de bots basado solo en UA es evadible; considerar `rate limit` y patrones de comportamiento.
  - `X-XSS-Protection` es obsoleto en Chromium; CSP estricta (en `next.config.js`) es más efectiva.
  - Redirecciones a login no distinguen entre 401/403 en páginas; UX podría mejorar con página de acceso denegado.
