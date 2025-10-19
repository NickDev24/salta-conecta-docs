# 🗄️ SCRIPT DE MIGRACIÓN - CHAT SCHEMA

Script para migrar los chats existentes y agregar los campos faltantes.

---

## 📁 CREAR ARCHIVO migrations/001-add-chatId-field.ts

```typescript
import { connectToDatabase } from '@/lib/mongodb';
import Chat from '@/models/Chat';
import mongoose from 'mongoose';

interface ParticipantData {
  userId: string;
  role: string;
}

function determineChatType(participants: ParticipantData[]): string {
  const roles = participants.map(p => p.role);

  if (roles.includes('customer') && roles.includes('supplier')) {
    return 'customer_supplier';
  } else if (roles.includes('customer') && roles.includes('admin')) {
    return 'customer_admin';
  } else if (roles.includes('supplier') && roles.includes('admin')) {
    return 'supplier_admin';
  } else if (roles.includes('driver')) {
    return 'order';
  } else if (roles.includes('support')) {
    return 'support';
  }

  return 'general';
}

export async function migrateChatSchema() {
  try {
    console.log('🔄 Iniciando migración de schema de Chat...');
    console.log('📅', new Date().toISOString());

    // Conectar a base de datos
    await connectToDatabase();

    // Buscar chats sin los nuevos campos
    const chatsToMigrate = await Chat.find({
      $or: [
        { chatId: { $exists: false } },
        { chatType: { $exists: false } },
        { isActive: { $exists: false } },
      ],
    });

    console.log(`📊 Encontrados ${chatsToMigrate.length} chats para migrar\n`);

    if (chatsToMigrate.length === 0) {
      console.log('✅ No hay chats para migrar');
      return { success: true, migrated: 0, duplicates: 0, total: 0 };
    }

    let migrated = 0;
    let duplicates = 0;
    let errors = 0;
    const seenChatIds = new Set<string>();
    const duplicateList: any[] = [];

    for (const chat of chatsToMigrate) {
      try {
        // Extraer IDs de participantes
        const participantIds = chat.participants
          .map((p: any) => p.userId.toString())
          .sort()
          .join(':');

        // Determinar tipo de chat
        const chatType = determineChatType(
          chat.participants.map((p: any) => ({
            userId: p.userId.toString(),
            role: p.role,
          }))
        );

        // Generar chatId único
        const orderId = chat.orderId ? chat.orderId.toString() : 'no-order';
        const chatId = `${chatType}:${orderId}:${participantIds}`;

        // Detectar duplicados
        if (seenChatIds.has(chatId)) {
          console.warn(`⚠️  Chat duplicado: ${chatId}`);
          console.warn(`   ID: ${chat._id}`);
          duplicates++;
          duplicateList.push({
            chatId,
            _id: chat._id,
            orderId: chat.orderId,
          });
          
          // Marcar como inactivo en lugar de eliminar
          chat.isActive = false;
          chat.status = 'archived';
        } else {
          seenChatIds.add(chatId);
          chat.isActive = chat.status !== 'archived';
        }

        // Actualizar campos
        chat.chatId = chatId;
        chat.chatType = chatType as any;

        // Migrar lastMessage si no existe
        if (!chat.lastMessage && chat.messages.length > 0) {
          const lastMsg = chat.messages[chat.messages.length - 1];
          chat.lastMessage = {
            content: lastMsg.content,
            timestamp: lastMsg.createdAt || new Date(),
            senderName: (lastMsg as any).senderName || 'Usuario',
          };
        }

        // Guardar cambios
        await chat.save();
        migrated++;

        // Progreso
        if (migrated % 10 === 0) {
          console.log(`   ✓ Migrados ${migrated}/${chatsToMigrate.length} chats`);
        }
      } catch (error: any) {
        console.error(`❌ Error migrando chat ${chat._id}:`, error.message);
        errors++;
      }
    }

    console.log(`\n✅ Migración completada:`);
    console.log(`   - Chats migrados: ${migrated}`);
    console.log(`   - Duplicados encontrados: ${duplicates}`);
    console.log(`   - Errores: ${errors}`);

    if (duplicateList.length > 0) {
      console.log(`\n📋 Lista de duplicados (marcados como inactivos):`);
      duplicateList.forEach(dup => {
        console.log(`   - ChatID: ${dup.chatId}`);
        console.log(`     MongoDB ID: ${dup._id}`);
        console.log(`     Order: ${dup.orderId || 'N/A'}`);
      });
    }

    // Crear índices
    console.log('\n📑 Creando índices...');
    try {
      await Chat.collection.createIndex({ chatId: 1 }, { unique: true });
      await Chat.collection.createIndex({ chatType: 1, orderId: 1, isActive: 1 });
      await Chat.collection.createIndex({ 'participants.userId': 1, isActive: 1 });
      console.log('✅ Índices creados correctamente');
    } catch (indexError: any) {
      console.warn('⚠️  Error creando índices (puede que ya existan):', indexError.message);
    }

    return {
      success: true,
      migrated,
      duplicates,
      errors,
      total: chatsToMigrate.length,
      duplicateList,
    };
  } catch (error: any) {
    console.error('❌ Error en migración:', error);
    throw error;
  } finally {
    await mongoose.disconnect();
    console.log('\n🔌 Desconectado de la base de datos');
  }
}

// Ejecutar si se llama directamente
if (require.main === module) {
  migrateChatSchema()
    .then(result => {
      console.log('\n🎉 Migración exitosa');
      console.log('📊 Resumen:', JSON.stringify(result, null, 2));
      process.exit(0);
    })
    .catch(error => {
      console.error('\n💥 Migración fallida:', error);
      process.exit(1);
    });
}

export default migrateChatSchema;
```

