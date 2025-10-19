# Criterios de aceptación (05-data-models)

- **[Conexión Mongo]**
  - Opciones de conexión definidas y documentadas: timeouts, pool, `retryWrites`, `readPreference`.
  - `autoIndex` configurado según entorno (off en prod, on en dev).

- **[Índices]**
  - Índices compuestos clave creados y verificados con explain/profiler.
  - Índices únicos/sparse definidos donde corresponda.

- **[TTL]**
  - TTL activo en colecciones efímeras (e.g., `PaymentLog` 90d) y políticas de retención acordadas.

- **[Esquemas]**
  - Disminución de `Schema.Types.Mixed` a favor de subesquemas tipados.
  - Validaciones y límites aplicados en campos críticos.

- **[Lean()]**
  - Guía de uso definida y aplicada; no se rompen invariantes por `lean()`.

- **[Migraciones]**
  - Scripts de creación/actualización de índices disponibles y ejecutados en despliegue.
