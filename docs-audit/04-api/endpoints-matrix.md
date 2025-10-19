# Resumen humano de endpoints (muestra representativa)

- **/app/api/admin/analytics**
  - **Métodos**: GET
  - **Auth/Roles**: Sí (`requireAdmin()`)
  - **Validación**: No-Zod
  - **Usa**: DB
  - **Errores**: try/catch
  - **Idempotencia**: No

- **/app/api/account/addresses**
  - **Métodos**: GET, POST
  - **Auth/Roles**: Sí (`getServerSession`, rol `client`)
  - **Validación**: Manual
  - **Usa**: DB
  - **Errores**: try/catch
  - **Idempotencia**: No

- **/app/api/mercadopago/webhook**
  - **Métodos**: GET, POST
  - **Auth/Roles**: No (webhook)
  - **Validación**: Firma MP
  - **Usa**: DB, Resend, MP, WebSocket
  - **Errores**: try/catch
  - **Idempotencia**: Sí (locks + logs)

- **/app/api/orders/[id]/retry-payment**
  - **Métodos**: POST
  - **Auth/Roles**: Sí (sesión + owner/admin)
  - **Validación**: Zod opcional
  - **Usa**: DB, MP
  - **Errores**: try/catch
  - **Idempotencia**: No

- **/app/api/orders/[id]/confirm-delivery**
  - **Métodos**: POST
  - **Auth/Roles**: Sí (sesión + driver/supplier)
  - **Validación**: Zod
  - **Usa**: DB
  - **Errores**: try/catch
  - **Idempotencia**: No

- **/app/api/drivers/connect**
  - **Métodos**: POST
  - **Auth/Roles**: No (debería exigir auth driver)
  - **Validación**: Zod
  - **Usa**: DB
  - **Errores**: try/catch
  - **Idempotencia**: No

Nota: El CSV completo se mantiene en `docs-audit/02-inventory/endpoints-matrix.csv`. Este resumen destaca patrones y riesgos.
