# Alcance y Reglas de Auditoría

## Alcance

- **Código**: `app/`, `pages/`, `components/`, `lib/`, `models/`, `hooks/`, `contexts/`.
- **APIs**: `app/api/` y `pages/api/`.
- **Configuración**: `.env*`, `next.config.js`, `docker-compose.yml`, `vercel.json`.
- **Infra/DevOps**: scripts de `package.json`, `docker-compose.yml`.
- **Pruebas**: `__tests__/`.

## Reglas

- **Sin cambios de código**: No modificar archivos fuera de `docs-audit/`.
- **Formato de evidencias**:
  - Referenciar con rutas y símbolos: `ruta/archivo.ts`, `miFuncion()`, `MiClase.Metodo()`.
  - Incluir fragmentos mínimos necesarios y contexto.
  - CSVs en `docs-audit/02-inventory/`.
- **Severidad**:
  - **Critical**: riesgo inmediato de explotación o caída total (exposición de secretos, RCE, auth rota).
  - **High**: afecta seguridad/negocio significativamente (IDOR, autorización insuficiente, pagos).
  - **Medium**: impacta confiabilidad/mantenibilidad (errores de validación, manejo de errores débil).
  - **Low**: mejoras menores (estilo, micro-optimizaciones).
- **Impacto**: alcance del efecto en usuarios/negocio/sistemas (Alto/Medio/Bajo).
- **Esfuerzo**: complejidad para remediar (Alto/Medio/Bajo) considerando cambios, pruebas y despliegue.
