# 🏛️ Modelo: Auction

## 📋 Información General

**Ubicación:** `models/Auction.ts`
**Tipo:** Modelo Mongoose para subastas
**Propósito:** Gestión completa del sistema de subastas en tiempo real

## 🔧 Estructura del Modelo

### Interface Principal

```typescript
export interface IAuction {
  title: string;
  description: string;
  productId: Schema.Types.ObjectId;
  basePrice: number;
  currentPrice: number;
  minIncrement: number;
  status:
    | 'draft'
    | 'active'
    | 'paused'
    | 'ended'
    | 'cancelled'
    | 'finished'
    | 'scheduled';
  startsAt: Date;
  endsAt: Date;
  endDate?: Date;
  participants: {
    userId: Schema.Types.ObjectId;
    registered: boolean;
    registeredAt: Date;
    guaranteeHeld?: number;
  }[];
  bids: IAuctionBid[];
  lastBidUserId?: Schema.Types.ObjectId;
  lastBidAmount?: number;
  winnerId?: Schema.Types.ObjectId;
  winnerAmount?: number;
  providerId?: Schema.Types.ObjectId;
  paymentSettings?: {
    subscriptionPrice?: number;
    guaranteePercent?: number;
  };
  paymentStatus?: 'pending_payment' | 'paid' | 'failed' | 'cancelled';
  mercadoPagoId?: string;
  paidAt?: Date;
  bannerId?: string;
  createdBy: Schema.Types.ObjectId;
  updatedBy?: Schema.Types.ObjectId;
}
```

### Bids Interface

```typescript
interface IAuctionBid {
  userId: Schema.Types.ObjectId;
  amount: number;
  timestamp: Date;
  status: 'active' | 'cancelled' | 'won' | 'lost';
}
```

## 📊 Campos del Modelo

### Información Básica

| Campo         | Tipo     | Requerido | Descripción                       |
| ------------- | -------- | --------- | --------------------------------- |
| `title`       | String   | ✅        | Título de la subasta              |
| `description` | String   | ✅        | Descripción detallada             |
| `productId`   | ObjectId | ✅        | Referencia al producto en subasta |

### Precios y Pujas

| Campo           | Tipo   | Requerido | Descripción                   |
| --------------- | ------ | --------- | ----------------------------- |
| `basePrice`     | Number | ✅        | Precio base de la subasta     |
| `currentPrice`  | Number | ✅        | Precio actual (mejor puja)    |
| `minIncrement`  | Number | ✅        | Incremento mínimo entre pujas |
| `lastBidAmount` | Number | ❌        | Monto de la última puja       |

### Estado y Fechas

| Campo      | Tipo | Requerido | Descripción                 |
| ---------- | ---- | --------- | --------------------------- |
| `status`   | Enum | ✅        | Estado actual de la subasta |
| `startsAt` | Date | ✅        | Fecha de inicio             |
| `endsAt`   | Date | ✅        | Fecha de finalización       |
| `endDate`  | Date | ❌        | Fecha real de cierre        |

### Participantes y Ganador

| Campo          | Tipo     | Requerido | Descripción                     |
| -------------- | -------- | --------- | ------------------------------- |
| `participants` | Array    | ❌        | Lista de usuarios registrados   |
| `winnerId`     | ObjectId | ❌        | Usuario ganador de la subasta   |
| `winnerAmount` | Number   | ❌        | Monto final de la puja ganadora |

### Información de Proveedor

| Campo        | Tipo     | Requerido | Descripción                      |
| ------------ | -------- | --------- | -------------------------------- |
| `providerId` | ObjectId | ❌        | Proveedor que creó la subasta    |
| `createdBy`  | ObjectId | ✅        | Usuario que creó la subasta      |
| `updatedBy`  | ObjectId | ❌        | Usuario que actualizó la subasta |

## 🚀 Características Avanzadas

### Estados de Subasta

