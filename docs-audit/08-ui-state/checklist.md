# Checklist (08-ui-state)

- **[SSR-safety]**
  - Hooks/contexts que acceden a browser APIs marcan `'use client'` y limpian efectos (`EventSource`, geoloc, sockets).
  - Stores con `persist` se usan sólo en Client Components; evitar importar en server.
  - Guards `typeof window !== 'undefined'` donde haya riesgo.

- **[A11y]**
  - Botones con íconos tienen `aria-label`; inputs con `label` o `aria-label` y `id`/`htmlFor`.
  - Componentes dinámicos (chat/notifs) usan `aria-live` y focus management al abrir/cerrar.
  - Progreso con `role="progressbar"`, `aria-valuenow/min/max` y texto alternativo.

- **[Performance]**
  - Memoizar derivados O(n) con `useMemo` si listas >50; usar `useCallback` para handlers.
  - Virtualizar listas grandes; evitar cálculos en render si pueden precalcularse.
  - Evitar renders por cambios irrelevantes (split context/selectores de store).

- **[Patrones]**
  - Contextos granulares (Loading/Notifications/Maps) con APIs consistentes.
  - Stores `zustand` con `partialize` donde aplique y nombres de storage únicos.
  - Hooks con cleanup robusto y API estable.
