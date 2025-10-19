# Evidencias (08-ui-state)

- **SSR-safety y Maps – `contexts/GoogleMapsContext.tsx`**

```tsx
1 'use client'
27 const { isLoaded, loadError } = useLoadScript({
28   googleMapsApiKey: process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY || '',
29   libraries: ['places', 'geometry'],
})
```

- **Notifications – `contexts/NotificationContext.tsx`**

```tsx
39 export function NotificationProvider({ children }: { children: ReactNode }) {
41   const { socket, on, off } = useSocket()
42   const [notifications, setNotifications] = useState<Notification[]>([])
57   setNotifications(prev => [notification, ...prev].slice(0, 50))
179 on('notification', handleNewNotification)
181 on('payment-update', handlePaymentUpdate)
192 return () => {
193   off('notification', handleNewNotification)
}
```

- **Loading – `contexts/LoadingContext.tsx`**

```tsx
55 const withLoading = useCallback(async function <T>(key: string, asyncFn: () => Promise<T>): Promise<T> {
56   setLoading(key, true)
61   setLoading(key, false)
```

- **Zustand persist – `store/cart-store.ts`**

```ts
83 export const useCartStore = create<CartState>()(
84   persist(
91   get subtotal() { return get().items.reduce(...) },
96   get discount() { /* O(n) calc */ },
117   get total() { return get().subtotal - get().discount },
210   name: 'cart-storage',
```

- **useSSE – `hooks/use-sse.ts`**

```ts
17 const es = new EventSource(url);
37 return () => { es.close(); sourceRef.current = null; setConnected(false) };
```

- **Driver Location – `hooks/use-driver-location.ts`**

```ts
29 const { location: unifiedLocation, loading, error, watchLocation, clearWatch } = useUnifiedLocation()
52 const startTracking = useCallback(() => {
56   return () => { stopWatching(); setWatchId(null) }
```

- **A11y: Chat – `components/chat/support-chat.tsx`**

```tsx
115 <Button variant="ghost" size="sm" onClick={onToggleMinimize} className="h-8 w-8 p-0">
121   <Minimize2 className="h-4 w-4" />
...
171 <Input placeholder="Escribe tu mensaje..." ... />
179 <Button onClick={sendMessage} disabled={!newMessage.trim() || isTyping} size="sm">
183   <Send className="h-4 w-4" />
```

- **A11y: Progress – `components/cart/free-shipping-progress.tsx`**

```tsx
33 <div className="w-full bg-gray-200 rounded-full h-2">
35   <div className="bg-primary h-2 rounded-full" style={{ width: `${progress}%` }} />
```