```typescript
type AuctionStatus =
  | 'draft' // Borrador, no visible
  | 'scheduled' // Programada para iniciar
  | 'active' // Activa y aceptando pujas
  | 'paused' // Pausada temporalmente
  | 'ended' // Finalizada normalmente
  | 'cancelled' // Cancelada por el proveedor
  | 'finished'; // Procesada completamente
```

### Sistema de Participantes

```typescript
participants: [
  {
    userId: { type: Schema.Types.ObjectId, ref: 'User', required: true },
    registered: { type: Boolean, default: false },
    registeredAt: { type: Date, default: Date.now },
    guaranteeHeld: { type: Number, default: 0 }, // Garantía retenida
  },
];
```

### Historial de Pujas

```typescript
bids: [
  {
    userId: { type: Schema.Types.ObjectId, ref: 'User', required: true },
    amount: { type: Number, required: true },
    timestamp: { type: Date, default: Date.now },
    status: {
      type: String,
      enum: ['active', 'cancelled', 'won', 'lost'],
      default: 'active',
    },
  },
];
```

## 🔧 Índices de Performance

### Índices Configurados

```typescript
// Índice por estado para consultas eficientes
AuctionSchema.index({ status: 1 });

// Índice compuesto para búsquedas por estado y fechas
AuctionSchema.index({ status: 1, startsAt: 1, endsAt: 1 });

// Índices específicos para búsquedas frecuentes
AuctionSchema.index({ productId: 1 });
AuctionSchema.index({ providerId: 1 });
AuctionSchema.index({ winnerId: 1 });
```

## 📊 Validaciones

### Validaciones de Schema

```typescript
const AuctionSchema = new Schema<IAuction>({
  title: { type: String, required: true },
  description: { type: String, required: true },
  productId: { type: Schema.Types.ObjectId, ref: 'Product', required: true },
  basePrice: { type: Number, required: true, min: 0 },
  currentPrice: { type: Number, required: true, min: 0 },
  minIncrement: { type: Number, required: true, min: 0 },
  status: {
    type: String,
    enum: [
      'draft',
      'active',
      'paused',
      'ended',
      'cancelled',
      'finished',
      'scheduled',
    ],
    default: 'draft',
    index: true,
  },
  // ... resto de validaciones
});
```

## 🚀 Métodos de Instancia

### Gestión de Pujas

```typescript
// Verificar si un usuario puede pujar
auction.canUserBid(userId: string): boolean

// Agregar nueva puja
auction.addBid(userId: string, amount: number): Promise<void>

// Actualizar precio actual
auction.updateCurrentPrice(amount: number): void
```

### Gestión de Participantes

```typescript
// Registrar participante
auction.registerParticipant(userId: string): Promise<void>

// Verificar si usuario está registrado
auction.isParticipantRegistered(userId: string): boolean
```

## 📈 Estadísticas y Métricas

### Información Derivada

```typescript
// Calcular duración de la subasta
auction.getDuration(): number // en milisegundos

// Obtener número total de pujas
auction.getTotalBids(): number

// Obtener número de participantes activos
auction.getActiveParticipants(): number
```

## 🔒 Seguridad y Permisos

### Validaciones de Acceso

```typescript
// Verificar si usuario puede modificar la subasta
auction.canUserModify(userId: string): boolean

// Verificar si usuario puede ver la subasta
auction.canUserView(userId: string): boolean
```

## 💾 Persistencia

### Configuración de Colección

```typescript
{
  timestamps: true,
  collection: 'auctions'
}
```

### Opciones de Modelo

```typescript
const Auction = (models.Auction ||
  model<IAuction>('Auction', AuctionSchema)) as Model<IAuction> & AuctionModel;
```

## 🔧 Próximas Mejoras

- [ ] **Sistema de garantías:** Gestión automática de depósitos de garantía
- [ ] **Subastas programadas:** Sistema avanzado de scheduling
- [ ] **Métricas avanzadas:** Estadísticas detalladas de participación
- [ ] **Sistema de notificaciones:** Notificaciones push para eventos importantes

---

_Última modificación: Octubre 2025_
_Estado: ✅ Completamente funcional y documentado_
