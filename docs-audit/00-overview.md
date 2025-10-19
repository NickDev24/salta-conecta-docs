# Auditoría técnica - Salta Conecta

## Objetivos

- **Evaluar seguridad** de API, autenticación, autorización y manejo de datos sensibles.
- **Revisar calidad y mantenibilidad** del código (Next.js 15, TypeScript, patrones, testing).
- **Analizar integración** con servicios externos (MongoDB, Redis, Mercado Pago, Cloudinary, Resend).
- **Detectar riesgos** en dependencias, configuración, y despliegue (Docker/DevOps).
- **Proveer evidencias** y una hoja de ruta de mejoras priorizadas por severidad e impacto.

## Alcance

- **Frontend/Backend** en `app/`, `pages/`, `components/`, `lib/`, `models/`, `hooks/`, `contexts/`.
- **API Routes** en `app/api/` y `pages/api/`.
- **Configuración**: `.env*`, `next.config.js`, `docker-compose.yml`, `vercel.json`.
- **Tests** en `__tests__/`.
- **Dependencias** en `package.json`.

No se realizarán cambios de código. La auditoría se limita a lectura, inventario, matrices y recomendaciones.

## Supuestos

- Entorno de desarrollo local estándar (Linux) con Node 18+.
- Variables de entorno provistas por `.env.example` y `env.example` representan el set esperado.
- MongoDB como base principal; Redis disponible (ioredis/Upstash) para caché/colas; Mercado Pago como PSP; Cloudinary para media; Resend para email.
- Autenticación con NextAuth (`next-auth`).

## Metodología

- **Inventario** del repositorio y endpoints.
- **Revisión estática** de patrones de auth/roles/validaciones y manejo de errores.
- **Extracción** de variables de entorno y dependencias con clasificación de riesgos.
- **Priorización** por severidad (Critical/High/Medium/Low), impacto y esfuerzo.
- **Evidencias** en matrices CSV y referencias a archivos y símbolos (`ruta/archivo.ts`, `miFuncion()`).
