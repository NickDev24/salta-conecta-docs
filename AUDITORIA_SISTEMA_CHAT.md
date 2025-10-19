# 🔍 AUDITORÍA COMPLETA DEL SISTEMA SALTA CONECTA

**Fecha:** 19 de Octubre 2025  
**Auditor:** Sistema de Análisis  
**Enfoque:** Sistema de Chat, Panel Administrativo, Flujo Conversacional, Almacenamiento de Datos

---

## 📋 RESUMEN EJECUTIVO

El proyecto SaltaConecta es una plataforma mayorista Next.js 15 con sistema de chat multi-usuario. Se identificaron **problemas críticos** en el sistema de almacenamiento de chats que causan la creación de múltiples conversaciones en lugar de reutilizar las existentes.

### Hallazgos Principales
- ❌ **CRÍTICO:** Inconsistencia en modelos de datos del chat
- ❌ **CRÍTICO:** Campo `chatId` no definido en el Schema pero usado en la API
- ⚠️ **ALTO:** Duplicación de lógica de mensajería entre modelos
- ⚠️ **MEDIO:** APIs de drivers usan datos mock en producción
- ⚠️ **MEDIO:** Falta de WebSocket real para chat en tiempo real
- ✅ **POSITIVO:** Buena estructura general del proyecto
- ✅ **POSITIVO:** Implementación de roles y permisos

---

## 🚨 PROBLEMA CRÍTICO: SISTEMA DE CHAT

### 1. Inconsistencia en el Modelo de Datos

#### Problema Identificado
El archivo `/models/Chat.ts` define un schema de Mongoose **SIN** el campo `chatId`:

```typescript
// Chat.ts - Schema definido
const ChatSchema = new Schema<IChat>({
  type: { type: String, enum: ['support', 'order', 'delivery', 'general'] },
  title: { type: String, required: true },
  participants: [ChatParticipantSchema],
  messages: [ChatMessageSchema],  // Mensajes EMBEBIDOS
  orderId: { type: Schema.Types.ObjectId, ref: 'Order' },
  // ... NO HAY CAMPO chatId
});
```

Pero la API `/api/chat/route.ts` intenta usar `chatId`:

```typescript
// route.ts línea 142-154
const chatIdKey = `${chatType}:${orderId}:${idsForKey}`;

// ❌ ESTO FALLA - chatId no existe en el schema
let chat = await Chat.findOne({ chatId: chatIdKey });
if (!chat) {
  chat = await Chat.create({
    chatId: chatIdKey,  // ❌ Campo no definido
    chatType,           // ❌ Campo no definido
    participants: participants as any,
    orderId,
    messages: [],
  });
}
```

**Resultado:** Cada consulta crea un nuevo chat porque:
1. `findOne({ chatId: chatIdKey })` siempre devuelve `null` (campo no existe)
2. `create()` inserta un documento nuevo SIN validar `chatId`
3. Se generan múltiples chats para la misma conversación

---

### 2. Duplicación de Modelos

Existen **DOS modelos** conflictivos:

#### A. `/models/Chat.ts`
- Mensajes **embebidos** dentro del documento
- Schema: `messages: [ChatMessageSchema]`
- Métodos: `addMessage()`, `markAsRead()`, etc.

#### B. `/models/Message.ts`
- Mensajes como **colección separada**
- Schema: `chatId: { type: Schema.Types.ObjectId, ref: 'Chat' }`
- Métodos propios independientes

**Problema:** Las APIs usan ambos modelos de forma inconsistente:
- `/api/chat/route.ts` → Usa `Chat` con mensajes embebidos
- `/api/admin/chat/route.ts` → Usa `Chat` y `Message` separados
- Esto causa confusión y duplicación de datos

---

### 3. Análisis de Rutas API

#### `/api/chat/route.ts` ❌ PROBLEMÁTICA
```typescript
// Línea 26: Busca por chatType que no existe en schema
if (type) query.chatType = type;

// Línea 151: Busca por chatId inexistente
let chat = await Chat.findOne({ chatId: chatIdKey });
```

**Campos usados pero NO definidos en schema:**
- `chatId` ❌
- `chatType` ❌  
- `isActive` ❌

