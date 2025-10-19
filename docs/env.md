# Variables de entorno (Build y Runtime)

Esta guía documenta todas las variables que el proyecto utiliza, indicando si son requeridas en build, en runtime, o ambas. No almacenes secretos reales en el repositorio.

## Principios

- Usa Node 20 LTS.
- Build no aborta si faltan variables (se advierte), pero en runtime pueden ocurrir errores si no están presentes.
- Gestioná secretos vía `.env.local` (dev), secrets de CI/CD, o variables del orquestador (Docker/Vercel/etc.).

## Variables críticas (requeridas en producción)

- NEXT_PUBLIC_APP_URL: URL pública base de la app (p.ej. https://example.com). [Build: opcional] [Runtime: requerido]
- NEXTAUTH_URL: URL base para NextAuth (igual a APP_URL). [Build: opcional] [Runtime: requerido]
- NEXTAUTH_SECRET: secreto de NextAuth (64+ chars). [Build: opcional] [Runtime: requerido]
- MONGODB_URI: cadena de conexión MongoDB. [Build: opcional] [Runtime: requerido]
- JWT_SECRET: secreto para firmar JWT propios. [Build: opcional] [Runtime: requerido]
- ENCRYPTION_KEY: clave simétrica para cifrado (32 bytes base64/utf8). [Build: opcional] [Runtime: requerido]
- NEXT_PUBLIC_GOOGLE_MAPS_API_KEY: API key de Maps expuesta al cliente. [Build: opcional] [Runtime: requerido]

## MercadoPago

- NEXT_PUBLIC_MERCADOPAGO_PUBLIC_KEY: public key cliente. [Runtime]
- MP_ACCESS_TOKEN / MERCADOPAGO_ACCESS_TOKEN: token server. [Runtime]
- MP_APPLICATION_ID: id de app (opcional). [Runtime]
- NEXT_WEBHOOK_MERCADOPAGO_KEY_SECRET: secreto para validar webhooks. [Runtime]
- MP_CLIENT_ID / MP_CLIENT_SECRET: credenciales OAuth (si se usa). [Runtime]
- MP_REDIRECT_URI: callback OAuth. [Runtime]
- MP_AUTH_HOST, NEXT_PUBLIC_MERCADOPAGO_LOCALE, NEXT_PUBLIC_MP_INTEGRATOR_ID, MP_PAYOUTS_ENABLED, MP_USE_APPLICATION_FEE: configuración avanzada. [Runtime]

## Email (Resend)

- RESEND_API_KEY, RESEND_WEBHOOK_SECRET, RESEND_FROM, RESEND_MONTHLY_LIMIT, RESEND_POLICY_STRICT, RESEND_ALLOWED_TYPES. [Runtime]

## Cloudinary

- CLOUDINARY_CLOUD_NAME, CLOUDINARY_API_KEY, CLOUDINARY_API_SECRET, CLOUDINARY_UPLOAD_PRESET, NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME. [Runtime]

## Redis / Realtime

- REDIS_URL, REDIS_TOKEN, REDIS_HOST, REDIS_PORT, REDIS_PASSWORD. [Runtime]
- UPSTASH_REDIS_REST_URL, UPSTASH_REDIS_REST_TOKEN. [Runtime]

## Varias de App

- NEXT*PUBLIC_APP_NAME, DRIVER*_, ORDER\__, PUSH_NOTIFICATIONS_ENABLED, EMAIL_NOTIFICATIONS_ENABLED, SMS_NOTIFICATIONS_ENABLED, ANALYTICS_ENABLED, DEBUG_MODE. [Runtime]
- NEXT_PUBLIC_VAPID_PUBLIC_KEY, VAPID_PRIVATE_KEY (si se usan push). [Runtime]

## Otras

- PORT: puerto de escucha (por defecto 3000). [Runtime]
- LOG_LEVEL: nivel de logging. [Runtime]

## Recomendaciones de seguridad

- No commitear `.env`, `.env.local`, `.env.production`. Usar plantillas `*.example` con placeholders.
- Rotar credenciales expuestas y moverlas a un gestor de secretos.
- CSP y cabeceras ya se aplican en `next.config.js`; validar dominios externos que uses.

## Matriz Build vs Runtime

- Build: no requiere variables (solo warnings si faltan). Permite generar artefactos.
- Runtime: variables marcadas "requerido" deben estar definidas para funcionamiento correcto.

## Ejemplos de definición (Docker Compose)

```yaml
services:
  app:
    build: .
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_APP_URL=https://example.com
      - NEXTAUTH_URL=https://example.com
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
      - MONGODB_URI=${MONGODB_URI}
      - JWT_SECRET=${JWT_SECRET}
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}
      - NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=${NEXT_PUBLIC_GOOGLE_MAPS_API_KEY}
      # ... resto de integraciones
```

## Local

- Usar `.env.local` para desarrollo.
- No reutilizar secretos de producción.
