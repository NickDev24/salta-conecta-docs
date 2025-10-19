# F1 – Modificaciones Técnicas

## 1. Endurecimiento CSP

### Objetivo

- **Eliminar** `unsafe-inline` y `unsafe-eval` de `script-src`/`style-src`.
- **Introducir** `nonce` dinámico para scripts/styles permitidos.
- **Validar primero** en `Content-Security-Policy-Report-Only` y luego aplicar `Content-Security-Policy`.

### Pasos detallados (planificación, sin ejecutar todavía)

1. **Generación de nonce** por request en `middleware.ts` usando `crypto.randomUUID()` → guardar en header `x-nonce`.
2. **Aplicación de headers** en `next.config.js` (función `headers()`) o, alternativamente, directamente en `middleware.ts` si se requiere lógica condicional por ruta/entorno. Si existe `vercel.json`, alinear configuración para evitar duplicidad.
3. **Definir CSP** mínima segura:
   - `default-src 'self'`.
   - `script-src 'self' 'nonce-<dinámico>'` y dominios estrictamente necesarios (p.ej., `https://maps.googleapis.com` y `https://maps.gstatic.com`).
   - `style-src 'self' 'nonce-<dinámico>'` y `https://fonts.googleapis.com` si aplica.
   - `img-src 'self' data: https://maps.gstatic.com` si corresponde.
   - `object-src 'none'`, `frame-ancestors 'none'`, `base-uri 'self'`, `upgrade-insecure-requests`.
4. **Migración de inline**:
   - Reemplazar `<script>inline</script>` por inserción controlada con `nonce` usando `next/script` o mover a módulos TS/TSX.
   - Revisar `pages/_app.tsx` y componentes críticos en `app/**` para eliminar atributos inline y handlers en markup cuando sea posible.
5. **Evaluaciones/SDKs**: identificar dependencias que usan `eval()`; sustituir o aislar. Como medida temporal documentada, se podría permitir `sha256-...` para un script específico mientras se elimina esa dependencia.
6. **Fase Report-Only**: desplegar inicialmente `Content-Security-Policy-Report-Only` con la misma directiva para recolectar violaciones (opcional `report-to`/`report-uri`). Tras corregir violaciones, pasar a `Content-Security-Policy` bloqueante.

### Fragmentos de ejemplo (no código completo)

```ts
// middleware.ts (fragmento conceptual)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(req: NextRequest) {
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64');
  const res = NextResponse.next({ request: { headers: req.headers } });
  res.headers.set('x-nonce', nonce);
  return res;
}
```

```js
// next.config.js (fragmento conceptual)
const headers = async () => [
  {
    source: '/(.*)',
    headers: [
      // Fase 1: Report-Only; luego cambiar a Content-Security-Policy
      {
        key: 'Content-Security-Policy-Report-Only',
        value:
          "default-src 'self'; script-src 'self' 'nonce-${'${nonce}'}' https://maps.googleapis.com https://maps.gstatic.com; style-src 'self' 'nonce-${'${nonce}'}' https://fonts.googleapis.com; img-src 'self' data: https://maps.gstatic.com; object-src 'none'; frame-ancestors 'none'; base-uri 'self'; upgrade-insecure-requests",
      },
      { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
      { key: 'X-Content-Type-Options', value: 'nosniff' },
      { key: 'X-Frame-Options', value: 'DENY' },
    ],
  },
];
module.exports = { headers };
```

```tsx
// pages/_app.tsx (fragmento conceptual)
import Script from 'next/script';

export default function App({ Component, pageProps, nonce }) {
  return (
    <>
      <Script id="bootstrap" nonce={nonce} strategy="afterInteractive">
        {`window.__BOOTSTRAP = true;`}
      </Script>
      <Component {...pageProps} />
    </>
  );
}
```

### Justificación y riesgos

- Reduce superficie de XSS al eliminar `unsafe-*` y exigir `nonce`.
- Riesgo: ruptura de scripts de terceros o inline no migrados; se mitiga con `Report-Only` y ventana de corrección.

### Plan de rollback

- Cambiar header de `Content-Security-Policy` a `Content-Security-Policy-Report-Only` temporalmente.
- Revertir directivas a la versión previa con los diffs preparados en `01-preparacion.md`.

---

## 2. Validador de Entorno Fail-Fast

### Objetivo técnico

- Validar variables críticas definidas en `docs-audit/02-inventory/env-matrix.csv`.
- Hacer fallar el build/deploy si falta alguna requerida.

### Pasos detallados (planificación, sin ejecutar todavía)

1. **Módulo** `lib/env.ts` con Zod (o envsafe) definiendo `EnvSchema` para todas las claves críticas.
2. **Separar** variables públicas `NEXT_PUBLIC_*` de privadas, validando formatos (URLs, tokens, mínimos de longitud, enums).
3. **Integración**: invocar `validateEnv(process.env)` desde `next.config.js` al inicio; si `safeParse` falla, lanzar `Error` para cortar el build.
4. **Paridad**: actualizar `.env.example` para reflejar todas las variables requeridas (sin valores secretos).
5. **Bypass de emergencia**: `DISABLE_ENV_VALIDATION=true` solo para incidentes, documentado y temporal.

