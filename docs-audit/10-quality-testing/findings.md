# Hallazgos (10-quality-testing)

- **[Configuración Jest]**
  - `jest.config.js` usa `next/jest`, mapea alias `@/`, `testEnvironment: jsdom`, `setupFilesAfterEnv` con Testing Library, y define `collectCoverageFrom` sobre `components/`, `lib/`, `app/`, `store/` con exclusiones estándar.
  - `coverageThreshold` global en 70% para branches/functions/lines/statements. `testTimeout` 10s.

- **[Suite de tests]**
  - Presencia de tests de UI (`__tests__/components/sections.test.tsx`) que cubren estados loading/data/error y mockean `fetch`.
  - Tests de API `GET /api/admin/orders` basados en `app/api/admin/orders/route.ts` con mocks de auth y modelo, validan filtros, paginación, y errores 401/403.
  - Test de integración con DB in-memory para `scheduler` usando `mongodb-memory-server`, con `jest.setTimeout(30000)` y manejo de skip si falla el start.

- **[Cobertura y tiempos]**
  - Umbral actual 70% (global). No hay reporte del valor real de cobertura generado en repo; `test:coverage` disponible.
  - Tiempos: unitarios UI rápidos; integración de scheduler ajusta timeout a 20–30s.

- **[Tipado/TS]**
  - `tsconfig.json` con `strict: true`, `noEmit`, `moduleResolution: bundler`. Archivo `type-check-errors.txt` vacío (sin errores conocidos).

- **[ESLint]**
  - `.eslintrc.json` extiende `next/core-web-vitals`. Reglas: `no-unused-vars` error, `no-debugger` warn, `prefer-const` error, `no-console` off.

- **[Oportunidades]**
  - Falta reporter de cobertura en formatos `lcov`/`cobertura` para CI.
  - Tests de API adicionales: rutas críticas de pagos/webhooks y permisos RBAC de `middleware`.
  - Mocks más robustos de `next-auth` y `mongoose` para reducir flaky.
