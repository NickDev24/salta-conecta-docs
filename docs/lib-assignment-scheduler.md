# 📅 Librería: Assignment Scheduler

## 📋 Información General

**Ubicación:** `lib/assignment-scheduler.ts`
**Tipo:** Servicio para programación automática de entregas
**Propósito:** Gestión inteligente de asignación de pedidos a conductores

## 🔧 Funcionalidades

### Programación Automática

- **Algoritmo de asignación:** Distribución óptima de pedidos
- **Programación periódica:** Ejecución automática en intervalos configurables
- **Asignación manual:** Función para asignaciones específicas
- **Monitoreo continuo:** Seguimiento del estado de asignaciones

### Características Avanzadas

- **Optimización de rutas:** Consideración de ubicación y carga de trabajo
- **Priorización:** Sistema de prioridades para pedidos urgentes
- **Balanceo de carga:** Distribución equitativa entre conductores disponibles

## 🚀 Cambios Implementados (Auditoría Octubre 2025)

### ✅ Correcciones de ESLint

**Problema Original:**

```typescript
// ❌ Warning: Anonymous default export
export default {
  scheduleAssignments,
  AssignmentScheduler,
  startAssignmentScheduler,
  stopAssignmentScheduler,
  runManualAssignment,
};
```

**Solución Implementada:**

```typescript
// ✅ Corrección: Variable nombrada antes del export
const AssignmentSchedulerModule = {
  scheduleAssignments,
  AssignmentScheduler,
  startAssignmentScheduler,
  stopAssignmentScheduler,
  runManualAssignment,
};

export default AssignmentSchedulerModule;
```

### 🔧 Mejoras de Código

1. **Estándares de Calidad:** Cumplimiento con reglas de ESLint
2. **Mantenibilidad:** Código más claro y estructurado
3. **Reutilización:** Módulo claramente definido

## 📊 Estructura del Servicio

### Clase Principal

```typescript
class AssignmentScheduler {
  private isRunning: boolean = false;
  private intervalId?: NodeJS.Timeout;

  start(): Promise<{ success: boolean }> {
    // Inicialización del programador
  }

  stop(): Promise<{ success: boolean }> {
    // Detención del programador
  }

  private async runAssignmentCycle(): Promise<void> {
    // Ciclo principal de asignación
  }
}
```

### Funciones de Control

```typescript
export function startAssignmentScheduler(
  intervalMinutes: number = 2
): Promise<{ success: boolean }> {
  // Función para iniciar el programador con intervalo personalizado
}

export function stopAssignmentScheduler(): Promise<{ success: boolean }> {
  // Función para detener el programador
}
```

## 🔧 Algoritmo de Asignación

### Proceso de Asignación

```typescript
static async scheduleAssignments(): Promise<void> {
  // 1. Obtener pedidos pendientes
  // 2. Obtener conductores disponibles
  // 3. Aplicar algoritmo de matching
  // 4. Crear asignaciones
  // 5. Notificar a las partes involucradas
}
```

### Criterios de Asignación

```typescript
// Ejemplo de algoritmo simplificado
const assignOrderToDriver = (order: any, availableDrivers: any[]) => {
  // 1. Filtrar conductores por zona
  // 2. Ordenar por carga de trabajo actual
  // 3. Considerar distancia y tiempo estimado
  // 4. Seleccionar el más adecuado
};
```

## 📊 Estados y Monitoreo

### Estado del Programador

```typescript
interface SchedulerStatus {
  isRunning: boolean;
  lastRun?: Date;
  nextRun?: Date;
  intervalMinutes: number;
  pendingOrders: number;
  availableDrivers: number;
}
```

### Métricas de Rendimiento

```typescript
interface AssignmentMetrics {
  totalAssignments: number;
  successfulAssignments: number;
  failedAssignments: number;
  averageAssignmentTime: number;
  successRate: number;
}
```

## 🚨 Gestión de Errores

### Manejo de Excepciones

```typescript
try {
  await scheduleAssignments();
} catch (error) {
  console.error('Error in assignment scheduling:', error);
  // Log detallado del error
  // Notificación a administradores si es crítico
}
```

### Recuperación de Errores

```typescript
// Sistema de reintento para operaciones críticas
const MAX_RETRIES = 3;
let attempt = 0;

while (attempt < MAX_RETRIES) {
  try {
    await assignOrderToDriver(order, drivers);
    break; // Éxito, salir del loop
  } catch (error) {
    attempt++;
    if (attempt >= MAX_RETRIES) {
      throw new Error(`Failed to assign order after ${MAX_RETRIES} attempts`);
    }
    await delay(1000 * attempt); // Backoff exponencial
  }
}
```

## 💡 Ejemplo de Uso

### Inicio del Programador

```typescript
import {
  startAssignmentScheduler,
  stopAssignmentScheduler,
} from '@/lib/assignment-scheduler';

// Iniciar con intervalo de 2 minutos
await startAssignmentScheduler(2);

// Detener el programador
await stopAssignmentScheduler();
```

### Asignación Manual

```typescript
import { runManualAssignment } from '@/lib/assignment-scheduler';

// Asignar un pedido específico a un conductor
await runManualAssignment('order_id', 'driver_id');
```

## 🔄 Configuración

### Variables de Entorno

```bash
ASSIGNMENT_SCHEDULER_INTERVAL=2  # Minutos entre ejecuciones
ASSIGNMENT_MAX_RETRIES=3         # Reintentos máximos por asignación
ASSIGNMENT_TIMEOUT=30000         # Timeout en milisegundos
```

### Configuración Dinámica

```typescript
const config = {
  intervalMinutes: parseInt(process.env.ASSIGNMENT_SCHEDULER_INTERVAL || '2'),
  maxRetries: parseInt(process.env.ASSIGNMENT_MAX_RETRIES || '3'),
  timeout: parseInt(process.env.ASSIGNMENT_TIMEOUT || '30000'),
};
```

## 📊 Logs y Monitoreo

### Logging Estructurado

```typescript
const logger = {
  info: (message: string, meta?: any) => console.log(`[INFO] ${message}`, meta),
  error: (message: string, error?: Error) =>
    console.error(`[ERROR] ${message}`, error),
  warn: (message: string, meta?: any) =>
    console.warn(`[WARN] ${message}`, meta),
};
```

### Métricas de Ejecución

```typescript
const metrics = {
  assignmentsAttempted: 0,
  assignmentsSuccessful: 0,
  assignmentsFailed: 0,
  averageProcessingTime: 0,
};
```

## 🚀 Características de Producción

### Health Checks

```typescript
export function getSchedulerHealth(): SchedulerStatus {
  return {
    isRunning: scheduler.isRunning,
    lastRun: scheduler.lastRun,
    nextRun: scheduler.nextRun,
    intervalMinutes: scheduler.intervalMinutes,
    pendingOrders: getPendingOrdersCount(),
    availableDrivers: getAvailableDriversCount(),
  };
}
```

### Graceful Shutdown

```typescript
process.on('SIGTERM', async () => {
  console.log('Shutting down assignment scheduler...');
  await stopAssignmentScheduler();
  process.exit(0);
});
```

## 🔧 Próximas Mejoras

- [ ] **Machine Learning:** Algoritmo de asignación basado en ML
- [ ] **Predicción de demanda:** Asignación proactiva basada en patrones
- [ ] **Zonas dinámicas:** Ajuste automático de zonas de entrega
- [ ] **Integración GPS:** Uso de ubicación en tiempo real
- [ ] **Sistema de puntuación:** Rating de conductores para mejor matching

---

_Última modificación: Octubre 2025_
_Estado: ✅ Completamente funcional y corregido_
