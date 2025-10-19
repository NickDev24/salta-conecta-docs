# Checklist (09-security-env) + OWASP

- **[Env validation]**
  - Build: fallar si faltan requeridas en prod (no solo `console.warn`).
  - Runtime: validador central (Zod/envsafe) para server y client (`NEXT_PUBLIC_*`).
  - Segregar server-only vs client-only; revisar `env-matrix.csv`.

- **[Secrets handling]**
  - Rotación periódica (NextAuth, JWT, webhooks, MP/Resend/Cloudinary/Redis).
  - Nunca exponer secretos en `next.config.js env` (solo `NEXT_PUBLIC_*`).
  - Logging: evitar imprimir valores de env.

- **[CSP/headers]**
  - Reducir `'unsafe-inline'/'unsafe-eval'`; usar nonce (`x-nonce`) y CSP basada en nonce.
  - Mantener `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`.
  - Verificar `frame-src`, `connect-src` mínimos necesarios (Maps/MP/WS).

- **[CORS]**
  - Evitar `*` en prod; usar dominio exacto (`NEXT_PUBLIC_BASE_URL|APP_URL`).
  - Permitir headers/métodos mínimos.

- **[HTTPS/HSTS]**
  - Forzar HTTPS en prod; aplicar HSTS correctamente detrás de CDN/proxy.

- **[Rate limiting]**
  - Implementar limitador (IP/UA) en `/api` y webhooks.
  - Protecciones anti-bot/abuso complementarias (captcha/challenge).

- **[OWASP ASVS/Top 10]**
  - A01 Broken Access Control: RBAC en `middleware.ts` + pruebas.
  - A02 Cryptographic Failures: manejo de secrets y TLS.
  - A03 Injection: sanitización entradas (Zod, queries), no evals.
  - A05 Security Misconfiguration: CSP/CORS/headers consistentes.
  - A07 Identification and Authentication: NextAuth secrets y expiración.
  - A08 Software and Data Integrity: pin de deps, CI checks.
  - A09 Security Logging and Monitoring: logs sin PII sensible, alertas.
  - A10 SSRF: listas de allow para `fetch` server-side si aplica.
