# Evidencias (09-security-env)

- **Validación de envs en build – `next.config.js`**

```js
17 if (process.env.NODE_ENV === 'production') {
19   const missingVars = requiredEnvVars.filter(key => !process.env[key]);
21   console.warn('[next.config] Missing environment variables detected (build will continue):');
```

- **Exposición de env al cliente – `next.config.js`**

```js
99  env: {
101   NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
104   NEXT_PUBLIC_GOOGLE_MAPS_API_KEY: process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY,
```

- **Headers y CSP – `next.config.js`**

```js
27 const securityHeaders = [
36   key: 'Content-Security-Policy',
40   "script-src 'self' 'unsafe-inline' 'unsafe-eval' https://maps.googleapis.com ...",
112 async headers() { return [{ source: '/(.*)', headers: [...securityHeaders, { key: 'Access-Control-Allow-Origin', value: process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000' }, ...] }, { source: '/api/:path*', headers: [{ key: 'Access-Control-Allow-Origin', value: process.env.NEXT_PUBLIC_BASE_URL || process.env.NEXT_PUBLIC_APP_URL || '*' }, ...] }] }
```

- **HTTPS/HSTS y nonce – `middleware.ts`**

```ts
60 response.headers.set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
64 if (shouldForceHttps && request.headers.get('x-forwarded-proto') !== 'https') { ... redirect }
51 response.headers.set('x-nonce', nonce);
```

- **Mongo env fail-fast – `lib/mongodb.ts`**

```ts
13 if (!MONGODB_URI) throw new Error('MONGODB_URI is not set');
```

- **Rate limiting ausente – búsqueda**

```
rg -n "rate limit|rate-limit|ratelimit|limiter|brute|slowdown" -- **/*.{ts,tsx,js}
# sin resultados
```