#### `/api/chat/[chatId]/messages/route.ts` ❌ PROBLEMÁTICA
```typescript
// Línea 21-25: Intenta buscar por chatId inexistente
const chat = await Chat.findOne({
  chatId,  // ❌ No existe en schema
  isActive: true,  // ❌ No existe en schema
  'participants.userId': user.id,
});
```

#### `/api/admin/chat/route.ts` ⚠️ PARCIALMENTE CORRECTA
```typescript
// Usa estructura diferente - participants como array de IDs
const query: any = {
  participants: user.id,  // Asume array simple
};

// Crea chat con modelo Message separado
const message = new Message({
  chatId: chat._id,
  sender: user.id,
  content: initialMessage,
});
```

**Problema:** Incompatibilidad entre estructuras de datos

---

### 4. APIs de Drivers con Datos Mock

#### `/api/drivers/chat/conversations/route.ts`
```typescript
// Línea 28-119: TODO es MOCK DATA
const conversations = [
  {
    _id: '1',
    orderId: 'ORD-001',
    customer: { id: 'customer1', name: 'María González' },
    // ... datos hardcodeados
  }
];
```

**Problema CRÍTICO:** En producción, estas APIs devuelven datos falsos. Los drivers NO pueden ver conversaciones reales.

---

## 🔧 SOLUCIÓN PROPUESTA

### Paso 1: Unificar el Modelo de Datos

**Opción A - Mensajes Embebidos (Recomendado para < 1000 msgs/chat):**

```typescript
// models/Chat.ts - VERSIÓN CORREGIDA
interface IChat {
  chatId: string;  // ✅ NUEVO campo único
  chatType: 'customer_supplier' | 'customer_admin' | 'supplier_admin';  // ✅ NUEVO
  participants: IChatParticipant[];
  messages: IChatMessage[];  // Embebidos
  orderId?: Schema.Types.ObjectId;
  isActive: boolean;  // ✅ NUEVO
  lastMessage?: {
    content: string;
    timestamp: Date;
    senderName: string;
  };
  createdAt?: Date;
  updatedAt?: Date;
}

const ChatSchema = new Schema<IChat>({
  chatId: {
    type: String,
    required: true,
    unique: true,  // ✅ IMPORTANTE
    index: true,
  },
  chatType: {
    type: String,
    enum: ['customer_supplier', 'customer_admin', 'supplier_admin'],
    required: true,
  },
  // ... resto del schema
  isActive: {
    type: Boolean,
    default: true,
    index: true,
  }
});

// Índice compuesto para evitar duplicados
ChatSchema.index({ chatType: 1, orderId: 1, participants: 1 }, { unique: true });
```

**Opción B - Mensajes Separados (Recomendado para > 1000 msgs/chat):**

```typescript
// models/Chat.ts - SOLO conversación
interface IChat {
  chatId: string;
  chatType: string;
  participants: IChatParticipant[];
  orderId?: Schema.Types.ObjectId;
  lastMessageRef: Schema.Types.ObjectId;  // Referencia
  messageCount: number;
  isActive: boolean;
}

// models/Message.ts - Mensajes independientes
// (Ya existe, solo hay que usarlo consistentemente)
```

---

### Paso 2: Corregir API de Creación de Chat

```typescript
// /api/chat/route.ts - VERSIÓN CORREGIDA
export async function POST(request: NextRequest) {
  const { orderId, supplierId, customerId, message } = await request.json();
  
  // Validaciones
  if (!orderId || !message) {
    return NextResponse.json({ error: 'Datos requeridos faltantes' }, { status: 400 });
  }

  const chatType = supplierId && customerId ? 'customer_supplier' 
                 : customerId ? 'customer_admin' 
                 : 'supplier_admin';

  // Generar chatId único y determinístico
  const participantIds = [supplierId, customerId].filter(Boolean).sort();
  const chatId = `${chatType}:${orderId}:${participantIds.join(':')}`;

  // ✅ Buscar o crear con el campo correcto
  let chat = await Chat.findOne({ chatId });
  
  if (!chat) {
    chat = await Chat.create({
      chatId,
      chatType,
      participants: [
        // ... crear participantes
      ],
      orderId,
      messages: [],
      isActive: true,
    });
  }

  // Agregar mensaje al chat existente
  const newMessage = {
    senderId: user.id,
    senderName: user.name,
    senderRole: user.role,
    content: message,
    messageType: 'text',
    timestamp: new Date(),
    isRead: false,
  };

  chat.messages.push(newMessage);
  chat.lastMessage = {
    content: message,
    timestamp: new Date(),
    senderName: user.name,
  };
  
  await chat.save();

  return NextResponse.json({ message: 'Mensaje enviado', chat });
}
```

