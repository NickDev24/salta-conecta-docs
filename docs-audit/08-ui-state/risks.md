# Matriz de riesgos (08-ui-state)

| ID  | Riesgo                                 | Evidencia                                           | Severidad | Impacto | Esfuerzo |
| --- | -------------------------------------- | --------------------------------------------------- | --------- | ------- | -------- |
| U1  | Botones icon-only sin `aria-label`     | `components/chat/support-chat.tsx` 115-123, 171-185 | Medium    | Medio   | Bajo     |
| U2  | Barra de progreso sin atributos ARIA   | `components/cart/free-shipping-progress.tsx` 33-37  | Low       | Bajo    | Bajo     |
| U3  | Recalculo no memoizado en listas       | `NotificationContext` (unreadCount/filter)          | Low       | Medio   | Bajo     |
| U4  | Uso de `persist` en SSR accidental     | `store/*-store.ts` con `persist`                    | Medium    | Medio   | Bajo     |
| U5  | Hooks cliente importables desde server | `use-sse.ts`, `use-driver-location.ts`              | Low       | Medio   | Bajo     |
