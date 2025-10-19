# Inventario del Repositorio

## Estructura general

- **app/**: Rutas de aplicación y API Routes (`app/api/`).
- **pages/**: Compatibilidad con `pages/api/` (Socket.IO).
- **components/**: Componentes UI reutilizables.
- **lib/**: Utilitarios, integraciones (auth, redis, comisión, etc.).
- **models/**: Modelos Mongoose.
- **hooks/**, **contexts/**: Hooks/Contextos React.
- ****tests**/**: Pruebas unitarias/integración.
- Config: `next.config.js`, `docker-compose.yml`, `.env*`, `tailwind.config.js`.

## Conteos clave

- **API route handlers (app/api)**: 366 archivos `route.ts`
- **API en pages/**: 1 (`pages/api/socket.ts`)
- **Componentes (.tsx)**: 93
- **Modelos (.ts)**: 52
- **Tests**: 28

## Observaciones iniciales

- **Autenticación/roles**: Uso de `getServerSession()` y helpers como `requireAdmin` en endpoints (`app/api/admin/analytics/route.ts`).
- **Validación**: Presencia de validaciones manuales; uso de `zod` detectado en el repo, no siempre aplicado en endpoints.
- **Servicios externos**: Mercado Pago, Resend, Cloudinary, Google Maps, Redis (ioredis/Upstash), Socket.IO.
- **DB**: Mongoose/MongoDB predominante; también existe `drizzle-orm` en dependencias.
