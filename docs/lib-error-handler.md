# 🚨 Librería: Error Handler

## 📋 Información General

**Ubicación:** `lib/error-handler.ts`
**Tipo:** Sistema centralizado de manejo de errores
**Propósito:** Gestión unificada de errores en toda la aplicación

## 🔧 Funcionalidades

### Manejo de Errores

- **Clasificación automática:** Categorización de tipos de error
- **Logging estructurado:** Registro detallado de errores
- **Respuestas consistentes:** Formato uniforme para respuestas de API
- **Recuperación inteligente:** Estrategias de recuperación según tipo de error

### Tipos de Error Personalizados

- **ApiError:** Errores generales de API
- **ValidationError:** Errores de validación de datos
- **AuthenticationError:** Problemas de autenticación
- **AuthorizationError:** Errores de permisos
- **NotFoundError:** Recursos no encontrados
- **ConflictError:** Conflictos de estado
- **RateLimitError:** Límites de uso excedidos

## 🚀 Cambios Implementados (Auditoría Octubre 2025)

### ✅ Correcciones de ESLint

**Problema Original:**

```typescript
// ❌ Warning: Anonymous default export
export default {
  ApiError,
  ValidationError,
  AuthenticationError,
  // ... muchos exports
};
```

**Solución Implementada:**

```typescript
// ✅ Corrección: Variable nombrada antes del export
const ErrorHandlerModule = {
  ApiError,
  ValidationError,
  AuthenticationError,
  AuthorizationError,
  NotFoundError,
  ConflictError,
  RateLimitError,
  ExternalServiceError,
  DatabaseError,
  PaymentError,
  BusinessLogicError,
  errorHandler,
  withApiErrorHandler,
  sanitizeError,
  logError,
  createError,
  isRecoverableError,
};

export default ErrorHandlerModule;
```

### 🔧 Mejoras de Código

1. **Estándares de Calidad:** Cumplimiento con reglas de ESLint
2. **Organización:** Módulo claramente estructurado
3. **Mantenibilidad:** Código más legible y profesional

## 📊 Clases de Error

### ApiError Base

```typescript
export class ApiError extends Error {
  public statusCode: number;
  public isOperational: boolean;
  public details?: any;

  constructor(
    message: string,
    statusCode: number = 500,
    isOperational: boolean = true,
    details?: any
  ) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    this.details = details;
  }
}
```

### ValidationError

```typescript
export class ValidationError extends ApiError {
  constructor(message: string, details?: any) {
    super(message, 400, true, details);
  }
}
```

### AuthenticationError

```typescript
export class AuthenticationError extends ApiError {
  constructor(message: string = 'Authentication required') {
    super(message, 401, true);
  }
}
```

## 🔧 Funciones de Utilidad

### Manejador de Errores Principal

```typescript
export function errorHandler(
  error: Error,
  request?: NextRequest,
  context?: any
): NextResponse {
  // 1. Clasificación del error
  // 2. Logging apropiado
  // 3. Sanitización de información sensible
  // 4. Respuesta HTTP consistente
}
```

### Wrapper para API Routes

```typescript
export function withApiErrorHandler(handler: any) {
  return async (request: NextRequest, context?: any) => {
    try {
      return await handler(request, context);
    } catch (error) {
      return errorHandler(error, request, context);
    }
  };
}
```

## 📊 Clasificación de Errores

### Errores Operacionales vs Programación

```typescript
export function isRecoverableError(error: Error): boolean {
  if (error instanceof ApiError) {
    return error.isOperational;
  }

  // Errores de red y base de datos generalmente son recuperables
  return (
    error.name === 'MongoNetworkError' ||
    error.name === 'MongoTimeoutError' ||
    error.code === 'ECONNRESET' ||
    error.code === 'ETIMEDOUT' ||
    error.code === 'ENOTFOUND'
  );
}
```

### Logging por Severidad

