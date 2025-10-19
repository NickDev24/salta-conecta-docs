# Preparación previa (F1 – Seguridad y Entorno)

## Alcance de la preparación

- **Objetivo**: Establecer línea base y respaldos antes de endurecer CSP, validar variables de entorno y configurar CORS/rate limiting.
- **Archivos/áreas afectadas**:
  - `next.config.js`, `middleware.ts`, `pages/_app.tsx`, `server.js` (si aplica SSR custom), `vercel.json`.
  - Rutas API en `app/api/**` y `pages/api/**` (p.ej. `pages/api/socket.ts`).
  - Configuraciones de entorno: `.env`, `.env.local`, `.env.example`, `docker-compose.yml`, `Dockerfile`.
  - Endpoints de webhooks en `app/api/**/webhook*` o similares.

## Inventario del estado actual

- **CSP/headers**
  - Revisar si CSP se establece en `next.config.js` (`headers()`), en `middleware.ts` o mediante proxy/CDN.
  - Identificar presencia de `unsafe-inline` / `unsafe-eval` en `script-src`, `style-src`.
  - Detectar uso de inline scripts/styles en `pages/_app.tsx`, componentes en `app/**`, o atributos `style`/`on*`.
- **Variables de entorno**
  - Enumerar variables usadas en `next.config.js`, `lib/**`, `app/api/**`, `server.js`.
  - Diferenciar variables públicas (expuestas al cliente) vs. privadas.
  - Verificar paridad entre `.env.example` y variables requeridas en código.
- **CORS y orígenes**
  - Ubicar cualquier middleware/handler CORS existente en `app/api/**` o `server.js`.
  - Listar orígenes actuales permitidos (wildcards, `*`, localhost, dominios productivo/staging).
- **Rate limiting**
  - Comprobar si existe algún limitador (e.g., `rate-limiter-flexible`, middleware custom) en `app/api/**` o edge functions.
- **Webhooks**
  - Enumerar endpoints de webhooks (p.ej., `app/api/mercadopago/webhook`, `app/api/**/webhook`) y sus validaciones de firma.
- **Infra/Deploy**
  - Relevar `vercel.json` (headers, edge, routes), `docker-compose.yml`, `Dockerfile` y `server.js` para impacto de headers/CORS.

## Respaldos y línea base (no destructivo)

- **Snapshot de configuración**
  - Copiar contenido actual de `next.config.js`, `middleware.ts`, `vercel.json`, `server.js` a `docs-implementation/F1-Seguridad-Entorno/artefactos/` (solo para documentación; no se versionará código duplicado en producción).
  - Exportar contenido de `.env.example` (sin secretos reales) a `artefactos/env.example.snapshot`.
- **Evidencia de headers actuales**
  - Registrar muestra de respuestas de endpoints de sitio y API (producción/staging/local) con headers actuales (CSP, CORS). Incluir host/fecha/endpoint y salida resumida.
- **Paridad de variables**
  - Generar checklist de variables requeridas y marcarlas presentes/ausentes respecto de `.env.example`.
- **Plan de rollback (documental)**
  - Mantener diffs preparados para revertir cambios en `next.config.js`, `middleware.ts` y handlers API si fuera necesario.

## Chequeos específicos previos

- **CSP**
  - Confirmar si hay scripts inline críticos que requieran migración a `nonce` o `sha256`.
  - Validar dependencia de librerías que usan `eval` (p.ej., ciertas analytics o SDKs antiguos).
- **Entorno**
  - Identificar variables mínimas para que el build falle temprano si faltan (`process.env.*` en inicialización).
  - Verificar valores por entorno: desarrollo (`.env.local`), CI, producción (Vercel/host) y Docker.
- **CORS/rate limit**
  - Precisar orígenes legítimos (dominios productivo/staging), métodos y headers necesarios.
  - Determinar rutas sensibles (auth, webhooks, pagos) con límites más estrictos.

## Riesgos y mitigación

- **Ruptura por CSP**: Bloqueo de scripts de terceros o inline. Mitigar con rollout gradual y `report-uri` opcional.
- **Falsos negativos de entorno**: Variables no validadas causan fallos en runtime. Mitigar con validador en `next.config.js` y pruebas en CI.
- **CORS excesivo**: Orígenes demasiado amplios. Mitigar con lista blanca explícita y verificación por entorno.
- **Rate limiting**: Configuración demasiado estricta provocando 429. Mitigar con valores diferenciados por ruta y entorno.

## Criterios de inicio de ejecución

- Inventario completado y snapshots documentados.
- Lista blanca de orígenes acordada.
- Variables críticas definidas y reflejadas en `.env.example`.
- Plan de rollback anotado en esta fase.
