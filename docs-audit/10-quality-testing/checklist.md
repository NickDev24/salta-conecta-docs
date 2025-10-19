# Checklist (10-quality-testing)

- **[Configuración]** Jest usa `next/jest`; aliases `@/` mapeados; `jest.setup.js` con Testing Library.
- **[Cobertura]** Umbral global inicial ≥70% (branches/functions/lines/statements). Reportes `lcov`, `text-summary` para CI.
- **[Pruebas]** Casos para estados loading/success/error, paginación/filtros, errores 401/403, y paths críticos (pagos/webhooks).
- **[Integración]** DB in-memory (`mongodb-memory-server`) con `beforeAll/afterAll` robusto y skip si falla setup.
- **[Tiempos]** `testTimeout` por defecto 10s; tests de integración con 20–30s. Evitar sleeps innecesarios.
- **[Mocks]** `next-auth`, `mongoose` y `fetch` mockeados consistentemente; evitar flaky por red/tiempos.
- **[TS/ESLint]** `tsc --noEmit` en CI; ESLint `next/core-web-vitals`; fail on error.
