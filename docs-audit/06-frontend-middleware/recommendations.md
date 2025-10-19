# Recomendaciones (06-frontend-middleware)

- **[RBAC más robusto]**
  - Centralizar lista de prefijos protegidos y mapear roles → rutas. Añadir página 403 para denegación (en vez de solo redirect a login).
  - Validar token/jwt expirado y renovar sesión si corresponde.

- **[HTTPS/HSTS]**
  - Confirmar encabezados `x-forwarded-proto`/`x-forwarded-host` en la plataforma. Añadir bypass para healthchecks si aplica.

- **[Bots y abuso]**
  - Complementar UA-block con rate limiting por IP/UA, detección de patrones y tarpit para scrapers recurrentes.
  - Opcional: CAPTCHA/journey para paths sensibles públicos.

- **[Nonce y CSP]**
  - Usar `x-nonce` inyectándolo en layouts cuando haya inline scripts. Avanzar hacia CSP sin `'unsafe-inline'`.

- **[Caché/SSR]**
  - Definir por página `revalidate`/`cache: 'no-store'` según sensibilidad. Evitar cachear contenido con datos de usuario.

- **[Observabilidad]**
  - Reducir nivel de `logger.info` a `debug` en producción o muestrear. Enmascarar IP/UA si compliance lo requiere.
