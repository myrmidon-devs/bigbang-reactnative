---
title: Services & API
version: 2.0
---

# Services y API

Los `services/` contienen toda la lógica de comunicación HTTP y almacenamiento. Son funciones TypeScript puras, sin dependencias de React.

Este proyecto usa un patrón de API genérica con métodos reutilizables para cualquier entidad del backend. Un único cliente Axios centralizado cubre todas las operaciones CRUD, rutas anidadas y subidas de archivos sin duplicar código por recurso.

---

## Estructura

```
services/
├── api.ts           ← Cliente Axios + métodos genéricos CRUD + sub-entidades
├── auth.ts          ← Login, logout, registro y social auth
├── storage.ts       ← Token seguro con expo-secure-store
└── logger.ts        ← Logging (reemplaza console.log)
```

---

## `api.ts` — Cliente Axios centralizado con métodos genéricos

Métodos genéricos CRUD que funcionan con cualquier ruta REST de Laravel (`/entity`, `/entity/{id}`, `/entity/{id}/subEntity/{id}`). No se duplica código por cada recurso del backend.

### Cliente base e interceptores

```typescript
// src/services/api.ts
import axios, { AxiosError } from 'axios'
import { getToken, removeToken } from './storage'
import { logger } from './logger'

export const api = axios.create({
  baseURL: process.env.EXPO_PUBLIC_API_URL,
  timeout: 10000,
  headers: { 'Content-Type': 'application/json' },
})

// Inyecta el Bearer token automáticamente en cada request
api.interceptors.request.use(async (config) => {
  const token = await getToken()
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// Captura 401 globalmente y limpia la sesión
// Para conectar con authStore, usa useAuthStore.getState().clearAuth()
api.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    if (error.response?.status === 401) {
      await removeToken()
      // useAuthStore.getState().clearAuth() ← se conecta desde authStore.ts
      logger.warn('Sesión expirada, token eliminado')
    }
    return Promise.reject(error)
  }
)

// Tipo de error normalizado
export type ApiError = {
  message: string
  status?: number
}

export const parseApiError = (err: unknown): ApiError => {
  if (axios.isAxiosError(err)) {
    const e = err as AxiosError<{ message?: string }>
    return {
      message: e.response?.data?.message ?? e.message,
      status: e.response?.status,
    }
  }
  return { message: String(err) }
}
```

---

### Métodos GET

```typescript
// Obtener lista o entidad por id
export const getEntity = async <T>(entity: string, id?: number, params?: Record<string, unknown>): Promise<T> => {
  try {
    const url = id ? `/${entity}/${id}` : `/${entity}`
    const { data } = await api.get<T>(url, { params })
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// GET con query params explícitos: /entity?key=value
export const getEntityWithParams = async <T>(entity: string, params: Record<string, unknown>): Promise<T> => {
  try {
    const { data } = await api.get<T>(`/${entity}`, { params })
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// GET sub-ruta anidada: /entity/{id}/subEntity o /entity/{id}/subEntity/{subId}
// Usado con rutas anidadas de Laravel (Route::apiResource dentro de Route::apiResource)
export const getSubEntity = async <T>(
  entity: string,
  idEntity: number,
  subEntity: string,
  idSubEntity?: number
): Promise<T> => {
  try {
    const url = idSubEntity
      ? `/${entity}/${idEntity}/${subEntity}/${idSubEntity}`
      : `/${entity}/${idEntity}/${subEntity}`
    const { data } = await api.get<T>(url)
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// GET sub-ruta de tercer nivel: /entity/{id}/subEntity/{subId}/subSubEntity
export const getSubSubEntity = async <T>(
  entity: string,
  idEntity: number,
  subEntity: string,
  idSubEntity: number,
  subSubEntity: string,
  idSubSubEntity?: number
): Promise<T> => {
  try {
    const url = idSubSubEntity
      ? `/${entity}/${idEntity}/${subEntity}/${idSubEntity}/${subSubEntity}/${idSubSubEntity}`
      : `/${entity}/${idEntity}/${subEntity}/${idSubEntity}/${subSubEntity}`
    const { data } = await api.get<T>(url)
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}
```

---

### Métodos POST (crear)