---

### Paso 3: Implementar APIs de Drivers Reales

```typescript
// /api/drivers/chat/conversations/route.ts - VERSIÓN REAL
export async function GET(request: NextRequest) {
  const session = await getServerSession(authOptions);
  
  if (!session || session.user.role !== 'driver') {
    return NextResponse.json({ error: 'No autorizado' }, { status: 401 });
  }

  await connectToDatabase();

  // ✅ Buscar conversaciones reales del driver
  const conversations = await Chat.find({
    isActive: true,
    'participants.userId': session.user.id,
    'participants.role': 'driver',
  })
    .populate('orderId', 'orderNumber status')
    .populate('participants.userId', 'name email')
    .sort({ updatedAt: -1 })
    .lean();

  return NextResponse.json({ conversations });
}
```

---

## 📊 PANEL ADMINISTRATIVO

### Estado Actual ✅
- **Ubicación:** `/app/admin/chat/page.tsx`
- **Funcionalidades:**
  - ✅ Lista de conversaciones
  - ✅ Vista de mensajes
  - ✅ Envío de mensajes
  - ✅ Filtros por orden
  - ✅ UI responsive con shadcn/ui

### Problemas Identificados
1. **Normalización inconsistente** (línea 136-146):
   - Maneja múltiples formatos de mensaje (`message` vs `content`)
   - Indica que las APIs devuelven estructuras diferentes

2. **Falta de actualización en tiempo real:**
   - No usa WebSocket
   - Requiere refresh manual

3. **Identificador confuso:**
   - Usa `_id` O `chatId` (línea 87-88)
   - Debería haber un identificador único consistente

### Recomendaciones
```typescript
// Agregar WebSocket para actualizaciones en tiempo real
import { useSocket } from '@/hooks/useSocket';

function AdminChatPageInner() {
  const { on, emit } = useSocket();

  useEffect(() => {
    if (!selectedChat) return;

    const handleNewMessage = (data: any) => {
      if (data.chatId === selectedChat.chatId) {
        setMessages(prev => [...prev, data.message]);
      }
    };

    on('chat:new-message', handleNewMessage);
    
    // Unirse a la sala del chat
    emit('chat:join', { chatId: selectedChat.chatId });

    return () => {
      off('chat:new-message', handleNewMessage);
      emit('chat:leave', { chatId: selectedChat.chatId });
    };
  }, [selectedChat, on, emit]);
}
```

---

## 🔄 FLUJO CONVERSACIONAL

### Flujo Actual (Problemático)

```
Usuario Solicita Chat
       ↓
API: Genera chatIdKey
       ↓
API: findOne({ chatId: chatIdKey })  ← ❌ SIEMPRE NULL (campo no existe)
       ↓
API: create() nuevo chat  ← ❌ SIEMPRE CREA NUEVO
       ↓
Resultado: Chat duplicado
```

### Flujo Correcto Propuesto

```
Usuario Solicita Chat (OrderID + Participants)
       ↓
Generar chatId determinístico: "tipo:orden:usuarios"
       ↓
findOne({ chatId }) con campo indexado
       ↓
¿Existe?
  ├─ SI → Agregar mensaje al chat existente
  └─ NO → Crear nuevo chat con chatId único
       ↓
Emitir evento WebSocket a participantes
       ↓
Guardar en BD con lastMessage actualizado
```

---

## 🗄️ ALMACENAMIENTO DE CHATS

### Problemas Actuales

