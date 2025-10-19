# 🏆 Modelo: Achievement

## 📋 Información General

**Ubicación:** `models/Achievement.ts`
**Tipo:** Modelo Mongoose para sistema de logros
**Propósito:** Gestión completa del sistema de achievements y progresión de usuario

## 🔧 Estructura del Modelo

### Interface Principal

```typescript
export interface IAchievement {
  name: string;
  description: string;
  icon: string;
  category: 'orders' | 'spending' | 'loyalty' | 'social' | 'special';
  type: 'count' | 'amount' | 'streak' | 'special';
  requirement: {
    target: number;
    field?: string; // Campo a evaluar (ej: 'totalOrders', 'totalSpent')
    condition?: 'gte' | 'eq' | 'custom';
  };
  reward: {
    points: number;
    badge?: string; // URL del badge/medalla
    title?: string; // Título especial
  };
  rarity: 'common' | 'uncommon' | 'rare' | 'epic' | 'legendary';
  isActive: boolean;
  maxProgress?: number; // Para achievements que pueden ser completados múltiples veces
  statistics: {
    totalEarned: number;
    completionRate: number;
  };
  createdAt?: Date;
  updatedAt?: Date;
}
```

### Interface de Progreso de Usuario

```typescript
export interface IUserAchievement {
  userId: Schema.Types.ObjectId;
  achievementId: Schema.Types.ObjectId;
  progress: number;
  isCompleted: boolean;
  completedAt?: Date;
  claimedAt?: Date;
  metadata?: any;
  createdAt?: Date;
  updatedAt?: Date;
}
```

## 📊 Campos del Modelo

### Información Básica

| Campo         | Tipo   | Requerido | Descripción                  |
| ------------- | ------ | --------- | ---------------------------- |
| `name`        | String | ✅        | Nombre único del logro       |
| `description` | String | ✅        | Descripción del logro        |
| `icon`        | String | ✅        | URL del ícono representativo |

### Categorización

| Campo      | Tipo | Requerido | Descripción         |
| ---------- | ---- | --------- | ------------------- |
| `category` | Enum | ✅        | Categoría del logro |
| `type`     | Enum | ✅        | Tipo de requisito   |
| `rarity`   | Enum | ✅        | Rareza del logro    |

### Requisitos y Recompensas

| Campo         | Tipo   | Requerido | Descripción                |
| ------------- | ------ | --------- | -------------------------- |
| `requirement` | Object | ✅        | Condiciones para completar |
| `reward`      | Object | ✅        | Recompensas al completar   |

## 🚀 Tipos y Categorías

### Categorías de Logros

```typescript
type AchievementCategory =
  | 'orders' // Relacionados con pedidos
  | 'spending' // Relacionados con gasto
  | 'loyalty' // Relacionados con fidelidad
  | 'social' // Relacionados con interacciones sociales
  | 'special'; // Logros especiales
```

### Tipos de Requisito

```typescript
type AchievementType =
  | 'count' // Contar ocurrencias (ej: 10 pedidos)
  | 'amount' // Alcanzar monto (ej: gastar $1000)
  | 'streak' // Racha continua (ej: 7 días seguidos)
  | 'special'; // Lógica personalizada
```

### Niveles de Rareza

```typescript
type AchievementRarity =
  | 'common' // Comunes y fáciles de obtener
  | 'uncommon' // Poco comunes
  | 'rare' // Difíciles de conseguir
  | 'epic' // Muy difíciles
  | 'legendary'; // Extremadamente raros
```

## 📊 Sistema de Requisitos

### Ejemplos de Requisitos

```typescript
// Logro de pedidos
{
  type: 'count',
  requirement: {
    target: 10,
    field: 'totalOrders',
    condition: 'gte'
  }
}

// Logro de gasto
{
  type: 'amount',
  requirement: {
    target: 1000,
    field: 'totalSpent',
    condition: 'gte'
  }
}

// Logro de racha
{
  type: 'streak',
  requirement: {
    target: 7,
    field: 'dailyLogin',
    condition: 'gte'
  }
}
```

## 🏆 Sistema de Recompensas

### Tipos de Recompensa

```typescript
interface AchievementReward {
  points: number; // Puntos otorgados
  badge?: string; // URL de badge especial
  title?: string; // Título honorífico
}
```

### Ejemplos de Recompensas

```typescript
// Recompensa básica
{
  points: 50
}

// Recompensa con badge
{
  points: 200,
  badge: '/badges/first-order.png'
}

// Recompensa legendaria
{
  points: 1000,
  badge: '/badges/vip-client.png',
  title: 'Cliente VIP'
}
```

