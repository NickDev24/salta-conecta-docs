# Hallazgos (08-ui-state)

- **[SSR-safety]**
  - `contexts/GoogleMapsContext.tsx` usa `'use client'` y `useLoadScript` con `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY`; no accede a `window` directamente. Seguro en SSR si sólo se consume en cliente.
  - `hooks/use-sse.ts` y `hooks/use-driver-location.ts` tienen `'use client'` y limpiezas (`EventSource.close()`, `clearWatch`) correctas. Bien para evitar fugas.
  - Tiendas `zustand` (`store/cart-store.ts`, `store/wishlist-store.ts`, `store/auth-store.ts`) no declaran `'use client'` (OK). Consumidores deben ser Client Components. `persist` implica `localStorage`; usar sólo en cliente.

- **[Patrones de estado/contexto]**
  - `contexts/LoadingContext.tsx` provee `withLoading()` y granularidad por claves. Patrón claro y reusable.
  - `contexts/NotificationContext.tsx` integra Socket y toasts; límite en memoria de 50 notificaciones. Muchos selectores derivados via `useCallback`.
  - `GoogleMapsContext` expone `isLoaded`, `loadError` y `map`, patrón minimalista y correcto.

- **[Performance]**
  - `NotificationContext`: `unreadCount` se recalcula en cada render (`filter`). Podría memoizarse con `useMemo` (O(n)). Filtrados por categoría/prioridad recrean arrays frecuentemente; aceptable por tamaño limitado (<=50) pero documentar.
  - `components/chat/support-chat.tsx`: renderiza lista en `ScrollArea` sin virtualización; volumen pequeño, ok. Auto-scroll en `useEffect`; consulta el DOM por selector de Radix.
  - `store/cart-store.ts`: derived getters calculan `subtotal/discount/total` cada acceso. Coste O(n) sobre `items`. Aceptable por tamaño de carrito típico, considerar memoizar calculados costosos.

- **[A11y]**
  - `support-chat.tsx`: botones sólo ícono (`Minimize2`, `MessageSquare`, `Send`) sin `aria-label`. Input sin `label` visible; placeholder no sustituye label. Badges y estados "En línea" OK visual, considerar `aria-live` para mensajes nuevos.
  - `free-shipping-progress.tsx`: barra de progreso sin atributos `role="progressbar"`, `aria-valuenow/min/max`; buen texto auxiliar presente.
  - `shipping-calculator.tsx`: placeholder, falta encabezados/labels semánticos al implementar.

- **[Manejo de efectos y limpieza]**
  - `NotificationContext`: agrega/remueve listeners de Socket correctamente; `joinRoom/leaveRoom` por usuario y rol. Bien.
  - `use-sse.ts`: limpia `EventSource`; marca desconectado en error/cierre.
  - `use-driver-location.ts`: cleanup en `unmount` y `stopTracking`; actualiza estado con `Date.now()`.

- **[Cohesión y separación]**
  - Contextos separados por responsabilidad (Loading, Notifications, Google Maps) favorecen composición.
  - Tiendas `zustand` separan carrito/deseos/autenticación. Persistencia parcial en `wishlist` (`partialize`).

- **[Oportunidades]**
  - Añadir polyfills/guards en hooks si se importan accidentalmente en SSR (ej. chequear `typeof window !== 'undefined'`).
  - Estándar de A11y para componentes de UI (labels, aria-attrs, roles, focus states). Añadir `aria-live="polite"` para feed de chat/notificaciones.
  - Documentar patrón de consumo de stores en Client Components y evitar hidratos inconsistentes.
