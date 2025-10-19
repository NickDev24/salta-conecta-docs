# Evidencias (06-frontend-middleware)

- **`middleware.ts` – matcher/scope**

```ts
6 export const config = {
7   matcher: [
16     '/((?!api|_next/static|_next/image|favicon.ico|public/).*)',
17   ],
18 };
```

- **`middleware.ts` – HTTPS/HSTS**

```ts
53 // Redirect HTTP to HTTPS in production
60 if (shouldForceHttps) {
61   response.headers.set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
62 }
64 if (shouldForceHttps && request.headers.get('x-forwarded-proto') !== 'https') {
67   return NextResponse.redirect(httpsUrl, 301);
}
```

- **`middleware.ts` – RBAC y next-auth/jwt**

```ts
86 const needsAdmin = pathname.startsWith('/admin');
89 if (needsAdmin || needsSupplier) {
93   const getToken = (mod as any).getToken as (...)
95   const token = await getToken({ req: request as any, secret });
97   const isAuthorized = !!token && (
98     (needsAdmin && token.role === USER_ROLES.ADMIN) ||
99     (needsSupplier && token.role === USER_ROLES.SUPPLIER)
101 );
102 if (!isAuthorized) {
105   return NextResponse.redirect(loginUrl);
}
```

- **`middleware.ts` – headers seguridad y nonce**

```ts
41 response.headers.set('X-Content-Type-Options', 'nosniff');
43 response.headers.set('X-Frame-Options', 'DENY');
45 response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
46 response.headers.set('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
51 response.headers.set('x-nonce', nonce);
```

- **`middleware.ts` – bots**

```ts
71 const badBots = [ 'ahrefsbot', 'semrushbot', ... ];
79 if (isBadBot) {
81   return new NextResponse('Access denied', { status: 403 });
}
```
