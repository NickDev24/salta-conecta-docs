# 🚀 SaltaConecta - Plataforma B2B

SaltaConecta es una plataforma B2B completa que conecta mayoristas, proveedores y repartidores en Salta, Argentina.

## 📋 Características Principales

### 🛒 **E-commerce B2B**

- Catálogo de productos con aprobación de administradores
- Carrito de compras inteligente
- Sistema de pedidos completo
- Gestión de inventario en tiempo real

### 💳 **Pagos Integrados**

- Integración completa con MercadoPago
- Webhooks para confirmación de pagos
- Sistema de comisiones automático
- Pagos a proveedores

### 🚚 **Logística**

- Sistema de repartidores
- Tracking en tiempo real
- Optimización de rutas
- QR codes para entregas

### 🎯 **Gestión de Usuarios**

- Multi-rol: Clientes, Proveedores, Repartidores, Admins
- Autenticación segura con NextAuth
- Perfiles personalizados
- Sistema de notificaciones

### 🎰 **Gamificación**

- Ruleta de premios
- Sistema de puntos y recompensas
- Cupones de descuento
- Campañas promocionales

## 🛠️ Tecnologías

### **Frontend**

- **Next.js 15** - Framework React con App Router
- **TypeScript** - Tipado estático
- **Tailwind CSS** - Estilos utilitarios
- **Framer Motion** - Animaciones
- **Radix UI** - Componentes accesibles

### **Backend**

- **Next.js API Routes** - API REST
- **MongoDB** - Base de datos NoSQL
- **Mongoose** - ODM para MongoDB
- **NextAuth** - Autenticación
- **Socket.io** - WebSockets en tiempo real

### **Integraciones**

- **MercadoPago** - Pagos
- **Cloudinary** - Gestión de imágenes
- **Google Maps** - Mapas y geolocalización
- **Resend** - Emails transaccionales
- **Redis** - Cache y sesiones

### **DevOps**

- **Docker** - Containerización
- **Vercel** - Despliegue
- **Jest** - Testing
- **ESLint** - Linting
- **Prettier** - Formateo de código

## 🚀 Instalación y Configuración

### **Prerrequisitos**

- Node.js 18+
- MongoDB
- Redis (opcional)
- Cuentas de servicios externos

### **1. Clonar el repositorio**

```bash
git clone https://github.com/tu-usuario/salta-conecta.git
cd salta-conecta
```

### **2. Instalar dependencias**

```bash
npm install
```

### **3. Configurar variables de entorno**

```bash
cp env.example .env.local
```

Editar `.env.local` con tus credenciales:

```env
# Base de datos
MONGODB_URI=mongodb://localhost:27017/saltaconecta

# Autenticación
NEXTAUTH_SECRET=tu-secreto-aqui
NEXTAUTH_URL=http://localhost:3000

# MercadoPago
MERCADOPAGO_ACCESS_TOKEN=tu-token-mercadopago

# Cloudinary
CLOUDINARY_CLOUD_NAME=tu-cloud-name
CLOUDINARY_API_KEY=tu-api-key
CLOUDINARY_API_SECRET=tu-api-secret

# Google Maps
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=tu-google-maps-key

# Email
RESEND_API_KEY=tu-resend-key
```

### **4. Configurar base de datos**

```bash
# Iniciar MongoDB
mongod

# Ejecutar migraciones (si las hay)
npm run db:migrate

# Poblar datos de prueba (opcional)
npm run db:seed
```

### **5. Ejecutar en desarrollo**

```bash
npm run dev
```

La aplicación estará disponible en `http://localhost:3000`

## 🧪 Testing

### **Ejecutar todos los tests**

```bash
npm test
```

### **Tests con cobertura**

```bash
npm run test:coverage
```

### **Tests en modo watch**

```bash
npm run test:watch
```

## 🏗️ Scripts Disponibles

```bash
# Desarrollo
npm run dev          # Servidor de desarrollo
npm run build        # Build de producción
npm run start        # Servidor de producción
npm run lint         # Linting
npm run type-check   # Verificación de tipos

# Base de datos
npm run db:migrate   # Ejecutar migraciones
npm run db:seed      # Poblar datos

# Testing
npm test             # Ejecutar tests
npm run test:watch   # Tests en modo watch
npm run test:coverage # Tests con cobertura

# Utilidades
npm run clean:build  # Limpiar build
```

## 📁 Estructura del Proyecto

```
salta-conecta/
├── app/                    # App Router de Next.js
│   ├── api/               # API Routes
│   ├── admin/             # Panel de administración
│   ├── auth/              # Autenticación
│   ├── client/            # Panel de cliente
│   ├── driver/            # Panel de repartidor
│   ├── provider/          # Panel de proveedor
│   └── ...
├── components/            # Componentes React
│   ├── ui/               # Componentes base
│   ├── sections/         # Secciones de página
│   └── ...
├── lib/                  # Utilidades y configuraciones
├── models/               # Modelos de MongoDB
├── __tests__/            # Tests
├── scripts/              # Scripts de utilidad
└── ...
```

## 🔐 Seguridad

### **Autenticación y Autorización**

- NextAuth con JWT
- Middleware de protección de rutas
- Roles y permisos granulares
- Rate limiting

### **Validación de Datos**

- Zod para validación de esquemas
- Sanitización de inputs
- Validación de archivos

### **Seguridad de APIs**

- CORS configurado
- Headers de seguridad
- Validación de webhooks
- Logging de actividades

## 🚀 Despliegue

### **Vercel (Recomendado)**

1. Conectar repositorio a Vercel
2. Configurar variables de entorno
3. Desplegar automáticamente

### **Docker**

```bash
# Build de la imagen
docker build -t salta-conecta .

# Ejecutar con docker-compose
docker-compose up -d
```

### **Variables de Entorno de Producción**

```env
NODE_ENV=production
NEXT_PUBLIC_APP_URL=https://tu-dominio.com
NEXTAUTH_URL=https://tu-dominio.com
MONGODB_URI=mongodb://tu-mongo-uri
MERCADOPAGO_ACCESS_TOKEN=tu-token-produccion
# ... otras variables
```

## 📊 Monitoreo

### **Logs**

- Logging estructurado con Winston
- Logs de errores y actividades
- Métricas de performance

### **Health Checks**

- `/api/health` - Estado del sistema
- `/api/system/health` - Estado detallado

## 🤝 Contribución

1. Fork del proyecto
2. Crear rama feature (`git checkout -b feature/nueva-funcionalidad`)
3. Commit cambios (`git commit -m 'Agregar nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/nueva-funcionalidad`)
5. Crear Pull Request

## 📝 Licencia

Este proyecto está bajo la Licencia MIT - ver el archivo [LICENSE](LICENSE) para detalles.

## 📞 Soporte

Para soporte técnico o preguntas:

- Email: soporte@salta-conecta.com
- Documentación: [docs.salta-conecta.com](https://docs.salta-conecta.com)

## 🎯 Roadmap

### **Próximas Funcionalidades**

- [ ] App móvil nativa
- [ ] Integración con más métodos de pago
- [ ] Sistema de facturación electrónica
- [ ] Analytics avanzados
- [ ] API pública para integraciones
- [ ] Sistema de afiliados
- [ ] Marketplace de servicios

---

**Desarrollado con ❤️ para la comunidad de Salta**