---

## 🚀 CÓMO EJECUTAR LA MIGRACIÓN

### Opción 1: Con tsx (Recomendado)
```bash
npx tsx migrations/001-add-chatId-field.ts
```

### Opción 2: Con ts-node
```bash
npx ts-node migrations/001-add-chatId-field.ts
```

### Opción 3: Desde código
```typescript
import migrateChatSchema from './migrations/001-add-chatId-field';

await migrateChatSchema();
```

---

## ⚙️ VARIABLES DE ENTORNO REQUERIDAS

Asegúrate de tener configurado `.env`:

```env
MONGODB_URI=mongodb://localhost:27017/salta-conecta
# o tu URI de MongoDB Atlas
```

---

## 📋 QUÉ HACE LA MIGRACIÓN

1. **Conecta a MongoDB** usando la configuración existente
2. **Busca chats sin `chatId`** o campos faltantes
3. **Genera `chatId` único** basado en:
   - Tipo de chat (customer_supplier, etc.)
   - ID de la orden
   - IDs de participantes (ordenados)
4. **Detecta duplicados** usando el `chatId` generado
5. **Marca duplicados como inactivos** (no los elimina)
6. **Migra `lastMessage`** si no existe
7. **Crea índices** para mejorar performance
8. **Genera reporte** detallado

---

## 🔍 VERIFICAR MIGRACIÓN

Después de ejecutar, verifica en MongoDB:

```javascript
// En MongoDB Compass o mongosh

// Ver chats con los nuevos campos
db.chats.find({ 
  chatId: { $exists: true },
  chatType: { $exists: true }
}).limit(5);

// Contar chats activos vs inactivos
db.chats.aggregate([
  { $group: { 
    _id: "$isActive", 
    count: { $sum: 1 } 
  }}
]);

// Ver índices creados
db.chats.getIndexes();
```

---

## 🛡️ ROLLBACK (En caso de error)

Si algo sale mal, puedes hacer rollback:

```javascript
// Eliminar campos agregados
db.chats.updateMany(
  {},
  { 
    $unset: { 
      chatId: "",
      chatType: "",
      isActive: "",
      lastMessage: ""
    }
  }
);

// Eliminar índices
db.chats.dropIndex("chatId_1");
db.chats.dropIndex("chatType_1_orderId_1_isActive_1");
```

---

## 📊 OUTPUT ESPERADO

```
🔄 Iniciando migración de schema de Chat...
📅 2025-01-19T10:30:00.000Z
📊 Encontrados 127 chats para migrar

   ✓ Migrados 10/127 chats
   ✓ Migrados 20/127 chats
   ...
   ✓ Migrados 120/127 chats

⚠️  Chat duplicado: customer_supplier:ORDER123:USER1:USER2
   ID: 507f1f77bcf86cd799439011

✅ Migración completada:
   - Chats migrados: 125
   - Duplicados encontrados: 2
   - Errores: 0

📋 Lista de duplicados (marcados como inactivos):
   - ChatID: customer_supplier:ORDER123:USER1:USER2
     MongoDB ID: 507f1f77bcf86cd799439011
     Order: ORDER123

📑 Creando índices...
✅ Índices creados correctamente

🔌 Desconectado de la base de datos

🎉 Migración exitosa
📊 Resumen: {
  "success": true,
  "migrated": 125,
  "duplicates": 2,
  "errors": 0,
  "total": 127
}
```

---

## ⚠️ CONSIDERACIONES IMPORTANTES

1. **Backup primero**: Haz backup de tu base de datos antes de ejecutar
```bash
mongodump --uri="mongodb://localhost:27017/salta-conecta" --out=./backup
```

2. **Ejecuta en desarrollo primero**: Prueba en entorno de desarrollo

3. **Detén la aplicación**: Para evitar conflictos, detén la app durante la migración

4. **Duplicados**: Los chats duplicados se marcan como `isActive: false` pero NO se eliminan

5. **Puede tardar**: Si tienes muchos chats, la migración puede tardar varios minutos

---

## 🧪 TESTING POST-MIGRACIÓN

Después de la migración, ejecuta estos tests:

```bash
# 1. Verificar que no se crean nuevos duplicados
npm run test:chat-creation

# 2. Verificar que los chats existentes se reutilizan
npm run test:chat-reuse

# 3. Verificar panel admin
npm run test:admin-chat
```

---

## 📞 PRÓXIMOS PASOS

1. ✅ Ejecutar migración
2. ✅ Verificar en base de datos
3. ✅ Actualizar código de APIs (ver IMPLEMENTACION_FIXES_CHAT.md)
4. ✅ Testear flujo completo
5. ✅ Desplegar a producción

---

**IMPORTANTE:** Este script es idempotente. Puedes ejecutarlo múltiples veces sin problemas.
