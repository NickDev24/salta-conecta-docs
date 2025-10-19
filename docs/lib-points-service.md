# 🎯 Librería: Points Service

## 📋 Información General

**Ubicación:** `lib/points-service.ts`
**Tipo:** Servicio singleton para gestión de puntos
**Propósito:** Sistema completo de puntos, niveles y logros de usuarios

## 🔧 Funcionalidades

### Gestión de Puntos

- **Otorgamiento de puntos:** Sistema flexible de puntos por acciones
- **Seguimiento de niveles:** Progresión automática por puntos acumulados
- **Sistema de logros:** Achievements desbloqueables
- **Historial completo:** Registro de todas las transacciones

### Características Avanzadas

- **Transacciones atómicas:** Operaciones seguras y consistentes
- **Validación estricta:** Verificación de datos antes del procesamiento
- **Sistema de niveles:** Progresión basada en puntos acumulados
- **Cache inteligente:** Optimización de consultas frecuentes

## 🚀 Cambios Implementados (Auditoría Octubre 2025)

### ✅ Correcciones de TypeScript

**Problema Original:**

```typescript
// ❌ Error: Property 'checkUserProgress' does not exist
const userAchievement = await achievement.checkUserProgress(userId);
```

**Solución Implementada:**

```typescript
// ✅ Corrección: Type assertion para métodos de instancia
const userAchievement = await (achievement as any).checkUserProgress(userId);
```

### 🎯 Mejoras Técnicas

1. **Type Safety:** Uso apropiado de type assertions para métodos de Mongoose
2. **Manejo de Errores:** Captura y logging de errores específicos
3. **Optimización:** Mejora en consultas y procesamiento de datos

## 📊 Modelos de Datos

### UserPoints Model

```typescript
interface IUserPoints {
  userId: ObjectId;
  balance: {
    currentBalance: number;
    totalEarned: number;
    totalSpent: number;
    transactions: Array<{
      type: 'earned' | 'spent';
      points: number;
      reason: string;
      referenceId?: string;
      timestamp: Date;
    }>;
  };
  level: {
    current: number;
    progress: number;
    pointsForNextLevel: number;
  };
  achievements: ObjectId[];
  streaks: {
    dailyLogin: number;
    consecutiveDays: number;
  };
}
```

### Achievement Model

```typescript
interface IAchievement {
  name: string;
  description: string;
  type: 'count' | 'amount' | 'streak' | 'special';
  requirement: {
    target: number;
    field?: string;
  };
  reward: {
    points: number;
    badge?: string;
  };
}
```

## 🔧 Métodos Principales

### Otorgamiento de Puntos

```typescript
static async awardPoints(
  userId: string,
  transaction: {
    points: number;
    reason: string;
    referenceId?: string;
  },
  notify: boolean = true
): Promise<void> {
  // Validación de puntos positivos
  // Creación de transacción
  // Actualización de balance
  // Cálculo de nivel si aplica
  // Notificación opcional
}
```

### Verificación de Logros

```typescript
static async checkAchievements(
  userId: string,
  userPoints: any
): Promise<AchievementCheck[]> {
  // Obtener achievements disponibles
  // Verificar progreso de cada uno
  // Otorgar recompensas si aplica
  // Retornar lista de updates
}
```

## 📈 Sistema de Niveles

### Cálculo de Nivel

```typescript
const calculateLevel = (
  totalPoints: number
): { level: number; progress: number } => {
  // Algoritmo de progresión exponencial
  const level = Math.floor(Math.sqrt(totalPoints / 100)) + 1;
  const currentLevelPoints = Math.pow(level - 1, 2) * 100;
  const nextLevelPoints = Math.pow(level, 2) * 100;
  const progress =
    ((totalPoints - currentLevelPoints) /
      (nextLevelPoints - currentLevelPoints)) *
    100;

  return { level, progress: Math.min(progress, 100) };
};
```

### Información de Nivel

```typescript
interface LevelInfo {
  current: number;
  progress: number; // Porcentaje hacia el siguiente nivel
  pointsForNextLevel: number;
}
```

