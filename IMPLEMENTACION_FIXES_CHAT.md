# 🔧 GUÍA DE IMPLEMENTACIÓN - CORRECCIÓN SISTEMA DE CHAT

Esta guía contiene las instrucciones para implementar las correcciones identificadas en la auditoría.

---

## 📝 ORDEN DE IMPLEMENTACIÓN

1. ✅ Actualizar modelo `Chat.ts` - Agregar campos faltantes
2. ✅ Ejecutar script de migración de datos existentes
3. ✅ Actualizar APIs problemáticas
4. ✅ Actualizar APIs de drivers (quitar mock data)
5. ✅ Testear flujo completo

---

## 1️⃣ CAMPOS A AGREGAR EN /models/Chat.ts

Agregar estos campos al interface `IChat` y al `ChatSchema`:

### Nuevos campos en interface:
```typescript
interface IChat {
  chatId: string; // ✅ Campo único para identificar chat
  chatType: 'customer_supplier' | 'customer_admin' | 'supplier_admin' | 'support' | 'order' | 'delivery' | 'general';
  isActive: boolean; // ✅ Para soft-delete
  lastMessage?: { 
    content: string;
    timestamp: Date;
    senderName: string;
  };
  // ... resto de campos existentes
}
```

### Nuevos campos en Schema:
```typescript
const ChatSchema = new Schema<IChat>({
  chatId: {
    type: String,
    required: true,
    unique: true,
    index: true,
  },
  chatType: {
    type: String,
    enum: ['customer_supplier', 'customer_admin', 'supplier_admin', 'support', 'order', 'delivery', 'general'],
    required: true,
    index: true,
  },
  isActive: {
    type: Boolean,
    default: true,
    index: true,
  },
  lastMessage: {
    content: { type: String },
    timestamp: { type: Date },
    senderName: { type: String },
  },
  // ... resto de campos existentes
});
```

### Nuevos índices:
```typescript
ChatSchema.index({ chatId: 1 }, { unique: true });
ChatSchema.index({ chatType: 1, orderId: 1, isActive: 1 });
```

### Nuevo método estático:
```typescript
ChatSchema.statics.findOrCreateByOrder = async function (data: {
  orderId: string;
  customerId: string;
  supplierId?: string;
  driverId?: string;
  title?: string;
}) {
  const { orderId, customerId, supplierId, driverId } = data;
  
  let chatType: string;
  let participantIds: string[];
  
  if (supplierId && customerId) {
    chatType = 'customer_supplier';
    participantIds = [customerId, supplierId].sort();
  } else if (customerId && driverId) {
    chatType = 'order';
    participantIds = [customerId, driverId].sort();
  } else {
    chatType = 'customer_admin';
    participantIds = [customerId].sort();
  }
  
  const chatId = `${chatType}:${orderId}:${participantIds.join(':')}`;
  
  let chat = await this.findOne({ chatId });
  
  if (!chat) {
    // Crear nuevo chat con participantes
    chat = await this.create({
      chatId,
      chatType,
      type: 'order',
      title: data.title || `Chat - Pedido ${orderId}`,
      participants: [/* crear participantes */],
      messages: [],
      orderId,
      status: 'active',
      isActive: true,
    });
  }
  
  return chat;
};
```

---

## 2️⃣ CORRECCIONES EN /api/chat/route.ts

### Línea 26 - Corregir query type:
```typescript
// ❌ ANTES:
if (type) query.chatType = type; // chatType no existía

// ✅ DESPUÉS:
if (type) query.chatType = type; // Ahora existe en el schema
```

### Línea 142-161 - Corregir creación de chat:
```typescript
// ❌ ANTES:
const chatIdKey = `${chatType}:${orderId}:${idsForKey}`;
let chat = await Chat.findOne({ chatId: chatIdKey }); // ❌ chatId no existía
if (!chat) {
  chat = await Chat.create({
    chatId: chatIdKey, // ❌ campo no definido
    // ...
  });
}

// ✅ DESPUÉS:
const chat = await Chat.findOrCreateByOrder({
  orderId,
  customerId,
  supplierId,
  title: `Chat - Pedido ${orderId}`,
});

// Usar método del modelo
await chat.addMessage({
  senderId: user.id,
  senderRole: user.role,
  senderName: (user as any).name || 'Admin',
  content: message,
  type: messageType,
});
```

---

## 3️⃣ CORRECCIONES EN /api/chat/[chatId]/messages/route.ts

### GET - Línea 21-25:
```typescript
// ❌ ANTES:
const chat = await Chat.findOne({
  chatId, // ❌ campo no existía
  isActive: true, // ❌ campo no existía
  'participants.userId': user.id,
});

// ✅ DESPUÉS:
const chat = await Chat.findOne({
  chatId, // ✅ ahora existe
  isActive: true, // ✅ ahora existe
  'participants.userId': user.id,
  'participants.isActive': true,
});
```

### POST - Usar método addMessage:
```typescript
// ✅ Reemplazar lógica manual por método del modelo
const newMessage = await chat.addMessage({
  senderId: user.id,
  senderRole: user.role,
  senderName: (user as any).name || 'Usuario',
  content,
  type: messageType,
});
```

---

## 4️⃣ SCRIPT DE MIGRACIÓN

Ver archivo separado: `MIGRACION_CHAT_SCHEMA.md`

---

## 5️⃣ TESTING

### Test manual después de implementar:

```bash
# 1. Crear un chat nuevo
curl -X POST http://localhost:3000/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "ORDER123",
    "customerId": "USER1",
    "supplierId": "USER2",
    "message": "Hola"
  }'

# 2. Enviar otro mensaje al MISMO chat
curl -X POST http://localhost:3000/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "ORDER123",
    "customerId": "USER1",
    "supplierId": "USER2",
    "message": "Segundo mensaje"
  }'

# 3. Verificar que se reutiliza el mismo chat
# Debería devolver el mismo chat con 2 mensajes
```

---

## 📋 CHECKLIST DE IMPLEMENTACIÓN

- [ ] Actualizar `/models/Chat.ts` con nuevos campos
- [ ] Ejecutar migración de datos existentes
- [ ] Actualizar `/api/chat/route.ts`
- [ ] Actualizar `/api/chat/[chatId]/messages/route.ts`
- [ ] Actualizar `/api/admin/chat/route.ts`
- [ ] Actualizar `/api/drivers/chat/conversations/route.ts`
- [ ] Ejecutar tests manuales
- [ ] Verificar en base de datos que no se crean duplicados
- [ ] Monitorear logs por errores

---

**Ver archivos complementarios:**
- `AUDITORIA_SISTEMA_CHAT.md` - Análisis completo del problema
- `MIGRACION_CHAT_SCHEMA.md` - Script de migración detallado
