# Evidencias (07-integrations)

- **MercadoPago – `lib/mercadopago.ts`**

```ts
5  accessToken: process.env.MERCADOPAGO_ACCESS_TOKEN || process.env.MP_ACCESS_TOKEN || ''
7    timeout: 5000,
121 back_urls: data.backUrls || {
122   success: `${process.env.NEXT_PUBLIC_APP_URL}/orders/${data.orderId}/success`,
136   notification_url: `${process.env.NEXT_PUBLIC_APP_URL}/api/mercadopago/webhook`,
```

- **Webhooks MP – `app/api/mercadopago/webhook/route.ts`**

```ts
794 const secret = process.env.NEXT_WEBHOOK_MERCADOPAGO_KEY_SECRET || process.env.MERCADOPAGO_WEBHOOK_SECRET || ''
798 const isValid = validateMpWebhookSignature(...)
```

- **Cloudinary – `lib/cloudinary.ts`**

```ts
5  cloudinary.config({
6    cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
7    api_key: process.env.CLOUDINARY_API_KEY,
8    api_secret: process.env.CLOUDINARY_API_SECRET,
})
48 export function isCloudinaryConfigured(): boolean { /* checks env */ }
```

- **Resend – `lib/resend.ts`**

```ts
4  const resend = new Resend(process.env.RESEND_API_KEY);
35 export function isEmailEnabled(): boolean {
36   return !!(process.env.RESEND_API_KEY && process.env.RESEND_FROM);
}
49 from: template.from || process.env.RESEND_FROM || 'no-reply@salta-conecta.app',
```

- **Redis/Socket.IO – `lib/redis.ts` y `pages/api/socket.ts`**

```ts
// lib/redis.ts
7  const url = process.env.REDIS_URL;
9  redis = new Redis(url, { maxRetriesPerRequest: 2, enableReadyCheck: true, lazyConnect: false });

// pages/api/socket.ts
31 cors: { origin: process.env.NEXT_PUBLIC_APP_URL || '*', methods: ['GET', 'POST'] },
42 const pubClient = redis.duplicate(); const subClient = pubClient.duplicate();
45 io.adapter(createAdapter(pubClient, subClient));
```

- **Google Maps – `next.config.js` (exposición)**

```js
100 NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
104 NEXT_PUBLIC_GOOGLE_MAPS_API_KEY: process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY,
```
