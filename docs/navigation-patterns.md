---
title: Navigation patterns
version: 1.3
---

# Navegación tipada con React Navigation

La navegación sigue un sistema de tipado centralizado para eliminar errores en tiempo de desarrollo y obtener autocompletado completo en pantallas y hooks.

---

## Estructura de la carpeta `navigation/`

```
navigation/
├── navigation-types.ts   ← 🔑 ÚNICO lugar donde se definen rutas y tipos
├── hooks.ts              ← Hooks de navegación tipados
├── RootNavigator.tsx     ← Entry point principal (decide Auth vs App)
├── linking.ts            ← Deep linking config (opcional)
├── index.ts              ← Re-exporta todo públicamente
└── stacks/
    ├── AuthStack.tsx     ← Welcome → Login ↔ Register
    ├── AppStack.tsx      ← NativeStack que envuelve AppTabs + pantallas full-screen
    └── AppTabs.tsx       ← BottomTabNavigator: Home | Profile
```

---

## `navigation-types.ts` — Fuente única de verdad

```typescript
// src/navigation/navigation-types.ts
import type { NativeStackScreenProps } from '@react-navigation/native-stack'
import type { BottomTabScreenProps } from '@react-navigation/bottom-tabs'
import type { NavigatorScreenParams } from '@react-navigation/native'

// — Root: decide entre Auth y App —
export type RootStackParamList = {
  Auth: undefined
  App: NavigatorScreenParams<AppStackParamList>
}

// — Auth (usuario no autenticado) —
export type AuthStackParamList = {
  Welcome: undefined
  Login: undefined
  Register: undefined
}

// — App: stack principal autenticado (envuelve los tabs + pantallas full-screen) —
export type AppStackParamList = {
  MainTabs: NavigatorScreenParams<AppTabsParamList>
  // Pantallas full-screen fuera de tabs:
  // ProductDetail: { productId: string }
  // Settings: undefined
  // EditProfile: undefined
}

// — Tabs principales (usuario autenticado) —
export type AppTabsParamList = {
  Home: undefined
  Profile: undefined
}

// Props tipadas para componentes de pantalla
export type RootScreenProps<T extends keyof RootStackParamList> =
  NativeStackScreenProps<RootStackParamList, T>

export type AuthScreenProps<T extends keyof AuthStackParamList> =
  NativeStackScreenProps<AuthStackParamList, T>

export type AppStackScreenProps<T extends keyof AppStackParamList> =
  NativeStackScreenProps<AppStackParamList, T>

export type AppTabsScreenProps<T extends keyof AppTabsParamList> =
  BottomTabScreenProps<AppTabsParamList, T>

// Declaración global para autocompletado en useNavigation sin genérico
declare global {
  namespace ReactNavigation {
    interface RootParamList extends RootStackParamList {}
  }
}
```

### Jerarquía de navegación

El sistema tiene 4 niveles. `AppStack` es la capa clave que permite tener pantallas autenticadas **fuera** del tab bar:

```
RootNavigator (NativeStack)
├── Auth → AuthStack (NativeStack)
│   ├── Welcome
│   ├── Login
│   └── Register
└── App → AppStack (NativeStack)        ← pantallas full-screen fuera de tabs
    ├── MainTabs → AppTabs (BottomTab)
    │   ├── Home
    │   └── Profile
    ├── ProductDetail                    ← se pushea encima de los tabs
    ├── Settings
    └── EditProfile
```

### ¿Dónde va cada pantalla nueva?