1. **Sin índices adecuados:**
   ```typescript
   // ❌ FALTA en Chat.ts
   ChatSchema.index({ chatId: 1 }, { unique: true });
   ChatSchema.index({ orderId: 1, chatType: 1 });
   ChatSchema.index({ 'participants.userId': 1, isActive: 1 });
   ```

2. **Crecimiento ilimitado:**
   - Mensajes embebidos pueden crecer infinitamente
   - MongoDB tiene límite de 16MB por documento
   - Sin paginación de mensajes

3. **Sin TTL (Time To Live):**
   - Chats antiguos nunca se archivan
   - Base de datos crece indefinidamente

### Soluciones Propuestas

#### A. Agregar Índices
```typescript
// models/Chat.ts
ChatSchema.index({ chatId: 1 }, { unique: true });
ChatSchema.index({ orderId: 1, chatType: 1 });
ChatSchema.index({ 'participants.userId': 1, isActive: 1 });
ChatSchema.index({ updatedAt: -1 });
```

#### B. Limitar Mensajes Embebidos
```typescript
ChatSchema.pre('save', function(next) {
  // Mantener solo últimos 100 mensajes embebidos
  if (this.messages.length > 100) {
    // Opción 1: Mover a colección Message
    const oldMessages = this.messages.slice(0, -100);
    // Guardar en Message collection
    
    // Opción 2: Eliminar (no recomendado)
    this.messages = this.messages.slice(-100);
  }
  next();
});
```

#### C. Archivado Automático
```typescript
// Script de mantenimiento
async function archiveOldChats() {
  const sixMonthsAgo = new Date();
  sixMonthsAgo.setMonth(sixMonthsAgo.getMonth() - 6);

  await Chat.updateMany(
    {
      status: 'resolved',
      resolvedAt: { $lt: sixMonthsAgo },
      isActive: true
    },
    {
      $set: { isActive: false, status: 'archived' }
    }
  );
}
```

---

## 🎯 OTRAS CARACTERÍSTICAS AUDITADAS

### 1. Sistema de Autenticación ✅
- **Next-Auth** implementado correctamente
- Middleware de roles (`requireRole`) funciona bien
- Sesiones manejadas apropiadamente

### 2. Base de Datos ✅
- **MongoDB con Mongoose** configurado
- Conexión centralizada en `/lib/mongodb`
- Modelos bien estructurados (excepto Chat)

### 3. Socket.IO ⚠️
- Instalado (`socket.io` v4.8.1)
- Hook `useSocket` existe en `/hooks/useSocket`
- **Problema:** No hay servidor Socket.IO visible en el código
- **Falta:** Implementación del servidor en `server.js` o `pages/api/socket`

### 4. Panel de Drivers ⚠️
- UI bien implementada (`/app/driver/chat/page.tsx`)
- **CRÍTICO:** APIs usan datos mock (no funcional en producción)
- Necesita conexión a datos reales

### 5. Componente de Soporte ⚠️
- Componente flotante bien diseñado
- **Problema:** Solo respuestas simuladas
- No conectado a sistema real de chat

---

## 📝 PLAN DE ACCIÓN RECOMENDADO

### Prioridad CRÍTICA (Inmediato)
1. **Corregir modelo Chat.ts:**
   - Agregar campos `chatId`, `chatType`, `isActive`
   - Agregar índices únicos
   - Migrar datos existentes

2. **Actualizar API /api/chat/route.ts:**
   - Usar campo `chatId` correcto
   - Validar existencia antes de crear
   - Implementar lógica de reutilización

3. **Script de migración:**
   ```bash
   npm run db:migrate-chats
   ```

### Prioridad ALTA (Esta semana)
4. **Implementar APIs reales de drivers:**
   - Reemplazar mock data
   - Conectar a Chat model
   - Testear con datos reales

5. **Agregar WebSocket server:**
   - Configurar Socket.IO server
   - Implementar salas por chat
   - Events: `chat:join`, `chat:leave`, `chat:new-message`

6. **Paginación de mensajes:**
   - API para cargar mensajes antiguos
   - Infinite scroll en frontend
   - Limitar mensajes embebidos

### Prioridad MEDIA (Próximas 2 semanas)
7. **Panel administrativo mejorado:**
   - Actualización en tiempo real
   - Filtros avanzados
   - Búsqueda de mensajes

