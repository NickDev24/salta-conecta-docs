# Matriz de riesgos (07-integrations)

| ID  | Riesgo                                      | Evidencia                                                             | Severidad | Impacto | Esfuerzo |
| --- | ------------------------------------------- | --------------------------------------------------------------------- | --------- | ------- | -------- |
| I1  | Preferencias MP duplicadas en reintentos    | `app/api/orders/[id]/retry-payment/route.ts` 59-90                    | High      | Alto    | Medio    |
| I2  | Webhook MP depende de múltiples servicios   | `app/api/mercadopago/webhook/route.ts`                                | Medium    | Alto    | Medio    |
| I3  | Claves Cloudinary expuestas accidentalmente | `lib/cloudinary.ts` (usa server env); `NEXT_PUBLIC_*` solo cloud name | High      | Alto    | Bajo     |
| I4  | Resend sin validación de webhooks           | `.env.example` sugiere `RESEND_WEBHOOK_SECRET`                        | Medium    | Medio   | Bajo     |
| I5  | CORS Socket.IO permisivo si falta env       | `pages/api/socket.ts` 31                                              | Medium    | Medio   | Bajo     |
| I6  | API key de Maps sin restricciones           | `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` expuesta                            | Medium    | Alto    | Bajo     |
