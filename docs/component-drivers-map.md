# 🗺️ Componente: Drivers Live Map

## 📋 Información General

**Ubicación:** `components/admin/drivers-live-map.tsx`
**Tipo:** Componente React con hooks
**Propósito:** Visualización en tiempo real de conductores activos en mapa

## 🔧 Funcionalidades

### Mapa Interactivo

- **Google Maps integration:** Visualización de ubicaciones de conductores
- **Marcadores dinámicos:** Íconos diferenciados por estado de conductor
- **Información contextual:** Ventanas emergentes con detalles del conductor
- **Interacción del usuario:** Click para ver detalles completos

### Estados de Conductor

- **Disponible:** Ícono verde - Conductor libre para asignaciones
- **Ocupado:** Ícono rojo - Conductor en ruta o con pedido asignado

## 🚀 Cambios Implementados (Auditoría Octubre 2025)

### ✅ Correcciones de TypeScript

**Problema Original:**

```typescript
// ❌ Error: Argument of type 'string' not assignable to '"available" | "busy"'
icon={getDriverIcon(driver.status)}

// El tipo de driver.status no estaba correctamente tipado
```

**Solución Implementada:**

```typescript
// ✅ Corrección: Interfaces tipadas correctamente
type DriverInfo = {
  id: string;
  name: string;
  location: { lat: number; lng: number };
  status: 'available' | 'busy';
  currentOrder: string | null;
};

const [liveDrivers, setLiveDrivers] = React.useState<DriverInfo[]>(
  drivers.length > 0
    ? drivers.map(d => ({
        id: d._id,
        name: d.name,
        location: d.currentLocation
          ? {
              lat: d.currentLocation.lat,
              lng: d.currentLocation.lng,
            }
          : { lat: -24.7821, lng: -65.4232 },
        status: 'available' as 'available' | 'busy',
        currentOrder: null,
      }))
    : activeDrivers
);
```

### 🎨 Mejoras de UI/UX

1. **Tipado Estricto:** Interfaces específicas para información de conductores
2. **Íconos Dinámicos:** Generación SVG basada en estado del conductor
3. **Información Contextual:** Ventanas emergentes con detalles relevantes

## 📊 Estructura de Datos

### Props del Componente

```typescript
interface DriversLiveMapProps {
  drivers?: Array<{
    _id: string;
    name: string;
    currentLocation: { lat: number; lng: number } | null;
  }>;
  height?: string;
  fitToMarkers?: boolean;
  fitTrigger?: number;
  centerOn?: { lat: number; lng: number } | null;
}
```

### Información de Conductor

```typescript
interface DriverInfo {
  id: string;
  name: string;
  location: { lat: number; lng: number };
  status: 'available' | 'busy';
  currentOrder: string | null;
}
```

## 🗺️ Funcionalidades del Mapa

### Íconos de Conductor

```typescript
const getDriverIcon = (status: 'available' | 'busy') => {
  const color = status === 'available' ? '#10B981' : '#EF4444';
  return {
    url: `data:image/svg+xml;charset=UTF-8,${encodeURIComponent(`
      <svg width="40" height="40" viewBox="0 0 40 40" xmlns="http://www.w3.org/2000/svg">
        <circle cx="20" cy="20" r="18" fill="${color}" stroke="white" stroke-width="4"/>
        <circle cx="20" cy="20" r="8" fill="white"/>
        <path d="M15 18 L18 21 L25 14" stroke="${color}" stroke-width="2" fill="none"/>
      </svg>
    `)}`,
    scaledSize: new google.maps.Size(40, 40),
    anchor: new google.maps.Point(20, 20),
  };
};
```

### Información en Popup

```typescript
{selectedDriver && (
  <InfoWindow position={selectedDriver.location}>
    <div className="p-2 min-w-48">
      <h4 className="font-semibold text-sm">{selectedDriver.name}</h4>
      <p className="text-xs text-gray-600 capitalize">
        Estado: {selectedDriver.status === 'available' ? 'Disponible' : 'Ocupado'}
      </p>
      {selectedDriver.currentOrder && (
        <p className="text-xs text-blue-600">
          Orden actual: {selectedDriver.currentOrder}
        </p>
      )}
    </div>
  </InfoWindow>
)}
```

## 🔧 Configuración de Google Maps

### Variables de Entorno

```bash
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=your_google_maps_api_key
```

### Configuración del Mapa

```typescript
<GoogleMap
  mapContainerStyle={containerStyle}
  center={defaultCenter}
  zoom={13}
  options={{
    zoomControl: true,
    streetViewControl: false,
    mapTypeControl: false,
    fullscreenControl: false,
  }}
>
```

## 🚨 Estados y Gestión

### Estados del Componente

```typescript
const [isLoaded, setIsLoaded] = React.useState(false);
const [selectedDriver, setSelectedDriver] = React.useState<DriverInfo | null>(
  null
);
const [liveDrivers, setLiveDrivers] =
  React.useState<DriverInfo[]>(/* datos iniciales */);
```

### Centro por Defecto (Salta, Argentina)

```typescript
const defaultCenter: LatLng = {
  lat: -24.7821,
  lng: -65.4232,
};
```

## 💡 Ejemplo de Uso

```tsx
import DriversLiveMap from '@/components/admin/drivers-live-map';

// Uso básico
<DriversLiveMap />

// Con datos personalizados
<DriversLiveMap
  drivers={[
    {
      _id: '1',
      name: 'Carlos Rodríguez',
      currentLocation: { lat: -24.7821, lng: -65.4232 }
    }
  ]}
  height="400px"
/>

// Con ajuste automático de zoom
<DriversLiveMap
  fitToMarkers={true}
  fitTrigger={drivers.length}
/>
```

## 🎨 Características Visuales

### Colores por Estado

- **Disponible:** Verde (#10B981)
- **Ocupado:** Rojo (#EF4444)

### Información Mostrada

- Nombre del conductor
- Estado actual (Disponible/Ocupado)
- Orden actual (si aplica)
- Botones de acción (Llamar, Asignar Orden)

## 🔄 Datos de Ejemplo

```typescript
const activeDrivers: DriverInfo[] = [
  {
    id: '1',
    name: 'Carlos Rodríguez',
    location: { lat: -24.7821, lng: -65.4232 },
    status: 'available',
    currentOrder: null,
  },
  {
    id: '2',
    name: 'María González',
    location: { lat: -24.785, lng: -65.42 },
    status: 'busy',
    currentOrder: 'ORD-001',
  },
];
```

## 📱 Responsive Design

- **Altura adaptable:** Configurable vía prop `height`
- **Contenedor responsive:** Se adapta al ancho disponible
- **Controles optimizados:** Zoom y navegación táctil

## 🔧 Próximas Mejoras

- [ ] Seguimiento en tiempo real con WebSockets
- [ ] Filtros por zona geográfica
- [ ] Información de ruta actual del conductor
- [ ] Estadísticas de rendimiento por conductor

---

_Última modificación: Octubre 2025_
_Estado: ✅ Completamente funcional y corregido_