8. **Archivado automático:**
   - Cron job para archivar chats viejos
   - Dashboard de chats archivados

9. **Testing:**
   - Unit tests para modelos
   - Integration tests para APIs
   - E2E tests para flujo completo

### Prioridad BAJA (Backlog)
10. **Características adicionales:**
    - Notificaciones push
    - Adjuntar archivos en chats
    - Mensajes de voz
    - Indicadores de escritura
    - Historial exportable

---

## 🧪 SCRIPTS DE MIGRACIÓN SUGERIDOS

### Migración 1: Agregar campos faltantes

```typescript
// scripts/migrate-chat-schema.ts
import mongoose from 'mongoose';
import Chat from '@/models/Chat';

async function migrateChatSchema() {
  await mongoose.connect(process.env.MONGODB_URI);

  // Agregar chatId a chats existentes
  const chats = await Chat.find({ chatId: { $exists: false } });
  
  for (const chat of chats) {
    // Generar chatId basado en datos existentes
    const participantIds = chat.participants
      .map(p => p.userId.toString())
      .sort()
      .join(':');
    
    const chatType = determineChatType(chat.participants);
    const chatId = `${chatType}:${chat.orderId}:${participantIds}`;
    
    chat.chatId = chatId;
    chat.chatType = chatType;
    chat.isActive = chat.status !== 'archived';
    
    await chat.save();
  }

  console.log(`Migrados ${chats.length} chats`);
}
```

### Migración 2: Eliminar chats duplicados

```typescript
// scripts/cleanup-duplicate-chats.ts
async function cleanupDuplicateChats() {
  const duplicates = await Chat.aggregate([
    {
      $group: {
        _id: { orderId: '$orderId', participants: '$participants' },
        chats: { $push: '$_id' },
        count: { $sum: 1 }
      }
    },
    { $match: { count: { $gt: 1 } } }
  ]);

  for (const dup of duplicates) {
    // Mantener el más reciente, eliminar el resto
    const [keep, ...remove] = dup.chats.sort((a, b) => 
      b.createdAt - a.createdAt
    );
    
    // Combinar mensajes
    const oldChats = await Chat.find({ _id: { $in: remove } });
    const mainChat = await Chat.findById(keep);
    
    for (const old of oldChats) {
      mainChat.messages.push(...old.messages);
    }
    
    mainChat.messages.sort((a, b) => a.createdAt - b.createdAt);
    await mainChat.save();
    
    // Eliminar duplicados
    await Chat.deleteMany({ _id: { $in: remove } });
  }
}
```

---

## 📌 CONCLUSIONES

### Problemas Críticos Identificados
1. ❌ **Campo `chatId` faltante en schema** → Causa creación de chats duplicados
2. ❌ **APIs de drivers con mock data** → No funcional en producción
3. ⚠️ **Duplicación de modelos** → Chat vs Message inconsistente
4. ⚠️ **Falta WebSocket real** → Sin actualizaciones en tiempo real

### Aspectos Positivos
1. ✅ Buena estructura de proyecto Next.js 15
2. ✅ UI bien diseñada con shadcn/ui
3. ✅ Sistema de roles implementado
4. ✅ Modelos de datos generalmente bien diseñados

### Impacto del Bug Principal
- **Severidad:** CRÍTICA
- **Usuarios afectados:** Todos los que usan chat
- **Síntoma:** Cada mensaje crea un nuevo chat
- **Solución:** Agregar campo `chatId` y corregir lógica de búsqueda
- **Tiempo estimado de fix:** 4-6 horas

### Recomendación Final
**Iniciar correcciones inmediatamente** siguiendo el plan de acción en orden de prioridad. El sistema de chat es fundamental para la plataforma y actualmente está severamente comprometido.

---

## 📞 PRÓXIMOS PASOS

1. **Revisar este documento** con el equipo de desarrollo
2. **Priorizar fixes** según impacto en usuarios
3. **Crear branches** para cada fix mayor
4. **Implementar tests** antes de desplegar
5. **Monitorear** después del despliegue

---

**Documento generado automáticamente**  
**Requiere validación humana antes de implementar cambios**
