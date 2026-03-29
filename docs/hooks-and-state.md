---
title: Hooks & State
version: 2.0
---

# Hooks y gestión de estado

Los hooks actúan como puente entre las pantallas (Screens) y la lógica de negocio (Services). Gestionan el estado de React y formatean los datos para la vista.

---

## Reglas

- ✅ El nombre siempre empieza con `use`
- ✅ Llaman a `services/` y gestionan estados de carga/error
- ✅ Lógica independiente de componentes (reutilizable)
- ✅ Retornan datos y funciones tipados
- ✅ Agrupan lógica por entidad (ej: `useProducts` agrupa todo lo de productos)
- ❌ No hacer lógica de UI
- ❌ No retornar componentes JSX
- ❌ No tener side effects innecesarios

---

## Estructura

```
hooks/
├── useAuth.ts         ← Login/logout + estado de sesión
├── useFetch.ts        ← Hook genérico para cualquier llamada puntual
├── useProducts.ts     ← Ejemplo con React Query
├── useStorage.ts      ← Leer/escribir storage local
└── useFormState.ts    ← Gestión de formularios
```

---

## React Query (TanStack Query)

### ¿Qué es y por qué usarla?

React Query es la librería que gestiona el estado del servidor en React: caching, refetch automático, sincronización y control de errores. Sin ella, hay que gestionar manualmente `loading`, `error` y `data` en cada hook.

**Ventajas clave en React Native:**
- **Cache automática**: navegar a otra pantalla y volver no refetcha si los datos son recientes
- **Refetch al recuperar foco**: cuando el usuario vuelve a la app desde background, los datos se actualizan solos
- **`invalidateQueries`**: cuando creas, editas o borras un recurso, invalidas la cache y el dato se refresca en toda la app
- **Reintentos automáticos**: si falla la conexión, reintenta sin código extra
- **Estados unificados**: `isLoading`, `isError`, `data` en una sola llamada

### Setup obligatorio en `App.tsx`

```typescript
// src/App.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { NavigationContainer } from '@react-navigation/native'
import { RootNavigator } from '@/navigation/RootNavigator'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutos de cache por defecto
      retry: 2,                  // reintentar 2 veces si falla
    },
  },
})

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <NavigationContainer>
        <RootNavigator />
      </NavigationContainer>
    </QueryClientProvider>
  )
}
```

---

## Patrón base con React Query: `useProducts`

```typescript
// src/hooks/useProducts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { getEntity, addEntity, updateEntity, deleteEntity } from '@/services/api'
import type { Product } from '@/types'

// Clave única de cache para esta entidad
const QUERY_KEY = ['products']

export function useProducts() {
  const queryClient = useQueryClient()

  // GET /products — cacheado, refetch automático al volver a la pantalla
  const { data: products = [], isLoading, isError, error, refetch } = useQuery<Product[]>({
    queryKey: QUERY_KEY,
    queryFn: () => getEntity<Product[]>('products'),
  })

  // POST /products — invalida cache al terminar
  const addProduct = useMutation({
    mutationFn: (payload: Partial<Product>) => addEntity<Product>('products', payload),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: QUERY_KEY }),
  })

  // PUT /products/{id} — invalida cache al terminar
  const updateProduct = useMutation({
    mutationFn: ({ id, payload }: { id: number; payload: Partial<Product> }) =>
      updateEntity<Product>('products', id, payload),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: QUERY_KEY }),
  })

  // DELETE /products/{id}
  const removeProduct = useMutation({
    mutationFn: (id: number) => deleteEntity('products', id),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: QUERY_KEY }),
  })

  return {
    products,
    isLoading,
    isError,
    error,
    refetch,
    addProduct: addProduct.mutateAsync,
    updateProduct: updateProduct.mutateAsync,
    removeProduct: removeProduct.mutateAsync,
  }
}
```

Uso en screen:
```typescript
const { products, isLoading, addProduct } = useProducts()

// Crear producto — invalida cache y refresca la lista automáticamente
await addProduct({ name: 'Nuevo', price: 99 })
```

### Patrón con sub-entidades (rutas anidadas de Laravel)

```typescript
// src/hooks/useTrainerExercises.ts
import { useQuery } from '@tanstack/react-query'
import { getSubEntity } from '@/services/api'
import type { Exercise } from '@/types'

export function useTrainerExercises(trainerId: number) {
  return useQuery<Exercise[]>({
    queryKey: ['trainers', trainerId, 'exercises'],
    queryFn: () => getSubEntity<Exercise[]>('trainers', trainerId, 'exercises'),
    enabled: !!trainerId, // no ejecuta si trainerId es null/undefined
  })
}
```

---

## `useAuth` — Autenticación y estado de sesión

La parte HTTP vive en `services/auth.ts`. El estado reactivo de sesión vive en `store/authStore.ts` (Zustand). Este hook los conecta y expone una API limpia a las screens.

