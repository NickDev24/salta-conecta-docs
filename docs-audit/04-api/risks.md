# Matriz de riesgos (04-api)

| ID  | Riesgo                                                | Evidencia                                          | Severidad | Impacto | Esfuerzo |
| --- | ----------------------------------------------------- | -------------------------------------------------- | --------- | ------- | -------- |
| A1  | Endpoint sin auth modifica estado (`drivers/connect`) | `app/api/drivers/connect/route.ts` 21-37, 52-73    | High      | Alto    | Bajo     |
| A2  | Validación manual inconsistente                       | `app/api/account/addresses/route.ts` 55-61         | Medium    | Medio   | Bajo     |
| A3  | Falta de idempotencia en reintentos de pago           | `app/api/orders/[id]/retry-payment/route.ts` 59-90 | High      | Alto    | Medio    |
| A4  | Autorización por rol con strings                      | `orders/[id]/confirm-delivery` 29-40               | Medium    | Medio   | Bajo     |
| A5  | Manejo de errores no estandarizado (mensajes)         | Varias rutas, retornar mensajes internos           | Low       | Bajo    | Bajo     |
| A6  | Webhook crítico depende de múltiples servicios        | `mercadopago/webhook` (Resend/DB/WS/locks)         | Medium    | Alto    | Medio    |