### Fragmentos de ejemplo (no código completo)

```ts
// lib/env.ts (fragmento)
import { z } from 'zod';

export const EnvSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']),
  NEXT_PUBLIC_API_BASE: z.string().url(),
  MONGODB_URI: z.string().min(1),
  JWT_SECRET: z.string().min(32),
  // ... completar según env-matrix.csv
});

export function validateEnv(env = process.env) {
  const parsed = EnvSchema.safeParse(env);
  if (!parsed.success) {
    console.error(parsed.error.format());
    throw new Error('ENV_VALIDATION_FAILED');
  }
  return parsed.data;
}
```

```js
// next.config.js (fragmento)
const { validateEnv } = require('./lib/env');
if (!process.env.DISABLE_ENV_VALIDATION) {
  validateEnv(process.env); // build fail si falta variable crítica
}
module.exports = {
  /* resto config */
};
```

### Justificación y riesgos

- Previene fallas en runtime por configuración ausente o inválida.
- Riesgo: cortes de build en entornos no preparados; mitigación con `.env.example` actualizado y validación temprana en CI.

### Plan de rollback

- Habilitar `DISABLE_ENV_VALIDATION=true` temporalmente (documentado) y abrir hotfix.
- Quitar la llamada a `validateEnv` en `next.config.js` con PR de reversión si fuera necesario.

---

## 3. CORS Restrictivo y Rate Limiting

### Objetivo técnico

- Limitar orígenes permitidos por entorno y controlar el abuso en rutas sensibles de `/api/**` y webhooks.

### Pasos detallados (planificación, sin ejecutar todavía)

1. **ALLOWED_ORIGINS** por entorno en `.env` (coma-separado); reflejar en `.env.example`.
2. **Middleware CORS** en App Router para `/app/api/**` y webhooks:
   - Rechazar origen no listado (`403`).
   - Incluir `Vary: Origin`, `Access-Control-Allow-Origin` con eco del origen permitido, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` mínimos.
   - En producción, no usar `*`.
3. **Rate limiting** con `rate-limiter-flexible` u otro:
   - Storage in-memory en dev; Redis/Upstash en prod (F4 coordina infraestructura).
   - Límites por ruta: p.ej., `/api/mercadopago/webhook` 20 req/min, `/api/auth/*` 30 req/min, `/api/orders/*` 60 req/min.
   - Clave de limitación basada en IP (`x-forwarded-for`) o token de cliente cuando aplique.

### Fragmentos de ejemplo (no código completo)

```ts
// app/api/_middleware.ts (fragmento conceptual CORS)
import { NextResponse } from 'next/server';
const allowed = (process.env.ALLOWED_ORIGINS || '').split(',');

export function middleware(req: Request) {
  const origin = req.headers.get('origin') || '';
  if (origin && !allowed.includes(origin)) {
    return new NextResponse('Origin Not Allowed', { status: 403 });
  }
  const res = NextResponse.next();
  res.headers.set('Vary', 'Origin');
  res.headers.set('Access-Control-Allow-Origin', origin);
  res.headers.set(
    'Access-Control-Allow-Methods',
    'GET,POST,PUT,PATCH,DELETE,OPTIONS'
  );
  res.headers.set(
    'Access-Control-Allow-Headers',
    'Content-Type, Authorization'
  );
  return res;
}
```

```ts
// app/api/mercadopago/webhook/route.ts (fragmento conceptual rate limit)
import { RateLimiterMemory } from 'rate-limiter-flexible';
const limiter = new RateLimiterMemory({ points: 20, duration: 60 });

export async function POST(req: Request) {
  try {
    await limiter.consume(
      'webhook:ip:' + (req.headers.get('x-forwarded-for') || 'local')
    );
  } catch {
    return new Response('Too Many Requests', { status: 429 });
  }
  return new Response('ok', { status: 200 });
}
```

### Justificación y riesgos

- Minimiza superficie de abuso y acceso desde orígenes no autorizados.
- Riesgo: falsos 403/429 si la lista blanca o los límites están mal calibrados; mitigación con valores por entorno y pruebas controladas.

### Plan de rollback

- Relajar temporalmente la lista blanca agregando los orígenes necesarios.
- Incrementar límites o desactivar limitación en una ruta específica para mitigar impactos mientras se corrige.

---

## Referencias

- Hallazgos y criterios: `docs-audit/03-config-build/*`, `docs-audit/09-security-env/*`.
- Matriz de variables: `docs-audit/02-inventory/env-matrix.csv`.
- Dominios externos (p.ej., Google Maps) según `config/maps.ts` y uso en `contexts/GoogleMapsContext.tsx`.
