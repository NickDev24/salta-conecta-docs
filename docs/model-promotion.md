# 🎫 Modelo: Promotion

## 📋 Información General

**Ubicación:** `models/Promotion.ts`
**Tipo:** Modelo Mongoose para promociones y cupones
**Propósito:** Gestión completa del sistema de promociones y descuentos

## 🔧 Estructura del Modelo

### Interface Principal

```typescript
export interface IPromotion {
  name: string;
  title: string;
  description: string;
  type: 'percentage' | 'fixed_amount' | 'free_shipping' | 'buy_x_get_y';
  value: number; // porcentaje o monto
  minAmount?: number;
  minOrderValue?: number;
  maxDiscount?: number;
  code?: string; // código de cupón
  isActive: boolean;
  startDate: Date;
  endDate: Date;
  usageLimit?: number;
  usedCount: number;
  applicableCategories?: mongoose.Schema.Types.ObjectId[];
  applicableProducts?: mongoose.Schema.Types.ObjectId[];
  applicableUsers?: mongoose.Schema.Types.ObjectId[];
  createdBy: mongoose.Schema.Types.ObjectId;
  createdAt: Date;
  updatedAt: Date;
}
```

## 📊 Campos del Modelo

### Información Básica

| Campo         | Tipo   | Requerido | Descripción                  |
| ------------- | ------ | --------- | ---------------------------- |
| `name`        | String | ✅        | Nombre único de la promoción |
| `title`       | String | ✅        | Título descriptivo           |
| `description` | String | ✅        | Descripción detallada        |

### Configuración del Descuento

| Campo           | Tipo   | Requerido | Descripción                |
| --------------- | ------ | --------- | -------------------------- |
| `type`          | Enum   | ✅        | Tipo de descuento          |
| `value`         | Number | ✅        | Valor del descuento        |
| `minAmount`     | Number | ❌        | Monto mínimo de compra     |
| `minOrderValue` | Number | ❌        | Valor mínimo del pedido    |
| `maxDiscount`   | Number | ❌        | Descuento máximo aplicable |

### Control de Uso

| Campo        | Tipo    | Requerido | Descripción             |
| ------------ | ------- | --------- | ----------------------- |
| `code`       | String  | ❌        | Código único de cupón   |
| `isActive`   | Boolean | ❌        | Estado de la promoción  |
| `usageLimit` | Number  | ❌        | Límite de usos          |
| `usedCount`  | Number  | ❌        | Número de usos actuales |

### Aplicabilidad

| Campo                  | Tipo       | Requerido | Descripción             |
| ---------------------- | ---------- | --------- | ----------------------- |
| `applicableCategories` | ObjectId[] | ❌        | Categorías donde aplica |
| `applicableProducts`   | ObjectId[] | ❌        | Productos específicos   |
| `applicableUsers`      | ObjectId[] | ❌        | Usuarios específicos    |

## 🚀 Tipos de Promoción

### Percentage (Porcentaje)

```typescript
{
  type: 'percentage',
  value: 15,           // 15% de descuento
  maxDiscount: 1000    // Máximo $1000 de descuento
}
```

### Fixed Amount (Monto Fijo)

```typescript
{
  type: 'fixed_amount',
  value: 500,          // $500 de descuento fijo
  minOrderValue: 2000  // Mínimo $2000 de compra
}
```

### Free Shipping (Envío Gratis)

```typescript
{
  type: 'free_shipping',
  value: 0,            // Sin costo de envío
  minAmount: 1500      // Mínimo $1500 de compra
}
```

### Buy X Get Y (Compra X Lleva Y)

```typescript
{
  type: 'buy_x_get_y',
  value: 1,            // Lleva 1 gratis por cada X comprados
  // Configuración específica en otros campos
}
```

## 📊 Validaciones

### Validaciones de Schema

```typescript
const PromotionSchema = new Schema<IPromotion>({
  name: {
    type: String,
    required: true,
    unique: true,
    trim: true,
  },
  title: {
    type: String,
    required: true,
    trim: true,
  },
  type: {
    type: String,
    enum: ['percentage', 'fixed_amount', 'free_shipping', 'buy_x_get_y'],
    required: true,
  },
  value: {
    type: Number,
    required: true,
    min: 0,
  },
  // ... resto de validaciones
});
```

### Validaciones Personalizadas

```typescript
// Validar que endDate sea posterior a startDate
PromotionSchema.pre('validate', function (next) {
  if (this.startDate >= this.endDate) {
    next(new Error('La fecha de fin debe ser posterior a la fecha de inicio'));
  } else {
    next();
  }
});
```

## 🔧 Índices de Performance

### Índices Configurados

```typescript
// Índice único para códigos de cupón
PromotionSchema.index({ code: 1 });

// Índices para búsquedas frecuentes
PromotionSchema.index({ isActive: 1, startDate: 1, endDate: 1 });
PromotionSchema.index({ applicableCategories: 1 });
PromotionSchema.index({ applicableProducts: 1 });
```

## 🚀 Métodos de Instancia

### Verificación de Validez

```typescript
// Verificar si la promoción está activa y vigente
promotion.isValid(): boolean

// Verificar si aplica a un pedido específico
promotion.appliesToOrder(order: any): boolean

// Calcular descuento aplicable
promotion.calculateDiscount(orderAmount: number): number
```

### Gestión de Uso

```typescript
// Incrementar contador de uso
promotion.incrementUsage(): Promise<void>

// Verificar si alcanzó el límite de uso
promotion.hasReachedLimit(): boolean
```

## 📈 Estadísticas

### Información de Uso

```typescript
// Calcular tasa de uso
promotion.getUsageRate(): number

// Obtener días restantes de validez
promotion.getDaysRemaining(): number

// Obtener información de efectividad
promotion.getEffectiveness(): {
  usageRate: number,
  averageDiscount: number,
  totalDiscounted: number
}
```

## 🔒 Seguridad y Control de Acceso

### Validaciones de Creación

```typescript
// Solo usuarios autorizados pueden crear promociones
promotion.canBeCreatedBy(userId: string): boolean

// Validar permisos de modificación
promotion.canBeModifiedBy(userId: string): boolean
```

## 💾 Persistencia

### Configuración de Colección

```typescript
{
  timestamps: true,
  collection: 'promotions'
}
```

### Middleware

```typescript
// Actualizar estadísticas al completar promoción
PromotionSchema.post('save', async function (doc) {
  // Actualizar métricas relacionadas
});
```

## 🔧 Próximas Mejoras

- [ ] **Sistema de códigos QR:** Promociones escaneables
- [ ] **A/B Testing:** Comparación de efectividad entre promociones
- [ ] **Segmentación avanzada:** Aplicación basada en comportamiento de usuario
- [ ] **Integración con email marketing:** Promociones automáticas por email

---

_Última modificación: Octubre 2025_
_Estado: ✅ Completamente funcional y documentado_
