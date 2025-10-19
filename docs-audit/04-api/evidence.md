# Evidencias (`app/api/**/route.ts`)

- **`app/api/admin/analytics/route.ts`**

```ts
12 export async function GET(_req: NextRequest) {
14   const { user } = await requireAdmin(_req as any);
15   await dbConnect();
...
291 } catch (error) {
296   return NextResponse.json({ error: 'Error interno del servidor' }, { status: 500 });
```

- **`app/api/account/addresses/route.ts`**

```ts
9  const session = await getServerSession(authOptions)
11 if (!session || session.user.role !== 'client') {
...
55 // Validar campos requeridos
...
32 } catch (error) {
35   { error: 'Error interno del servidor' },
```

- **`app/api/mercadopago/webhook/route.ts`**

```ts
117 // Idempotencia general por paymentId + status
120 const gotGeneralLock = await acquireLock(generalLockKey)
139 if (existingLog) { return { ok: true, reason: 'duplicate_event' } }
...
789 // Validación de firma de MercadoPago
798 const isValid = validateMpWebhookSignature(...)
```

- **`app/api/orders/[id]/retry-payment/route.ts`**

```ts
15 const RetryPaymentSchema = z.object({ method: z.enum(['mercadopago', 'cash']).default('mercadopago') }).optional()
27 const session = await getServerSession(authOptions)
39 if (!order) { return NextResponse.json({ error: 'Order not found' }, { status: 404 }) }
60 const preference = await createMercadoPagoPreference({ ... })
91 } catch (error) { return NextResponse.json({ error: 'Internal server error' }, { status: 500 }) }
```

- **`app/api/orders/[id]/confirm-delivery/route.ts`**

```ts
7  const ConfirmDeliverySchema = z.object({ driverId: z.string().optional() })
15 const session = await getServerSessionFromRequest(request)
29 // Only assigned driver or supplier can mark delivered
31 if (order.assignedDriverId && order.assignedDriverId.toString() !== actorId) { return NextResponse.json({ error: 'Forbidden' }, { status: 403 }) }
49 } catch (err) { return NextResponse.json({ error: ... }, { status: 500 }) }
```

- **`app/api/drivers/connect/route.ts`**

```ts
8  const DriverConnectSchema = z.object({ userId: z.string(), action: z.enum(['connect', 'disconnect']) })
23 await connectToDatabase()
25 const parsed = DriverConnectSchema.safeParse(body)
29 let driver = await Driver.findOne({ userId })
54 assignedRoute = await Route.findOneAndUpdate({ status: 'pending', vehicleType: { $in: preferredTypes } }, { $set: { status: 'assigned', driverId: driver._id } }, { new: true })
76 return NextResponse.json({ message: 'ok', driver, assignedRoute })
```

- **`pages/api/socket.ts`**

```ts
28 const io = new SocketIOServer(httpServer, { path: '/api/socket', ... })
40 const redis = getRedis();
45 io.adapter(createAdapter(pubClient, subClient));
```
