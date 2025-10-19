# Recomendaciones (opciones A/B)

- **[CSP endurecida]**
  - Opción A: Remover `'unsafe-inline'` y `'unsafe-eval'`; usar Nonces/Hashes para scripts críticos.
    - Pros: Mayor protección XSS; cumplimiento estricto.
    - Contras: Requiere ajustar inyecciones y librerías de terceros (SDKs MP/Maps).
  - Opción B: Mantener `'unsafe-inline'` pero limitar a dominios estrictos y evaluar `strict-dynamic`.
    - Pros: Menor esfuerzo; compatibilidad inmediata.
    - Contras: Riesgo XSS residual.

- **[Validación de env en build]**
  - Opción A: Fallar el build si faltan variables críticas (throw error).
    - Pros: Falla temprano; despliegues consistentes.
    - Contras: Puede bloquear builds hasta configurar entornos.
  - Opción B: Mantener warning pero agregar chequeo en CI (job que valida envs).
    - Pros: Flexibilidad; control en pipeline.
    - Contras: Riesgo de pasar desapercibido si CI no es estricto.

- **[CORS]**
  - Opción A: Definir `ALLOWED_ORIGINS` y derivar CORS dinámico por env; sin `*` en prod.
    - Pros: Control granular; menor superficie de ataque.
    - Contras: Exige mantener listas por entorno.
  - Opción B: Un solo dominio base por entorno (`NEXT_PUBLIC_BASE_URL`).
    - Pros: Simplicidad.
    - Contras: Menos flexible con múltiples orígenes legítimos.

- **[Docker Secrets]**
  - Opción A: Mover secretos a `env_file` y usar `secrets`/`configs` (Swarm/K8s) y `healthcheck` + `resources.limits`.
    - Pros: Seguridad y resiliencia.
    - Contras: Cambios en infra y orquestación.
  - Opción B: `.env` montado y variables por entorno, manteniendo Compose simple.
    - Pros: Bajo esfuerzo.
    - Contras: Secretos en host; riesgo si no hay controles.

- **[TypeScript flags]**
  - Opción A: Desactivar `allowJs` y `skipLibCheck` (o limitar por paths).
    - Pros: Calidad de tipado consistente.
    - Contras: Puede descubrir errores latentes.
  - Opción B: Mantenerlos pero con `type-check` en CI (`tsc --noEmit`).
    - Pros: Balancea performance/seguridad.
    - Contras: Errores pueden filtrarse si exclusiones son amplias.

- **[Logs/ESLint]**
  - Opción A: `no-console` como `warn`/`error` condicionado por `NODE_ENV` (rule override).
    - Pros: Evita logs verbosos en prod.
    - Contras: Necesita configuración condicional.
  - Opción B: Mantener `no-console: off` pero estandarizar wrapper `logger`.
    - Pros: Menor fricción.
    - Contras: Logs incontrolados en prod.

- **[Vercel functions]**
  - Opción A: Definir `maxDuration` por ruta crítica y `regions`/`memory` específicas.
    - Pros: Control de performance/costos.
    - Contras: Mayor mantenimiento.
  - Opción B: Mantener global 30s y monitorear; ajustar por outliers.
    - Pros: Simple.
    - Contras: Puede ser insuficiente para tareas intensivas.

- **[Aliases/Tooling]**
  - Opción A: Alinear `tsconfig paths` con `jest.moduleNameMapper` exhaustivo.
    - Pros: DX consistente.
    - Contras: Requiere mantener mapeo doble.
  - Opción B: Reducir alias a `@/*` y eliminar específicos.
    - Pros: Menos mantenimiento.
    - Contras: Import paths más largos en algunos casos.