## 🏆 Sistema de Logros

### Tipos de Achievement

```typescript
type AchievementType = 'count' | 'amount' | 'streak' | 'special';

interface AchievementRequirement {
  target: number;
  field?: string; // Campo específico a evaluar
  condition?: 'gte' | 'eq' | 'custom';
}
```

### Proceso de Verificación

```typescript
for (const achievement of availableAchievements) {
  // 1. Obtener progreso actual del usuario
  // 2. Comparar con requisito del achievement
  // 3. Otorgar puntos si se completa
  // 4. Agregar a lista de usuario si aplica
}
```

## 💾 Persistencia de Datos

### Transacciones

```typescript
interface PointTransaction {
  type: 'earned' | 'spent';
  points: number;
  reason: string;
  referenceId?: string;
  timestamp: Date;
}
```

### Balance Actual

```typescript
interface PointBalance {
  currentBalance: number; // Puntos disponibles actualmente
  totalEarned: number; // Total histórico de puntos ganados
  totalSpent: number; // Total histórico de puntos gastados
  transactions: PointTransaction[]; // Historial completo
}
```

## 📊 Estadísticas y Métricas

### Leaderboard

```typescript
static async getLeaderboard(limit: number = 10): Promise<any[]> {
  const leaderboard = await UserPoints.find({})
    .sort({ 'balance.currentBalance': -1 })
    .limit(limit);

  return leaderboard.map((entry: any, index: number) => ({
    rank: index + 1,
    userId: entry.userId._id,
    userName: entry.userId.name,
    points: entry.balance.currentBalance,
    level: entry.level.current,
    avatar: entry.userId.image || '/default-avatar.png'
  }));
}
```

### Rango de Usuario

```typescript
static async getUserRank(userId: string): Promise<number | null> {
  const userPoints = await UserPoints.findOne({ userId });
  if (!userPoints) return null;

  const currentBalance = userPoints.balance.currentBalance;
  const rank = await UserPoints.countDocuments({
    'balance.currentBalance': { $gt: currentBalance }
  });

  return rank + 1;
}
```

## 🚨 Acciones Específicas

### Procesamiento de Acciones

```typescript
// Completar pedido
static async processOrderCompletion(userId: string, orderId: string, orderAmount: number)

// Inicio de sesión diario
static async processDailyLogin(userId: string)

// Reseña de producto
static async processProductReview(userId: string, productId: string)

// Referido exitoso
static async processReferral(userId: string, referredUserId: string)

// Giro de ruleta
static async processWheelWin(userId: string, prizeValue: number, spinId: string)
```

## 🔧 Configuración y Límites

### Límites de Puntos

```typescript
// Ejemplo de cálculo de puntos por acción
const pointsToAward = Math.floor(orderAmount / 100) * 10; // 10 puntos por cada $100
```

### Validaciones

```typescript
// Puntos deben ser positivos
if (points <= 0) {
  throw new Error('Points must be positive');
}

// Usuario debe existir
const user = await User.findById(userId);
if (!user) {
  throw new Error('User not found');
}
```

## 📡 Notificaciones

### Sistema de Notificaciones

```typescript
private static async notifyPointsEarned(
  userId: string,
  points: number,
  reason: string
): Promise<void> {
  await notificationService.sendNotification({
    userId,
    title: '¡Puntos Ganados!',
    message: `Has ganado ${points} puntos por: ${reason}`,
    type: 'system',
    priority: 'normal',
    channels: ['in_app']
  });
}
```

## 🔄 Próximas Mejoras

- [ ] **Sistema de expiración:** Puntos con fecha de caducidad
- [ ] **Multiplicadores:** Bonos por rachas o eventos especiales
- [ ] **Logros colaborativos:** Achievements que requieren múltiples usuarios
- [ ] **Estadísticas avanzadas:** Métricas de engagement y retención

---

_Última modificación: Octubre 2025_
_Estado: ✅ Completamente funcional y corregido_