```typescript
// POST /entity
export const addEntity = async <T>(entity: string, params: unknown): Promise<T> => {
  try {
    const { data } = await api.post<T>(`/${entity}`, params)
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// POST /entity/{id}/subEntity
export const addSubEntity = async <T>(
  entity: string,
  idEntity: number,
  subEntity: string,
  params?: unknown
): Promise<T> => {
  try {
    const { data } = await api.post<T>(`/${entity}/${idEntity}/${subEntity}`, params)
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// POST /entity/{id}/subEntity/{subId}/subSubEntity
export const addSubSubEntity = async <T>(
  entity: string,
  idEntity: number,
  subEntity: string,
  idSubEntity: number,
  subSubEntity: string,
  params?: unknown
): Promise<T> => {
  try {
    const { data } = await api.post<T>(
      `/${entity}/${idEntity}/${subEntity}/${idSubEntity}/${subSubEntity}`,
      params
    )
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}
```

---

### Métodos PUT (actualizar)

```typescript
// PUT /entity/{id}
export const updateEntity = async <T>(entity: string, id: number, params: unknown): Promise<T> => {
  try {
    const { data } = await api.put<T>(`/${entity}/${id}`, params)
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// PUT /entity/{id}/subEntity/{subId}
export const updateSubEntity = async <T>(
  entity: string,
  idEntity: number,
  subEntity: string,
  idSubEntity: number,
  params: unknown
): Promise<T> => {
  try {
    const { data } = await api.put<T>(`/${entity}/${idEntity}/${subEntity}/${idSubEntity}`, params)
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// PUT /entity/{id}/subEntity/{subId}/subSubEntity
export const updateSubSubEntity = async <T>(
  entity: string,
  idEntity: number,
  subEntity: string,
  idSubEntity: number,
  subSubEntity: string,
  params: unknown
): Promise<T> => {
  try {
    const { data } = await api.put<T>(
      `/${entity}/${idEntity}/${subEntity}/${idSubEntity}/${subSubEntity}`,
      params
    )
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}
```

---

### Métodos DELETE

```typescript
// DELETE /entity/{id}
export const deleteEntity = async (entity: string, id: number): Promise<void> => {
  try {
    await api.delete(`/${entity}/${id}`)
  } catch (err) {
    throw parseApiError(err)
  }
}

// DELETE /entity/{id}/subEntity/{subId}
export const deleteSubEntity = async (
  entity: string,
  idEntity: number,
  subEntity: string,
  idSubEntity: number
): Promise<void> => {
  try {
    await api.delete(`/${entity}/${idEntity}/${subEntity}/${idSubEntity}`)
  } catch (err) {
    throw parseApiError(err)
  }
}

// DELETE /entity/{id}/subEntity/{subId}/subSubEntity/{subSubId}
export const deleteSubSubEntity = async (
  entity: string,
  idEntity: number,
  subEntity: string,
  idSubEntity: number,
  subSubEntity: string,
  idSubSubEntity: number
): Promise<void> => {
  try {
    await api.delete(
      `/${entity}/${idEntity}/${subEntity}/${idSubEntity}/${subSubEntity}/${idSubSubEntity}`
    )
  } catch (err) {
    throw parseApiError(err)
  }
}
```

---

### Upload de archivos con progreso

Usa `onUploadProgress` de Axios para reportar porcentaje de subida. Se usa `FormData` nativo de React Native para enviar archivos `multipart/form-data`.

```typescript
export type UploadProgress = { status: 'progress'; percent: number } | { status: 'ok'; body: unknown }

// POST /entity (multipart/form-data con progreso)
export const uploadEntityFiles = (
  entity: string,
  formData: FormData,
  onProgress?: (percent: number) => void
): Promise<unknown> => {
  return new Promise((resolve, reject) => {
    api.post(`/${entity}`, formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
      onUploadProgress: (event) => {
        if (event.total) {
          const percent = Math.round((event.loaded * 100) / event.total)
          onProgress?.(percent)
        }
      },
    })
      .then((res) => resolve(res.data))
      .catch((err) => reject(parseApiError(err)))
  })
}

// POST /entity/{id} (actualizar archivo existente)
export const updateEntityFiles = (
  entity: string,
  id: number,
  formData: FormData,
  onProgress?: (percent: number) => void
): Promise<unknown> => {
  return new Promise((resolve, reject) => {
    api.post(`/${entity}/${id}`, formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
      onUploadProgress: (event) => {
        if (event.total) {
          const percent = Math.round((event.loaded * 100) / event.total)
          onProgress?.(percent)
        }
      },
    })
      .then((res) => resolve(res.data))
      .catch((err) => reject(parseApiError(err)))
  })
}
```