```typescript
// src/hooks/useAuth.ts
import { useCallback } from 'react'
import { useQueryClient } from '@tanstack/react-query'
import { useAuthStore } from '@/store/authStore'
import { login as loginService, logout as logoutService, loginGoogle as loginGoogleService, register as registerService } from '@/services/auth'
import type { LoginPayload, RegisterPayload } from '@/services/auth'

export function useAuth() {
  const queryClient = useQueryClient()
  const { token, user, isGuest, setAuth, setAsGuest, clearAuth } = useAuthStore()

  const login = useCallback(async (payload: LoginPayload) => {
    const response = await loginService(payload)
    setAuth(response.token, response.user)
  }, [setAuth])

  const loginGoogle = useCallback(async (googlePayload: unknown) => {
    const response = await loginGoogleService(googlePayload)
    setAuth(response.token, response.user)
  }, [setAuth])

  const register = useCallback(async (payload: RegisterPayload) => {
    const response = await registerService(payload)
    setAuth(response.token, response.user)
  }, [setAuth])

  const logout = useCallback(async () => {
    await logoutService()
    clearAuth()
    queryClient.clear() // Limpia toda la cache de React Query al cerrar sesión
  }, [clearAuth, queryClient])

  const loginAsGuest = useCallback(() => {
    setAsGuest()
  }, [setAsGuest])

  return {
    token,
    user,
    isGuest,
    isAuthenticated: !!token || isGuest,  // true si autenticado O invitado
    login,
    loginGoogle,
    register,
    loginAsGuest,
    logout,
  }
}
```

---

## `useFetch` — Hook genérico para llamadas puntuales

Para casos donde **no necesitas React Query** (llamadas únicas, sin cache):

```typescript
// src/hooks/useFetch.ts
import { useState, useEffect } from 'react'

export function useFetch<T>(fn: () => Promise<T>, deps: unknown[] = []) {
  const [data, setData] = useState<T | null>(null)
  const [isLoading, setIsLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    let mounted = true
    setIsLoading(true)
    fn()
      .then(res => { if (mounted) setData(res) })
      .catch(err => { if (mounted) setError((err as { message?: string })?.message ?? 'Error') })
      .finally(() => { if (mounted) setIsLoading(false) })
    return () => { mounted = false }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, deps)

  return { data, isLoading, error }
}
```

> Prefiere `useQuery` de React Query sobre `useFetch` cuando el dato se usa en múltiples pantallas o necesita cache.

---

## Patrón de formulario genérico: `useForm`

```typescript
// src/hooks/useFormState.ts
import { useState, useCallback } from 'react'

type UseFormProps<T> = {
  initialValues: T
  onSubmit: (values: T) => Promise<void>
  validate?: (values: T) => Record<string, string>
}

export function useForm<T extends Record<string, unknown>>({
  initialValues,
  onSubmit,
  validate,
}: UseFormProps<T>) {
  const [values, setValues] = useState<T>(initialValues)
  const [errors, setErrors] = useState<Record<string, string>>({})
  const [loading, setLoading] = useState(false)

  const handleChange = useCallback((field: keyof T, value: unknown) => {
    setValues((prev) => ({ ...prev, [field]: value }))
  }, [])

  const handleSubmit = useCallback(async () => {
    if (validate) {
      const validationErrors = validate(values)
      if (Object.keys(validationErrors).length > 0) {
        setErrors(validationErrors)
        return
      }
    }
    setLoading(true)
    try {
      await onSubmit(values)
    } finally {
      setLoading(false)
    }
  }, [values, validate, onSubmit])

  return { values, errors, loading, handleChange, handleSubmit }
}
```

---

## Uso en una Screen (patrón completo)

```tsx
// src/screens/Login/index.tsx
import { View, TextInput } from 'react-native'
import { Button } from '@/components/Button'
import { useAuth } from '@/hooks/useAuth'
import { useForm } from '@/hooks/useFormState'
import { isValidEmail } from '@/utils/validators'

export function Login() {
  const { login } = useAuth()

  const { values, errors, loading, handleChange, handleSubmit } = useForm({
    initialValues: { email: '', password: '' },
    onSubmit: async (vals) => { await login(vals) },
    validate: (vals) => {
      const errs: Record<string, string> = {}
      if (!isValidEmail(vals.email)) errs.email = 'Email inválido'
      if (vals.password.length < 8) errs.password = 'Mínimo 8 caracteres'
      return errs
    },
  })

  return (
    <View className="flex-1 p-6">
      <TextInput
        className="border border-gray-300 rounded-lg p-3 mb-4"
        value={values.email}
        onChangeText={(v) => handleChange('email', v)}
        placeholder="Email"
      />
      <Button
        label={loading ? 'Cargando...' : 'Iniciar sesión'}
        onPress={handleSubmit}
      />
    </View>
  )
}
```