## 🔧 Métodos de Instancia

### Verificación de Progreso

```typescript
AchievementSchema.methods.checkUserProgress = function (userId: string) {
  return models.UserAchievement.findOne({
    userId,
    achievementId: this._id,
  }).populate('achievementId');
};
```

### Actualización de Progreso

```typescript
AchievementSchema.methods.updateUserProgress = function (
  userId: string,
  newProgress: number
) {
  return models.UserAchievement.findOneAndUpdate(
    { userId, achievementId: this._id },
    {
      $set: { progress: Math.min(newProgress, this.requirement.target) },
      $setOnInsert: {
        userId,
        achievementId: this._id,
        isCompleted: false,
      },
    },
    {
      upsert: true,
      new: true,
      runValidators: true,
    }
  );
};
```

## 📈 Estadísticas y Métricas

### Información de Estadísticas

```typescript
statistics: {
  totalEarned: { type: Number, default: 0, min: 0 },
  completionRate: { type: Number, default: 0, min: 0, max: 100 }
}
```

### Métodos de Estadísticas

```typescript
// Obtener achievements disponibles para un usuario
AchievementSchema.statics.getAvailableForUser = function(userId: string)

// Obtener progreso del usuario
AchievementSchema.statics.getUserProgress = function(userId: string)

// Obtener achievements completados
AchievementSchema.statics.getUserCompleted = function(userId: string)
```

## 💾 Achievements Prediseñados

### Sistema de Defaults

```typescript
AchievementSchema.statics.getDefaultAchievements = function () {
  return [
    // Achievements de órdenes
    {
      name: 'Primer Pedido',
      description: 'Completa tu primer pedido',
      icon: '/icons/first-order.svg',
      category: 'orders',
      type: 'count',
      requirement: { target: 1, field: 'totalOrders' },
      reward: { points: 50, badge: '/badges/first-order.png' },
      rarity: 'common',
    },
    // Más achievements...
  ];
};
```

## 🔧 Configuración e Índices

### Índices de Performance

```typescript
// Índices para búsquedas eficientes
AchievementSchema.index({ category: 1, isActive: 1 });
AchievementSchema.index({ rarity: 1, isActive: 1 });
AchievementSchema.index({ type: 1, isActive: 1 });
```

### Configuración de Modelo

```typescript
const Achievement = (models.Achievement ||
  model<IAchievement>(
    'Achievement',
    AchievementSchema
  )) as Model<IAchievement> & AchievementModel;
```

## 🚀 Inicialización

### Sistema de Setup Automático

```typescript
AchievementSchema.statics.initializeDefaultAchievements = function () {
  return this.countDocuments().then((count: number) => {
    if (count === 0) {
      const defaultAchievements = (this as any).getDefaultAchievements();
      return this.insertMany(defaultAchievements);
    }
    return Promise.resolve();
  });
};
```

## 🔒 Seguridad y Validaciones

### Validaciones de Schema

```typescript
const AchievementSchema = new Schema<IAchievement>({
  name: {
    type: String,
    required: true,
    unique: true,
    trim: true,
  },
  rarity: {
    type: String,
    enum: ['common', 'uncommon', 'rare', 'epic', 'legendary'],
    default: 'common',
    index: true,
  },
  isActive: {
    type: Boolean,
    default: true,
    index: true,
  },
  // ... resto de validaciones
});
```

## 💡 Ejemplo de Uso Completo

```typescript
// Crear nuevo achievement
const achievement = new Achievement({
  name: 'Comprador VIP',
  description: 'Gasta más de $10,000 en total',
  icon: '/icons/vip-shopper.svg',
  category: 'spending',
  type: 'amount',
  requirement: {
    target: 10000,
    field: 'totalSpent',
  },
  reward: {
    points: 500,
    badge: '/badges/vip-shopper.png',
    title: 'Comprador VIP',
  },
  rarity: 'rare',
});

await achievement.save();
```

## 🔧 Próximas Mejoras

- [ ] **Logros colaborativos:** Achievements que requieren múltiples usuarios
- [ ] **Sistema de temporadas:** Logros temporales por eventos especiales
- [ ] **Progresión visual:** Gráficos de avance hacia logros
- [ ] **Gamificación avanzada:** Sistema de niveles y rankings

---

_Última modificación: Octubre 2025_
_Estado: ✅ Completamente funcional y documentado_