Uso en un hook:
```typescript
// Ejemplo de uso del upload con progreso
const [progress, setProgress] = useState(0)

await uploadEntityFiles('trainers/1/images', formData, (percent) => {
  setProgress(percent) // actualiza barra de progreso
})
```

---

## `storage.ts` — Token seguro

Usa `expo-secure-store` para guardar el token de forma **encriptada** en el dispositivo.

```typescript
// src/services/storage.ts
import * as SecureStore from 'expo-secure-store'

const TOKEN_KEY = 'auth-token'

export const getToken = async (): Promise<string | null> => {
  return SecureStore.getItemAsync(TOKEN_KEY)
}

export const setToken = async (token: string): Promise<void> => {
  await SecureStore.setItemAsync(TOKEN_KEY, token)
}

export const removeToken = async (): Promise<void> => {
  await SecureStore.deleteItemAsync(TOKEN_KEY)
}
```

> **Por qué `expo-secure-store` y no `AsyncStorage`**: AsyncStorage guarda datos en texto plano. SecureStore los encripta usando el keychain del sistema operativo (Keychain en iOS, Keystore en Android). Nunca guardes tokens con AsyncStorage.

---

## `auth.ts` — Autenticación

Contiene las llamadas HTTP de auth: login, logout, registro y social auth. La gestión del estado reactivo de sesión vive en `store/authStore.ts`.

```typescript
// src/services/auth.ts
import { api, parseApiError } from './api'
import { setToken, removeToken } from './storage'

export type LoginPayload = { email: string; password: string }
export type RegisterPayload = { name: string; email: string; password: string }
export type AuthResponse = { token: string; user: { id: number; name: string; email: string; role_id: number } }

// Login básico con email y contraseña
export const login = async (payload: LoginPayload): Promise<AuthResponse> => {
  try {
    const { data } = await api.post<AuthResponse>('/login', payload)
    await setToken(data.token)
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// Login con Google (datos devueltos por expo-auth-session o similar)
export const loginGoogle = async (googlePayload: unknown): Promise<AuthResponse> => {
  try {
    const { data } = await api.post<AuthResponse>('/login-google', googlePayload)
    await setToken(data.token)
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// Login con Facebook
export const loginFacebook = async (fbPayload: unknown): Promise<AuthResponse> => {
  try {
    const { data } = await api.post<AuthResponse>('/login-facebook', fbPayload)
    await setToken(data.token)
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// Recuperar contraseña
export const forgotPassword = async (email: string): Promise<void> => {
  try {
    await api.post('/forgot-password', { email })
  } catch (err) {
    throw parseApiError(err)
  }
}

// Registro de nuevo usuario
export const register = async (payload: RegisterPayload): Promise<AuthResponse> => {
  try {
    const { data } = await api.post<AuthResponse>('/register', payload)
    await setToken(data.token)
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}

// Logout — elimina el token del dispositivo y llama al backend
export const logout = async (): Promise<void> => {
  try {
    await api.post('/logout') // invalida el token en Laravel (Sanctum)
  } catch {
    // Ignoramos error de red en logout — el token local se limpia igual
  } finally {
    await removeToken()
  }
}

// Obtener datos del usuario autenticado
export const getMe = async <T>(): Promise<T> => {
  try {
    const { data } = await api.get<T>('/user')
    return data
  } catch (err) {
    throw parseApiError(err)
  }
}
```

---

## `logger.ts` — Sin `console.log`

```typescript
// src/services/logger.ts
const isDev = process.env.NODE_ENV !== 'production'

export const logger = {
  info: (...args: unknown[]) => { if (isDev) console.info('[INFO]', ...args) },
  warn: (...args: unknown[]) => { if (isDev) console.warn('[WARN]', ...args) },
  error: (...args: unknown[]) => console.error('[ERROR]', ...args),
}
```

---

## Reglas de Services

- ✅ Funciones puras, sin dependencias de React
- ✅ Error handling en cada función con `parseApiError`
- ✅ Tipos de retorno explícitos (`Promise<T>`)
- ✅ Token inyectado automáticamente vía interceptor
- ✅ URLs construidas de forma genérica (no hardcodeadas por entidad)
- ❌ No importar componentes ni hooks
- ❌ No usar `useState`, `useEffect` ni nada de React
- ❌ No usar `console.log` — usar `logger`
- ❌ No hardcodear URLs (usar `process.env.EXPO_PUBLIC_*`)

