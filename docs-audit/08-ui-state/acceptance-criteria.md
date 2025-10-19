# Criterios de aceptación (08-ui-state)

- **[SSR-safety]**
  - Hooks/contexts cliente marcados y con cleanup correcto; stores `persist` no usados en server components.
  - Guards añadidos donde haya acceso potencial a `window` o APIs del navegador.

- **[A11y]**
  - Botones de íconos con `aria-label`; inputs con `label`/`aria-*`.
  - Componentes de progreso con roles/atributos ARIA completos.
  - Anuncios dinámicos con `aria-live` y manejo de focus.

- **[Performance]**
  - Derivados O(n) memoizados o justificados (<50 items). Sin renders innecesarios por contextos globales.
  - Estrategias de virtualización documentadas para listas grandes.

- **[Patrones]**
  - `LoadingProvider` adoptado en flujos críticos; APIs de contextos consistentes y documentadas.
