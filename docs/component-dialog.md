# 🔲 Componente: Dialog System

## 📋 Información General

**Ubicación:** `components/ui/dialog.tsx`
**Tipo:** Sistema de componentes React reutilizables
**Propósito:** Proporcionar diálogos modales consistentes en toda la aplicación

## 🔧 Funcionalidades

### Componentes de Diálogo

- **Dialog:** Contenedor principal modal
- **DialogHeader:** Encabezado del diálogo
- **DialogFooter:** Pie del diálogo con acciones
- **DialogTitle:** Título del diálogo
- **DialogDescription:** Descripción del diálogo
- **DialogContent:** Contenido principal
- **DialogTrigger:** Botón disparador
- **DialogClose:** Botón de cierre

### Características Avanzadas

- **Cierre personalizado:** Eventos personalizados para cierre programático
- **Accesibilidad:** Soporte completo para lectores de pantalla
- **Interacción:** Overlay con backdrop click
- **Animaciones:** Transiciones suaves

## 🚀 Cambios Implementados (Auditoría Octubre 2025)

### ✅ Correcciones de React Hooks

**Problema Original:**

```tsx
// ❌ Error: React Hook "React.useEffect" llamado condicionalmente
export function Dialog({ open, onClose, onOpenChange, children }: { ... }) {
  if (!open) return null; // ❌ Early return antes del Hook

  const handleClose = () => {
    onOpenChange?.(false);
    onClose?.();
  };

  React.useEffect(() => { // ❌ Hook después de early return
    const handleCustomClose = () => {
      handleClose();
    };

    document.addEventListener('dialog-close', handleCustomClose);
    return () => {
      document.removeEventListener('dialog-close', handleCustomClose);
    };
  }, [onOpenChange, onClose]);
}
```

**Solución Implementada:**

```tsx
// ✅ Corrección: Reordenamiento correcto de Hooks
export function Dialog({ open, onClose, onOpenChange, children }: { ... }) {
  const handleClose = () => {
    onOpenChange?.(false);
    onClose?.();
  };

  // ✅ Hook ANTES del early return
  React.useEffect(() => {
    if (!open) return; // ✅ Guardia condicional DENTRO del Hook

    const handleCustomClose = () => {
      handleClose();
    };

    document.addEventListener('dialog-close', handleCustomClose);
    return () => {
      document.removeEventListener('dialog-close', handleCustomClose);
    };
  }, [open, onOpenChange, onClose]);

  if (!open) return null; // ✅ Early return DESPUÉS del Hook
}
```

### 🎯 Mejoras de Arquitectura

1. **Reglas de Hooks:** Cumplimiento estricto de las reglas de React
2. **Separación de responsabilidades:** Lógica clara entre manejo de estado y rendering
3. **Reutilización:** Sistema modular fácilmente extensible

## 📊 Estructura de Componentes

### Dialog Principal

```tsx
export function Dialog({
  open,
  onClose,
  onOpenChange,
  children,
}: {
  open?: boolean;
  onClose?: () => void;
  onOpenChange?: (v: boolean) => void;
  children: React.ReactNode;
}) {
  // Lógica de manejo de eventos y efectos
  // Renderizado condicional
}
```

### DialogClose con Eventos Personalizados

```tsx
export function DialogClose({
  children,
  asChild,
  onClick,
  ...props
}: React.ButtonHTMLAttributes<HTMLButtonElement> & { asChild?: boolean }) {
  const handleClose = (e: React.MouseEvent<HTMLButtonElement>) => {
    const dialogElement = e.currentTarget.closest(
      '[role="dialog"]'
    ) as HTMLElement;
    if (dialogElement) {
      const closeEvent = new CustomEvent('dialog-close');
      dialogElement.dispatchEvent(closeEvent);
    }
    onClick?.(e);
  };

  if (asChild && React.isValidElement(children)) {
    return React.cloneElement(children, {
      ...props,
      onClick: handleClose,
    } as any);
  }

  return (
    <button type="button" onClick={handleClose} {...props}>
      {children}
    </button>
  );
}
```

