# Evidencias (fragmentos con rutas y líneas)

- \*\*`next.config.js` líneas 36-44 (CSP con inline/eval):

```js
36     key: 'Content-Security-Policy',
37     value: [
38       "default-src 'self'",
39       // Scripts permitidos: Google Maps + Mercado Pago SDK/estáticos
40       "script-src 'self' 'unsafe-inline' 'unsafe-eval' https://maps.googleapis.com https://maps.gstatic.com https://sdk.mercadopago.com https://*.mercadopago.com https://*.mlstatic.com",
41       // Estilos: Google Fonts + Maps (igual que antes)
42       "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://maps.googleapis.com",
43       // Imágenes: mantener
44       "img-src 'self' data: https: blob:",
```

- \*\*`next.config.js` líneas 17-23 (validación de env en build):

```js
17 // Validate environment variables
18 if (process.env.NODE_ENV === 'production') {
19   const missingVars = requiredEnvVars.filter(key => !process.env[key]);
20   if (missingVars.length > 0) {
21     console.warn('[next.config] Missing environment variables detected (build will continue):');
22     missingVars.forEach(key => console.warn(`  - ${key}`));
23     // Do not exit the process; fail-fast will happen at runtime where applicable
```

- \*\*`next.config.js` líneas 120-131 (CORS headers):

```js
120   { key: 'Access-Control-Allow-Origin', value: process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000' },
...
126   source: '/api/:path*',
129   { key: 'Access-Control-Allow-Origin', value: process.env.NEXT_PUBLIC_BASE_URL || process.env.NEXT_PUBLIC_APP_URL || '*' },
130   { key: 'Access-Control-Allow-Methods', value: 'GET,OPTIONS,PATCH,DELETE,POST,PUT' },
131   { key: 'Access-Control-Allow-Headers', value: 'X-Requested-With, X-HTTP-Method-Override, Content-Type, Accept, Authorization' },
```

- \*\*`next.config.js` líneas 99-107 (env expuestos al cliente):

```js
99   env: {
100     NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
101     NEXT_PUBLIC_BASE_URL: process.env.NEXT_PUBLIC_BASE_URL,
102     NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
103     NEXT_PUBLIC_GOOGLE_MAPS_API_KEY: process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY,
104     NEXT_PUBLIC_MERCADOPAGO_PUBLIC_KEY: process.env.NEXT_PUBLIC_MERCADOPAGO_PUBLIC_KEY,
105     NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME: process.env.NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME,
106   },
```

- \*\*`tsconfig.json` líneas 9-16 (TS flags):

```json
9     "allowJs": true,
10     "skipLibCheck": true,
11     "strict": true,
12     "noEmit": true,
13     "esModuleInterop": true,
14     "module": "esnext",
15     "moduleResolution": "bundler",
16     "resolveJsonModule": true,
```

- \*\*`.eslintrc.json` líneas 6-12 (reglas):

```json
6     "@next/next/no-img-element": "off",
7     "no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
8     "no-console": "off",
9     "no-debugger": "warn",
10     "prefer-const": "error",
11     "import/no-anonymous-default-export": "warn"
```

- \*\*`jest.config.js` líneas 41-50 (cobertura y timeout):

```js
41   coverageThreshold: {
42     global: {
43       branches: 70,
44       functions: 70,
45       lines: 70,
46       statements: 70
47     }
48   },
49   moduleDirectories: ['node_modules', '<rootDir>/'],
50   testTimeout: 10000
```

- \*\*`docker-compose.yml` líneas 8-14 (entorno de app):

```yaml
8     environment:
9       - NODE_ENV=production
10       - MONGODB_URI=mongodb://mongo:27017/saltaconecta
11       - REDIS_URL=redis://redis:6379
12       - NEXTAUTH_SECRET=your-secret
13       - MERCADOPAGO_ACCESS_TOKEN=your-token
```

- \*\*`docker-compose.yml` líneas 18-31 (servicios y volúmenes):

```yaml
18   mongo:
19     image: mongo:7.0
---
25   redis:
26     image: redis:7.2-alpine
---
32 volumes:
33   mongo_data:
34   redis_data:
```

- \*\*`vercel.json` líneas 3-5 (funciones):

```json
3   "functions": {
4     "app/**/route.ts": { "maxDuration": 30 },
5     "app/**/route.js": { "maxDuration": 30 }
```

- \*\*`.env.example` líneas 10-13 (NextAuth):

```ini
10 # ===== NextAuth =====
11 NEXTAUTH_URL=http://localhost:3000
12 NEXTAUTH_SECRET=changeme-in-production
```

- \*\*`env.example` líneas 36-43 (MercadoPago alias):

```ini
36 MERCADOPAGO_ACCESS_TOKEN=your-mercadopago-access-token
37 MP_ACCESS_TOKEN=your-mercadopago-access-token
...
39 MERCADOPAGO_APPLICATION_ID=your-mercadopago-application-id
40 MP_APPLICATION_ID=your-mercadopago-application-id
41 MERCADOPAGO_WEBHOOK_SECRET=your-mercadopago-webhook-secret
42 NEXT_WEBHOOK_MERCADOPAGO_KEY_SECRET=your-mercadopago-webhook-secret
```
