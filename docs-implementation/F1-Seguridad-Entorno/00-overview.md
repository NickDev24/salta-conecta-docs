# F1 – Seguridad y Entorno: Overview

## Objetivo general

- **Endurecer la superficie de ataque** del frontend/backend Next.js reforzando la Content Security Policy (CSP) con `nonce`, eliminando `unsafe-inline` y `unsafe-eval`.
- **Prevenir fallas en runtime** mediante un validador fail-fast de variables de entorno en `next.config.js` (o inicialización equivalente) que detenga build/deploy si falta configuración crítica.
- **Reducir exposición y abuso** configurando CORS restrictivo con lista blanca por entorno y aplicando rate limiting a rutas sensibles de `/app/api/**` y webhooks.

## Alcance

- **Configuración de framework**: `next.config.js`, `middleware.ts`, `vercel.json`, `server.js` (si aplica SSR/custom server).
- **Rutas de API**: `app/api/**` y `pages/api/**` (p.ej., `pages/api/socket.ts`).
- **Webhooks**: endpoints bajo `app/api/**/webhook*` o equivalentes.
- **Cliente/Frontend**: identificación y mitigación de scripts inline en `pages/_app.tsx` y componentes `app/**` para adopción de `nonce`/`sha256`.
- **Entorno/Infra**: `.env`, `.env.local`, `.env.example`, `Dockerfile`, `docker-compose.yml` para coherencia y propagación segura de variables.
- Quedan fuera del alcance de F1 las refactorizaciones de modelos/dominio (F2) y cambios de UX/UI (F3), salvo lo mínimo indispensable para cumplir CSP.

## Criterios de finalización

- **CSP endurecida**:
  - `script-src` y `style-src` sin `unsafe-inline` ni `unsafe-eval`.
  - Mecanismo de `nonce` operativo y documentado; scripts críticos migrados o justificados con `sha256`.
  - Evidencia: headers en respuestas y pruebas mostrando bloqueo de inline no autorizado.
- **Validador de entorno**:
  - Build/deploy falla si faltan variables críticas; paridad actualizada en `.env.example`.
  - Evidencia: logs de build/CI y lista de variables validadas.
- **CORS restrictivo**:
  - Lista blanca por entorno (dev/staging/prod) aplicada a `/app/api/**` y webhooks.
  - Métodos y headers limitados al mínimo necesario; sin comodines amplios en producción.
- **Rate limiting**:
  - Límites configurados y probados en rutas sensibles (auth, pagos, webhooks).
  - Evidencia: respuestas 429 bajo carga de prueba y métricas básicas.
- **Documentación completa** en `docs-implementation/F1-Seguridad-Entorno/`:
  - `01-preparacion.md`, `02-modificaciones.md`, `03-validaciones.md`, `04-resultados.md`, `05-log-cambios.md` completos.

## Dependencias con otras fases

- **F2 – Backend y Modelos**: puede requerir exponer endpoints adicionales o modificar contratos; mantener alineación para no romper CORS ni rate limits.
- **F3 – Frontend y UX**: posibles ajustes para remover inline scripts/styles y adoptar patrones compatibles con CSP.
- **F4 – DevOps y Calidad**: integración del validador de entorno en CI/CD, gestión de variables por entorno, monitorización de headers y límites en despliegues.
