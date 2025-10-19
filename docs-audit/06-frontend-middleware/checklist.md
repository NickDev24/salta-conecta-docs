# Checklist (06-frontend-middleware)

- **[RBAC]** Prefijos protegidos (`/admin`, `/provider`) con `next-auth/jwt` y roles de `USER_ROLES`. Revisar que coincidan con rutas reales y que exista fallback 403.
- **[HTTPS/HSTS]** Redirección a HTTPS activa en prod y cabecera HSTS aplicada sólo cuando corresponde (tras CDN/proxy). Confirmar `x-forwarded-proto` en plataforma.
- **[Headers]** Verificar que CSP fuerte esté en `next.config.js` y que el `nonce` (`x-nonce`) se use en layouts si hay inline scripts.
- **[Bots y rate limiting]** UA-block listo; añadir rate limiting por IP/UA y challenge si es necesario.
- **[Scope]** `matcher` excluye `api` y assets; confirmar que no se bloquee contenido estático/imagenes. Revisar rutas públicas que no requieran RBAC.
- **[Cache/SSR]** Definir estrategia de caché por ruta/página (revalidate, headers) fuera del middleware. Evitar cachear respuestas con datos sensibles.
- **[Observabilidad]** Revisión de volumen de logs y PII; sanitizar IP/UA si se requiere cumplimiento.
