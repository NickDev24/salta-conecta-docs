# Criterios de aceptación (06-frontend-middleware)

- **[RBAC]**
  - Prefijos protegidos cubren todas las rutas sensibles; helper central de roles y página 403 implementada.
  - Redirecciones a login solo cuando no autenticado; 403 cuando autenticado sin permisos.

- **[HTTPS/HSTS]**
  - Forzado HTTPS en producción validado en plataforma (headers `x-forwarded-*`). HSTS activo y sin interferir con entornos locales.

- **[Headers]**
  - CSP estricta en `next.config.js`; `x-nonce` utilizado donde hay inline scripts.

- **[Bots y abuso]**
  - UA-block complementado con rate limiting por IP/UA y estrategias anti-scraping.

- **[Cache/SSR]**
  - Estrategia por ruta definida (revalidate/no-store) y verificada en endpoints sensibles.

- **[Observabilidad]**
  - Logging ajustado por entorno, sin PII innecesaria y con muestreo si aplica.