```typescript
enum LogLevel {
  ERROR = 'error',
  WARN = 'warn',
  INFO = 'info',
  DEBUG = 'debug',
}

function logError(error: Error, level: LogLevel = LogLevel.ERROR): void {
  const logData = {
    message: error.message,
    stack: error.stack,
    timestamp: new Date().toISOString(),
    level,
    // ... metadatos adicionales
  };

  // Logging según configuración (console, archivo, servicio externo)
}
```

## 🚨 Sanitización de Errores

### Información Sensible

```typescript
export function sanitizeError(error: Error): any {
  const sanitized = {
    message: error.message,
    statusCode: (error as any).statusCode || 500,
    timestamp: new Date().toISOString(),
  };

  // No incluir stack trace en producción
  if (process.env.NODE_ENV === 'production') {
    return sanitized;
  }

  return {
    ...sanitized,
    stack: error.stack,
    details: (error as any).details,
  };
}
```

## 💡 Ejemplo de Uso

### En API Routes

```typescript
import {
  withApiErrorHandler,
  NotFoundError,
  ValidationError,
} from '@/lib/error-handler';

export const GET = withApiErrorHandler(async (request: NextRequest) => {
  const user = await User.findById(params.id);

  if (!user) {
    throw new NotFoundError('Usuario no encontrado');
  }

  if (!user.isActive) {
    throw new ValidationError('Usuario inactivo');
  }

  return NextResponse.json(user);
});
```

### Manejo Manual

```typescript
try {
  await riskyOperation();
} catch (error) {
  if (error instanceof ValidationError) {
    return NextResponse.json(
      { error: error.message },
      { status: error.statusCode }
    );
  }

  throw error; // Re-lanzar para manejo centralizado
}
```

## 🔧 Configuración

### Variables de Entorno

```bash
ERROR_LOG_LEVEL=error     # Nivel mínimo de logging
ERROR_INCLUDE_STACK=true  # Incluir stack trace en desarrollo
ERROR_SERVICE_URL=        # URL de servicio de logging externo
```

### Configuración de Logging

```typescript
const errorConfig = {
  logLevel: process.env.ERROR_LOG_LEVEL || 'error',
  includeStack: process.env.NODE_ENV !== 'production',
  serviceUrl: process.env.ERROR_SERVICE_URL,
};
```

## 📊 Monitoreo y Métricas

### Métricas de Error

```typescript
interface ErrorMetrics {
  totalErrors: number;
  errorsByType: Record<string, number>;
  errorsByStatusCode: Record<number, number>;
  averageResponseTime: number;
  errorRate: number;
}
```

### Health Check

```typescript
export function getErrorHandlerHealth(): {
  status: 'healthy' | 'degraded' | 'unhealthy';
  metrics: ErrorMetrics;
  lastError?: Date;
} {
  // Implementación de health check
}
```

## 🔄 Estrategias de Recuperación

### Circuit Breaker Pattern

```typescript
class CircuitBreaker {
  private failures: number = 0;
  private lastFailureTime: number = 0;
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > 60000) {
        // 1 minuto
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
}
```

## 🚨 Alertas y Notificaciones

### Configuración de Alertas

```typescript
interface AlertConfig {
  errorThreshold: number; // Número de errores antes de alertar
  timeWindow: number; // Ventana de tiempo en minutos
  alertChannels: string[]; // Canales de notificación
}
```

### Notificaciones

```typescript
// Email a desarrolladores
// Slack notifications
// PagerDuty para errores críticos
// Métricas a sistemas de monitoreo
```

## 🔧 Próximas Mejoras

- [ ] **Distributed Tracing:** Seguimiento de errores en sistemas distribuidos
- [ ] **Machine Learning:** Detección automática de patrones de error
- [ ] **Auto-healing:** Recuperación automática de errores conocidos
- [ ] **Análisis predictivo:** Predicción de fallos antes de que ocurran

---

_Última modificación: Octubre 2025_
_Estado: ✅ Completamente funcional y corregido_
