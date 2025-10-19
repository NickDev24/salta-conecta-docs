# Matriz de riesgos (05-data-models)

| ID  | Riesgo                                                   | Evidencia                                                                      | Severidad | Impacto | Esfuerzo |
| --- | -------------------------------------------------------- | ------------------------------------------------------------------------------ | --------- | ------- | -------- |
| D1  | Conexión Mongo sin timeouts/retries/pool tuning          | `lib/mongodb.ts` 11-19                                                         | High      | Alto    | Bajo     |
| D2  | TTL no aplicado en logs de pago                          | `models/PaymentLog.ts` 70-71 (comentado)                                       | Medium    | Medio   | Bajo     |
| D3  | `Schema.Types.Mixed` extensivo sin subesquemas           | `Order.paymentDetails`, `Notification.metadata`, `PaymentLog.request/response` | Medium    | Medio   | Medio    |
| D4  | Índices potencialmente faltantes para queries frecuentes | `Order` (combinaciones adicionales), otros modelos                             | Medium    | Medio   | Medio    |
| D5  | `.lean()` mal aplicado podría romper invariantes         | Uso disperso (`grep .lean(`)                                                   | Low       | Medio   | Bajo     |