---

## Gestión de Estado Global

### Zustand para auth (`authStore`) — OBLIGATORIO

El `authStore` gestiona el estado de sesión de forma reactiva en toda la app: token, usuario autenticado y comprobación inicial al arrancar.

```typescript
// src/store/authStore.ts
import { create } from 'zustand'
import { getToken } from '@/services/storage'

type AuthUser = { id: number; name: string; email: string; role_id: number }

type AuthStore = {
  token: string | null
  user: AuthUser | null
  isGuest: boolean
  isLoaded: boolean         // true cuando ya se comprobó el token guardado al arrancar
  setAuth: (token: string, user: AuthUser) => void
  setAsGuest: () => void
  clearAuth: () => void
  loadToken: () => Promise<void>  // lee el token persistido y actualiza el estado
}

export const useAuthStore = create<AuthStore>((set) => ({
  token: null,
  user: null,
  isGuest: false,
  isLoaded: false,

  setAuth: (token, user) => set({ token, user, isGuest: false }),

  setAsGuest: () => set({ isGuest: true, token: null, user: null }),

  clearAuth: () => set({ token: null, user: null, isGuest: false }),

  // Lee el token persistido al arrancar la app
  loadToken: async () => {
    const token = await getToken()
    set({ token: token ?? null, isLoaded: true })
  },
}))
```

Uso en `App.tsx` para cargar el token al arrancar:
```typescript
// src/App.tsx
import { useEffect } from 'react'
import { useAuthStore } from '@/store/authStore'

function AppBootstrap() {
  const loadToken = useAuthStore((s) => s.loadToken)
  useEffect(() => { loadToken() }, [loadToken])
  return null
}
```

### React Query vs Zustand — cuándo usar cada uno

| Necesidad | Herramienta |
|---|---|
| Datos del servidor (lista de productos, usuarios...) | React Query |
| Estado de sesión (token, usuario logueado) | Zustand |
| Estado local de UI (modal abierto, filtro seleccionado) | `useState` local |
| Datos del servidor que también necesitan persistencia global | React Query + Zustand |

> ❌ **No usar Redux Toolkit.** Zustand + React Query cubre todos los casos de este stack. Redux añade boilerplate sin beneficio real cuando React Query ya gestiona el estado del servidor. Si en el futuro aparece un caso que genuinamente no se puede resolver con esta combinación, consultar con el equipo antes de introducir RTK.

---

## Testing de Hooks

```typescript
// tests/unit/useProducts.test.ts
import { renderHook, waitFor } from '@testing-library/react-native'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { useProducts } from '@/hooks/useProducts'
import * as api from '@/services/api'

jest.mock('@/services/api')

const wrapper = ({ children }: { children: React.ReactNode }) => (
  <QueryClientProvider client={new QueryClient()}>{children}</QueryClientProvider>
)

test('carga productos correctamente', async () => {
  const mockProducts = [{ id: 1, name: 'Test', price: 10 }]
  jest.spyOn(api, 'getEntity').mockResolvedValue(mockProducts)

  const { result } = renderHook(() => useProducts(), { wrapper })

  await waitFor(() => expect(result.current.isLoading).toBe(false))
  expect(result.current.products).toEqual(mockProducts)
})
```

---

## Guest Mode — Acceso sin autenticación

El template soporta un modo "invitado" que permite acceder a las tabs principales (`Home`, `Profile`, etc) sin necesidad de registrarse o loguearse.

### Cómo funciona

1. En la pantalla `Login`, hay un botón "Entrar como invitado" que llama a `useAuth().loginAsGuest()`
2. `loginAsGuest()` establece `isGuest = true` en el store (sin token ni user)
3. `RootNavigator` entra a `AppStack` si `token || isGuest` (no solo si `token`)
4. `isAuthenticated` devuelve `true` si hay token O si `isGuest === true`
5. Las screens pueden checar `isGuest` para adaptar su contenido:
   - `Profile` muestra UI diferente para invitados (sin datos de usuario)
   - Acciones que requieran token fallarán con 401 (los servicios deben manejar esto)

### Uso en una screen

```typescript
import { useAuth } from '@/hooks/useAuth'

export function Profile() {
  const { isGuest, user, logout } = useAuth()

  if (isGuest) {
    return (
      <View>
        <Text>Eres un invitado</Text>
        <Button label="Cerrar sesión" onPress={logout} />
      </View>
    )
  }

  return (
    <View>
      <Text>{user?.name}</Text>
    </View>
  )
}
```

### Notas importantes

- Guest mode **no persiste** en storage — si cierras la app, vuelves a Welcome
- Guest puede hacer `logout()` y vuelve a la pantalla de Login
- Si un guest intenta hacer una acción que requiere token (API call), la API devolverá 401 y se manejará en el hook/servicio
- Guest es útil para demos, previews o permitir exploración sin registro forzado

