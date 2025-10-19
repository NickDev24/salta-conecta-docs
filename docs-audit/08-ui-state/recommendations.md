# Recomendaciones (08-ui-state)

- **[A11y inmediata]**
  - Añadir `aria-label` a botones de ícono en `components/chat/support-chat.tsx` y similares.
  - En formularios/inputs, incluir `label` visible o `aria-label`/`aria-labelledby`.
  - En barras de progreso (e.g., `free-shipping-progress.tsx`), agregar `role="progressbar"`, `aria-valuenow`, `aria-valuemin`, `aria-valuemax` y texto alternativo.

- **[SSR-safety]**
  - Mantener `'use client'` en hooks/contexts que usan Web APIs y proteger imports desde server.
  - Documentar en README de UI el patrón de uso de stores `zustand` (solo en Client Components) para evitar hydration mismatch.

- **[Performance]**
  - Memoizar derivados O(n) (`unreadCount`, filtros por categoría) con `useMemo` si la lista supera 50 elementos.
  - Considerar virtualización si listas crecen (no necesario hoy).
  - Dividir contextos muy activos o exponer selectores para minimizar renders.

- **[Patrones]**
  - Estandarizar `LoadingProvider`/`withLoading` para operaciones async; promover su uso en páginas críticas.
  - Añadir `aria-live="polite"` para feeds de chat/notificaciones y manejo de focus al abrir/cerrar el chat.