| ¿La pantalla necesita…? | Añadir en… | Ejemplo |
|---|---|---|
| Tab en la barra inferior | `AppTabsParamList` + `AppTabs.tsx` | Home, Profile, Orders |
| Full-screen encima de tabs (sin tab bar visible) | `AppStackParamList` + `AppStack.tsx` | ProductDetail, Settings, EditProfile |
| Sub-pantalla dentro de un tab (tab bar visible) | Stack anidado dentro del tab | Home → ProductDetail (ver sección [Anidar stacks dentro de un tab](#anidar-stacks-dentro-de-un-tab)) |
| Pantalla sin autenticación | `AuthStackParamList` + `AuthStack.tsx` | Welcome, Login, ForgotPassword |

> **Nota:** Si quieres que una pantalla como `ProductDetail` **mantenga** el tab bar visible, anídala dentro de un stack local del tab (ver más abajo). Si quieres que el tab bar **desaparezca**, ponla en `AppStack`.

---

## `hooks.ts` — Hooks tipados de navegación

```typescript
// src/navigation/hooks.ts
import { useNavigation } from '@react-navigation/native'
import type { NativeStackNavigationProp } from '@react-navigation/native-stack'
import type { RootStackParamList, AuthStackParamList, AppStackParamList } from './navigation-types'

export function useRootNavigation() {
  return useNavigation<NativeStackNavigationProp<RootStackParamList>>()
}

export function useAuthNavigation() {
  return useNavigation<NativeStackNavigationProp<AuthStackParamList>>()
}

export function useAppNavigation() {
  return useNavigation<NativeStackNavigationProp<AppStackParamList>>()
}
```

> **`useAppNavigation()`** es el hook que usarás en cualquier pantalla autenticada para navegar a pantallas full-screen fuera de tabs (ej: `navigation.navigate('ProductDetail', { productId: '123' })`).

---

## `RootNavigator.tsx` — Entry point con lógica auth

```tsx
// src/navigation/RootNavigator.tsx
import React from 'react'
import { createNativeStackNavigator } from '@react-navigation/native-stack'
import type { RootStackParamList } from './navigation-types'
import { useAuthStore } from '@/store/authStore'
import { AuthStack } from './stacks/AuthStack'
import { AppStack } from './stacks/AppStack'

const Stack = createNativeStackNavigator<RootStackParamList>()

export function RootNavigator(): React.JSX.Element | null {
  const { token, isGuest, isLoaded } = useAuthStore()

  if (!isLoaded) return null

  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      {token || isGuest ? (
        <Stack.Screen name="App" component={AppStack} />
      ) : (
        <Stack.Screen name="Auth" component={AuthStack} />
      )}
    </Stack.Navigator>
  )
}
```

> **Importante:** `RootNavigator` apunta a `AppStack` (no a `AppTabs`). `AppStack` se encarga de montar los tabs y las pantallas full-screen.
> La condición `token || isGuest` permite que los usuarios en modo invitado accedan a la app sin autenticación completa.

---

## `stacks/AuthStack.tsx` — Stack de autenticación

```tsx
// src/navigation/stacks/AuthStack.tsx
import React from 'react'
import { createNativeStackNavigator } from '@react-navigation/native-stack'
import type { AuthStackParamList } from '../navigation-types'
import { Welcome } from '@/screens/Welcome'
import { Login } from '@/screens/Login'
import { Register } from '@/screens/Register'

const Stack = createNativeStackNavigator<AuthStackParamList>()

export function AuthStack() {
  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      <Stack.Screen name="Welcome" component={Welcome} />
      <Stack.Screen name="Login" component={Login} />
      <Stack.Screen name="Register" component={Register} />
    </Stack.Navigator>
  )
}
```

---

## `stacks/AppStack.tsx` — Stack principal autenticado

Este es el nivel intermedio entre `RootNavigator` y `AppTabs`. Contiene los tabs como primera screen y todas las pantallas full-screen que **no** necesitan tab bar:

```tsx
// src/navigation/stacks/AppStack.tsx
import React from 'react'
import { createNativeStackNavigator } from '@react-navigation/native-stack'
import type { AppStackParamList } from '../navigation-types'
import { AppTabs } from './AppTabs'
// Importar pantallas full-screen aquí:
// import { ProductDetail } from '@/screens/ProductDetail'
// import { Settings } from '@/screens/Settings'
// import { EditProfile } from '@/screens/EditProfile'

const Stack = createNativeStackNavigator<AppStackParamList>()

export function AppStack() {
  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      <Stack.Screen name="MainTabs" component={AppTabs} />
      {/* Pantallas full-screen (sin tab bar visible): */}
      {/* <Stack.Screen name="ProductDetail" component={ProductDetail} /> */}
      {/* <Stack.Screen name="Settings" component={Settings} /> */}
      {/* <Stack.Screen name="EditProfile" component={EditProfile} /> */}
    </Stack.Navigator>
  )
}
```

### Añadir una pantalla fuera de tabs

Para añadir una pantalla full-screen (sin tab bar):

1. Añade la ruta en `AppStackParamList` dentro de `navigation-types.ts`
2. Registra el `<Stack.Screen>` en `AppStack.tsx`
3. Navega desde cualquier pantalla autenticada con `useAppNavigation()`

```typescript
// 1. navigation-types.ts
export type AppStackParamList = {
  MainTabs: NavigatorScreenParams<AppTabsParamList>
  ProductDetail: { productId: string }  // ← nueva pantalla
}

// 2. AppStack.tsx
<Stack.Screen name="ProductDetail" component={ProductDetail} />

// 3. Desde cualquier pantalla (ej: Home)
import { useAppNavigation } from '@/navigation'

const navigation = useAppNavigation()
navigation.navigate('ProductDetail', { productId: '123' })
```

---

## `stacks/AppTabs.tsx` — Bottom Tab Navigator

```tsx
// src/navigation/stacks/AppTabs.tsx
import React from 'react'
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs'
import { Ionicons } from '@expo/vector-icons'
import type { AppTabsParamList } from '../navigation-types'
import { Home } from '@/screens/Home'
import { Profile } from '@/screens/Profile'

const Tab = createBottomTabNavigator<AppTabsParamList>()

export function AppTabs() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ color, size }) => {
          const icons: Record<keyof AppTabsParamList, keyof typeof Ionicons.glyphMap> = {
            Home: 'home-outline',
            Profile: 'person-outline',
          }
          return <Ionicons name={icons[route.name]} size={size} color={color} />
        },
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: 'gray',
      })}
    >
      <Tab.Screen name="Home" component={Home} />
      <Tab.Screen name="Profile" component={Profile} />
    </Tab.Navigator>
  )
}
```

### Añadir un nuevo tab

Para añadir un tab, actualiza `AppTabsParamList` en `navigation-types.ts` y registra la pantalla en `AppTabs.tsx`:

```typescript
// navigation-types.ts
export type AppTabsParamList = {
  Home: undefined
  Profile: undefined
  Orders: undefined  // ← nuevo tab
}
```

```tsx
// AppTabs.tsx — añadir dentro de <Tab.Navigator>
<Tab.Screen name="Orders" component={Orders} />
```

### Anidar stacks dentro de un tab

Si un tab necesita sub-pantallas **manteniendo el tab bar visible** (ej: Home → ProductDetail sin ocultar los tabs), crea un stack anidado dentro del tab:

```tsx
// src/navigation/stacks/HomeStack.tsx
import React from 'react'
import { createNativeStackNavigator } from '@react-navigation/native-stack'
import { Home } from '@/screens/Home'
import { ProductDetail } from '@/screens/ProductDetail'

// Definir estas rutas en navigation-types.ts
type HomeStackParamList = {
  HomeMain: undefined
  ProductDetail: { productId: string }
}

const Stack = createNativeStackNavigator<HomeStackParamList>()

export function HomeStack() {
  return (
    <Stack.Navigator>
      <Stack.Screen name="HomeMain" component={Home} />
      <Stack.Screen name="ProductDetail" component={ProductDetail} />
    </Stack.Navigator>
  )
}
```

Luego usa `HomeStack` como componente del tab `Home` en `AppTabs.tsx`.

> **¿Stack en tab o en AppStack?** Si quieres que el tab bar **siga visible** → stack local en el tab. Si quieres pantalla **full-screen sin tabs** → `AppStack`.

---

## `index.ts` — Re-exportar todo

```typescript
// src/navigation/index.ts
export { RootNavigator } from './RootNavigator'
export * from './navigation-types'
export * from './hooks'
```

---

## Uso en pantallas

```tsx
// Opción 1: pantalla dentro de tabs (con props tipadas)
import type { AppTabsScreenProps } from '@/navigation'

export function Profile({ navigation }: AppTabsScreenProps<'Profile'>) {
  return (/* ... */)
}

// Opción 2: pantalla dentro de AuthStack (con hook)
import { useAuthNavigation } from '@/navigation'

export function Welcome() {
  const navigation = useAuthNavigation()
  const goToLogin = () => navigation.navigate('Login') // ✅ autocompletado
  return (/* ... */)
}

// Opción 3: navegar a pantalla full-screen fuera de tabs (desde cualquier screen autenticada)
import { useAppNavigation } from '@/navigation'

export function Home() {
  const navigation = useAppNavigation()
  const goToDetail = (id: string) =>
    navigation.navigate('ProductDetail', { productId: id }) // ✅ tipado
  return (/* ... */)
}

// Opción 4: pantalla full-screen (con props tipadas)
import type { AppStackScreenProps } from '@/navigation'

export function ProductDetail({ route }: AppStackScreenProps<'ProductDetail'>) {
  const { productId } = route.params // ✅ tipado
  return (/* ... */)
}
```

---

## Importar siempre con path alias

```typescript
// ✅ Correcto
import type { AppTabsScreenProps } from '@/navigation'
import type { AppStackScreenProps } from '@/navigation'
import { useAppNavigation } from '@/navigation'

// ❌ Prohibido
import type { AppTabsScreenProps } from '../../../navigation/types'
```

---

## Anti-patrones de Navegación

```typescript
// ❌ 1. Definir tipos en múltiples archivos
// screens/Home/types.ts → type HomeRoutes = { Home: undefined }
// ✅ Un único navigation-types.ts

// ❌ 2. Navegar sin tipado
const navigation = useNavigation() // any
navigation.navigate('Profile', { wrongParam: true }) // sin error TS

// ✅ Hook tipado
const navigation = useRootNavigation()
navigation.navigate('Profile', { userId: '123' }) // error si params incorrectos

// ❌ 3. Duplicar rutas
// En AuthStack.tsx: const routes = { Login: undefined }
// En types.ts:     Login: undefined    ← duplicado

// ❌ 4. Imports relativos en pantallas
import { useRootNavigation } from '../../../navigation/hooks'
// ✅ import { useRootNavigation } from '@/navigation'
```

---

## Beneficios de esta estructura

| Aspecto | Sin tipado | Con tipado |
|---|---|---|
| Errores de ruta | En runtime | En compilación |
| Autocompletado | ❌ Ninguno | ✅ Completo |
| Parámetros olvidados | ❌ Posible | ✅ Imposible |
| Imports repetidos | ❌ En cada screen | ✅ Vía `@/navigation` |
| Mantener rutas | ❌ Tedioso | ✅ Un archivo |

