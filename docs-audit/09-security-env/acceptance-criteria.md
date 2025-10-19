# Criterios de aceptación (09-security-env)

- **[Env validation]**
  - Build falla si faltan envs críticas en producción; se registra cuáles faltan.
  - Validador de runtime central (server y client `NEXT_PUBLIC_*`) con mensajes claros.

- **[CSP/headers]**
  - CSP sin `'unsafe-inline'/'unsafe-eval'` o justificada con nonce; dominios mínimos.
  - Headers de seguridad activos y verificados en prod.

- **[CORS]**
  - Orígenes restringidos en `/api` y rutas generales; sin `*` en prod.

- **[Rate limiting]**
  - Limitar `/api` y webhooks con estrategia por IP/UA, con métricas y alertas.

- **[Secretos]**
  - Segregación server-only vs client-only validada; rotación periódica definida.

- **[Logging]**
  - Política de logs sin PII innecesaria y con muestreo en prod.