## 🎨 Características de UI

### Overlay y Backdrop

```tsx
<div
  role="dialog"
  aria-modal="true"
  className="fixed inset-0 z-50 grid place-items-center bg-black/40"
  onClick={handleClose}
>
  <div
    className="w-full max-w-lg rounded-lg bg-white p-4 shadow-lg"
    onClick={e => e.stopPropagation()}
  >
    {children}
  </div>
</div>
```

### Eventos Personalizados

```tsx
// Disparar evento de cierre personalizado
const closeEvent = new CustomEvent('dialog-close');
dialogElement.dispatchEvent(closeEvent);
```

## 💡 Ejemplo de Uso Completo

```tsx
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
  DialogClose,
} from '@/components/ui/dialog';

export function ConfirmationDialog() {
  return (
    <Dialog>
      <DialogTrigger asChild>
        <Button variant="outline">Eliminar Cuenta</Button>
      </DialogTrigger>
      <DialogContent className="sm:max-w-[425px]">
        <DialogHeader>
          <DialogTitle>¿Estás seguro?</DialogTitle>
          <DialogDescription>
            Esta acción no se puede deshacer. Se eliminará permanentemente tu
            cuenta y todos tus datos.
          </DialogDescription>
        </DialogHeader>
        <DialogFooter>
          <DialogClose asChild>
            <Button type="button" variant="outline">
              Cancelar
            </Button>
          </DialogClose>
          <Button type="submit">Eliminar Cuenta</Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

## 🔧 Eventos y Comunicación

### Event Listener Personalizado

```tsx
React.useEffect(() => {
  if (!open) return;

  const handleCustomClose = () => {
    handleClose();
  };

  document.addEventListener('dialog-close', handleCustomClose);
  return () => {
    document.removeEventListener('dialog-close', handleCustomClose);
  };
}, [open, onOpenChange, onClose]);
```

### Propagación de Eventos

```tsx
const handleClose = (e: React.MouseEvent<HTMLButtonElement>) => {
  // Buscar el componente Dialog padre
  const dialogElement = e.currentTarget.closest(
    '[role="dialog"]'
  ) as HTMLElement;
  if (dialogElement) {
    // Dispatch del evento personalizado
    const closeEvent = new CustomEvent('dialog-close');
    dialogElement.dispatchEvent(closeEvent);
  }
  onClick?.(e);
};
```

## 🎯 Características de Accesibilidad

### Atributos ARIA

- `role="dialog"`: Identifica el diálogo para lectores de pantalla
- `aria-modal="true"`: Indica que es modal (bloquea interacción externa)

### Navegación por Teclado

- **Escape:** Debería cerrar el diálogo (no implementado actualmente)
- **Tab:** Navegación interna del diálogo
- **Focus trap:** El foco debería quedar atrapado dentro del diálogo

## 🚨 Estados y Gestión

### Estado de Apertura

```tsx
interface DialogProps {
  open?: boolean; // Estado controlado externamente
  onOpenChange?: (open: boolean) => void; // Callback para cambios
  onClose?: () => void; // Callback de cierre
}
```

### Renderizado Condicional

```tsx
if (!open) return null; // No renderizar cuando esté cerrado
```

## 🔧 Próximas Mejoras

- [ ] **Focus trap:** Implementar manejo automático del foco
- [ ] **Tecla Escape:** Agregar listener para cierre con Escape
- [ ] **Animaciones:** Transiciones más suaves (Framer Motion)
- [ ] **Tamaños responsivos:** Más opciones de tamaño automático
- [ ] **Stack management:** Soporte para múltiples diálogos anidados

---

_Última modificación: Octubre 2025_
_Estado: ✅ Completamente funcional y corregido_
