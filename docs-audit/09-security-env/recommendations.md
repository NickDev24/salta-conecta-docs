# Recomendaciones (09-security-env)

- **[Fail-fast de build]**
  - Cambiar validación en `next.config.js` para fallar el build si faltan envs requeridas en prod (o usar script prebuild). Mantener lista blanca clara.

- **[Runtime env validator]**
  - Agregar validador central (Zod/envsafe) para server y para `NEXT_PUBLIC_*` consumidos en cliente. Lanzar errores con mensajes claros.

- **[CSP endurecida con nonce]**
  - Remover `'unsafe-inline'/'unsafe-eval'` donde sea posible y usar `nonce` (`x-nonce` desde `middleware.ts`) para scripts inline. Limitar `connect-src`/`frame-src`/`script-src` a dominios mínimos necesarios.

- **[CORS restrictivo]**
  - Evitar comodín `*` en prod; usar dominio exacto y sólo métodos/headers necesarios. Alinear con `globalCorsOrigin`.

- **[Rate limiting y anti-abuso]**
  - Implementar rate limiting (IP/UA) para `/api` y webhooks (e.g., `rate-limiter-flexible`, `upstash/ratelimit`). Añadir circuit breaker/timeout en integraciones externas.

- **[Gestión de secretos]**
  - Rotación de tokens/secrets (NextAuth/JWT/webhooks/MP/Cloudinary/Resend/Redis). Uso de secretos por entorno y bloqueo de exposición en cliente.

- **[Logging]**
  - Reducir granularidad de logs de request o muestrear; anonimizar IP/UA si aplica cumplimiento. Alertas para eventos de seguridad (bots bloqueados, intentos fallidos).
