# Evidencias (models/\*\*, Mongo)

- **`lib/mongodb.ts`**

```ts
11 if (!cached) {
12   cached = (global as MongooseGlobal)._mongooseConn = (async () => {
13     if (!MONGODB_URI) throw new Error('MONGODB_URI is not set');
15     await mongoose.connect(MONGODB_URI, {
16       dbName: process.env.MONGODB_DB || undefined,
17     });
```

- **`models/Order.ts`**

```ts
171 }, { timestamps: true });
173 // Índices para performance
174 OrderSchema.index({ orderNumber: 1 });
175 OrderSchema.index({ customer: 1, createdAt: -1 });
176 OrderSchema.index({ supplier: 1, status: 1, createdAt: -1 });
177 OrderSchema.index({ status: 1, paymentStatus: 1 });
178 OrderSchema.index({ assignedDriverId: 1, status: 1 });
179 OrderSchema.index({ 'shippingAddress.city': 1 });
182 OrderSchema.pre('save', function(next) { ... })
```

- **`models/Notification.ts`**

```ts
92 }, {
93   timestamps: true
94 });
96 // Índices para performance
100 NotificationSchema.index({ expiresAt: 1 }, { expireAfterSeconds: 0 }); // TTL para notificaciones expiradas
```

- **`models/PaymentLog.ts`**

```ts
61 }, {
62   timestamps: true
63 });
65 PaymentLogSchema.index({ type: 1, success: 1, createdAt: -1 });
66 PaymentLogSchema.index({ referenceId: 1, type: 1 });
67 PaymentLogSchema.index({ order: 1, type: 1, createdAt: -1 });
70 // Índice TTL para limpiar logs antiguos (opcional)
71 // PaymentLogSchema.index({ createdAt: 1 }, { expireAfterSeconds: 90 * 24 * 60 * 60 }); // 90 días
```

- **`models/DriverLocationHistory.ts`**

```ts
29 }, {
30   timestamps: true
31 });
34 DriverLocationHistorySchema.index({ driverId: 1, recordedAt: -1 });
35 DriverLocationHistorySchema.index({ userId: 1, recordedAt: -1 });
```
