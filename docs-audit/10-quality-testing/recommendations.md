# Recomendaciones (10-quality-testing)

- **[Cobertura progresiva]**
  - P0: mantener umbral global 70% y activar reporting `lcov`/`text-summary` en CI.
  - P1: subir a 80% global en 4-6 semanas; módulos críticos (pagos/webhooks/auth) ≥85% líneas/branches.
  - P2: objetivo 90% global con ramas clave ≥90% y exclusiones justificadas.

- **[Ampliar suite]**
  - Agregar tests para `app/api/mercadopago/webhook` (idempotencia, firma inválida, duplicados) y `middleware` (RBAC por prefijo, bots/HTTPS).
  - Cubrir caminos de error en stores/contexts (timeout SSE, socket desconectado, uploads fallidos).

- **[Estabilidad]**
  - Mock unificado de `next-auth`, `lib/mongodb` y `mongoose` para unit tests; usar `mongodb-memory-server` sólo en integración.
  - Evitar dependencias de red/tiempo real; usar fixtures determinísticas.

- **[CI]**
  - Añadir pasos: `npm run lint`, `npm run type-check`, `npm run test:coverage` con gates de umbral.
  - Publicar reportes cobertura (JUnit/LCOV) y tiempos por test para detectar regresiones.

- **[Buenas prácticas]**
  - Nomenclatura consistente `*.test.ts[x]`; estructura por dominio (api/components/lib/integration).
  - Tests accesibilidad (axe) en componentes críticos.
