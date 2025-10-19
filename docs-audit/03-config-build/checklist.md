# Checklist de verificación (03-config-build)

- **[CSP]** Revisar y ajustar `Content-Security-Policy` en `next.config.js` líneas 36-56 para remover `'unsafe-inline'` y `'unsafe-eval'` o justificarlas con Nonces/Hashes.
- **[ENV Build Gate]** Endurecer validación de env en `next.config.js` líneas 17-25 para fallar el build si faltan claves críticas en prod (`NEXT_PUBLIC_APP_URL`, `NEXTAUTH_URL`, `NEXTAUTH_SECRET`, `MONGODB_URI`, `JWT_SECRET`, `ENCRYPTION_KEY`, `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY`).
- **[CORS]** Revisar orígenes en `next.config.js` líneas 119-133 y 60-65. Reemplazar `*` en producción por dominios explícitos.
- **[Exposición de env]** Confirmar que solo `NEXT_PUBLIC_*` se exponen en `next.config.js` líneas 99-107 y que no contienen secretos.
- **[TS Flags]** Evaluar `allowJs` y `skipLibCheck` en `tsconfig.json` líneas 9-13. Desactivar si no son necesarios.
- **[Aliases]** Alinear aliases de `tsconfig.json` (líneas 25-71) con `jest.config.js` `moduleNameMapper` y tooling (TS/ESLint).
- **[ESLint]** Reconsiderar `no-console: off` en `.eslintrc.json` líneas 6-12. Forzar `error` en producción o usar `console` por niveles.
- **[Jest]** Validar `coverageThreshold` 70% (`jest.config.js` 41-47) y `collectCoverageFrom` para incluir/excluir paths relevantes.
- **[Docker Secrets]** Mover secretos de `docker-compose.yml` líneas 8-14 a `env_file` o variables seguras. Añadir `healthcheck` y límites de recursos.
- **[Mongo/Redis Puertos]** Confirmar si es necesario exponer `27017` y `6379` a host (`docker-compose.yml` líneas 20-29). Limitar a red interna si es posible.
- **[Vercel Functions]** Revisar `vercel.json` (líneas 3-5) y considerar `memory`, `regions` o `maxDuration` adecuados para rutas críticas.
- **[ENV Alias MP]** Consolidar alias MP (`env.example` 36-43). Elegir una sola convención y eliminar duplicados.
