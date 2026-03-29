---
title: Plantillas y snippets
version: 1.5
---

# Plantillas y snippets

Esqueletos de archivos listos para copiar y adaptar. El agente debe usar estas plantillas como punto de partida para nuevos archivos.

---

## Componente reutilizable — `Button`

```tsx
// src/components/Button/Button.tsx
import React from 'react'
import { TouchableOpacity, Text } from 'react-native'

type Variant = 'primary' | 'secondary' | 'danger'

type ButtonProps = {
  label: string
  onPress?: () => void
  variant?: Variant
  disabled?: boolean
  className?: string
}

export function Button({ label, onPress, variant = 'primary', disabled = false, className = '' }: ButtonProps) {
  const variantClass: Record<Variant, string> = {
    primary: 'bg-primary',
    secondary: 'bg-secondary',
    danger: 'bg-danger',
  }

  return (
    <TouchableOpacity
      onPress={onPress}
      disabled={disabled}
      className={`px-4 py-3 rounded-lg ${variantClass[variant]} ${disabled ? 'opacity-50' : ''} ${className}`}
    >
      <Text className="text-white font-semibold text-center">{label}</Text>
    </TouchableOpacity>
  )
}
```

```typescript
// src/components/Button/index.ts
export { Button } from './Button'
```

---

## `navigation-types.ts`

```typescript
// src/navigation/navigation-types.ts
import type { NativeStackScreenProps } from '@react-navigation/native-stack'
import type { BottomTabScreenProps } from '@react-navigation/bottom-tabs'
import type { NavigatorScreenParams } from '@react-navigation/native'

// — Auth (no autenticado) —
export type AuthStackParamList = {
  Welcome: undefined
  Login: undefined
  Register: undefined
}

// — Tabs principales (autenticado) —
export type AppTabsParamList = {
  Home: undefined
  Profile: undefined
}

// — App: stack autenticado (envuelve tabs + pantallas full-screen) —
export type AppStackParamList = {
  MainTabs: NavigatorScreenParams<AppTabsParamList>
  // Pantallas full-screen fuera de tabs:
  // ProductDetail: { productId: string }
  // Settings: undefined
}

// — Root (decide entre Auth y App) —
export type RootStackParamList = {
  Auth: undefined
  App: NavigatorScreenParams<AppStackParamList>
}

// Props tipadas para cada nivel
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

---

## `services/api.ts`

```typescript
// src/services/api.ts
import axios from 'axios'

export const api = axios.create({
  baseURL: process.env.EXPO_PUBLIC_API_URL,
  timeout: 10000,
})

export type ApiError = { message: string; status?: number }

export const parseApiError = (err: unknown): ApiError => {
  if (axios.isAxiosError(err)) {
    return { message: err.message, status: err.response?.status }
  }
  return { message: String(err) }
}
```

---

## `types/models.ts`

```typescript
// src/types/models.ts
export type User = { id: string; name: string; email: string; createdAt: Date }
export type Product = { id: string; name: string; price: number; description: string }
export type Order = {
  id: string
  userId: string
  items: OrderItem[]
  status: 'pending' | 'completed' | 'cancelled'
}
export type OrderItem = { productId: string; quantity: number; price: number }
```

```typescript
// src/types/index.ts
export * from './models'
export * from './api'
```

---

## Pantallas base

Las siguientes pantallas se crean en **todo** proyecto nuevo. Son el esqueleto mínimo de navegación.

### `Welcome` — Pantalla de bienvenida

```tsx
// src/screens/Welcome/index.tsx
import React from 'react'
import { View, Text } from 'react-native'
import { SafeAreaView } from 'react-native-safe-area-context'
import { Button } from '@/components/Button'
import { useNavigation } from '@react-navigation/native'
import type { NativeStackNavigationProp } from '@react-navigation/native-stack'
import type { AuthStackParamList } from '@/navigation'

type NavProp = NativeStackNavigationProp<AuthStackParamList, 'Welcome'>

export function Welcome() {
  const navigation = useNavigation<NavProp>()

  return (
    <SafeAreaView className="flex-1 bg-white">
      <View className="flex-1 items-center justify-center px-6">
        <Text className="text-3xl font-bold text-gray-900 text-center">
          Bienvenido a Mi App
        </Text>
        <Text className="text-base text-gray-500 text-center mt-3">
          Tu descripción aquí
        </Text>
        <View className="w-full mt-10">
          <Button label="Continuar" onPress={() => navigation.navigate('Login')} />
        </View>
      </View>
    </SafeAreaView>
  )
}
```

### `Login` — Inicio de sesión

```tsx
// src/screens/Login/index.tsx
import React from 'react'
import { View, Text, TextInput, TouchableOpacity } from 'react-native'
import { SafeAreaView } from 'react-native-safe-area-context'
import { Button } from '@/components/Button'
import { useAuth } from '@/hooks/useAuth'
import { useForm } from '@/hooks/useFormState'
import { isValidEmail } from '@/utils/validators'
import { useNavigation } from '@react-navigation/native'
import type { NativeStackNavigationProp } from '@react-navigation/native-stack'
import type { AuthStackParamList } from '@/navigation'

type NavProp = NativeStackNavigationProp<AuthStackParamList, 'Login'>

export function Login() {
  const navigation = useNavigation<NavProp>()
  const { login, loginAsGuest } = useAuth()

  const { values, errors, loading, handleChange, handleSubmit } = useForm({
    initialValues: { email: '', password: '' },
    onSubmit: async (vals) => {
      await login(vals)
    },
    validate: (vals) => {
      const errs: Record<string, string> = {}
      if (!isValidEmail(vals.email)) errs.email = 'Email inválido'
      if (vals.password.length < 8) errs.password = 'Mínimo 8 caracteres'
      return errs
    },
  })

  return (
    <SafeAreaView className="flex-1 bg-white">
      <View className="flex-1 justify-center px-6">
        <Text className="text-2xl font-bold text-gray-900 mb-6">Iniciar sesión</Text>

        <TextInput
          className="border border-gray-300 rounded-lg p-3 mb-1"
          value={values.email}
          onChangeText={(v) => handleChange('email', v)}
          placeholder="Email"
          keyboardType="email-address"
          autoCapitalize="none"
        />
        {errors.email ? <Text className="text-danger text-sm mb-3">{errors.email}</Text> : <View className="mb-3" />}

        <TextInput
          className="border border-gray-300 rounded-lg p-3 mb-1"
          value={values.password}
          onChangeText={(v) => handleChange('password', v)}
          placeholder="Contraseña"
          secureTextEntry
        />
        {errors.password ? <Text className="text-danger text-sm mb-3">{errors.password}</Text> : <View className="mb-3" />}

        <Button
          label={loading ? 'Cargando...' : 'Iniciar sesión'}
          onPress={handleSubmit}
          disabled={loading}
        />

        <Button
          label="Entrar como invitado"
          onPress={loginAsGuest}
          variant="secondary"
          disabled={loading}
          className="mt-3"
        />

        <TouchableOpacity onPress={() => navigation.navigate('Register')} className="mt-4">
          <Text className="text-center text-primary">
            ¿No tienes cuenta? <Text className="font-semibold">Regístrate</Text>
          </Text>
        </TouchableOpacity>
      </View>
    </SafeAreaView>
  )
}
```

### `Register` — Registro

```tsx
// src/screens/Register/index.tsx
import React from 'react'
import { View, Text, TextInput, TouchableOpacity } from 'react-native'
import { SafeAreaView } from 'react-native-safe-area-context'
import { Button } from '@/components/Button'
import { useAuth } from '@/hooks/useAuth'
import { useForm } from '@/hooks/useFormState'
import { isValidEmail } from '@/utils/validators'
import { useNavigation } from '@react-navigation/native'
import type { NativeStackNavigationProp } from '@react-navigation/native-stack'
import type { AuthStackParamList } from '@/navigation'

type NavProp = NativeStackNavigationProp<AuthStackParamList, 'Register'>

export function Register() {
  const navigation = useNavigation<NavProp>()
  const { register } = useAuth()

  const { values, errors, loading, handleChange, handleSubmit } = useForm({
    initialValues: { name: '', email: '', password: '', confirmPassword: '' },
    onSubmit: async (vals) => {
      await register({ name: vals.name, email: vals.email, password: vals.password })
    },
    validate: (vals) => {
      const errs: Record<string, string> = {}
      if (vals.name.trim().length < 2) errs.name = 'Nombre obligatorio'
      if (!isValidEmail(vals.email)) errs.email = 'Email inválido'
      if (vals.password.length < 8) errs.password = 'Mínimo 8 caracteres'
      if (vals.password !== vals.confirmPassword) errs.confirmPassword = 'Las contraseñas no coinciden'
      return errs
    },
  })

  return (
    <SafeAreaView className="flex-1 bg-white">
      <View className="flex-1 justify-center px-6">
        <Text className="text-2xl font-bold text-gray-900 mb-6">Crear cuenta</Text>

        <TextInput
          className="border border-gray-300 rounded-lg p-3 mb-1"
          value={values.name}
          onChangeText={(v) => handleChange('name', v)}
          placeholder="Nombre"
        />
        {errors.name ? <Text className="text-danger text-sm mb-3">{errors.name}</Text> : <View className="mb-3" />}

        <TextInput
          className="border border-gray-300 rounded-lg p-3 mb-1"
          value={values.email}
          onChangeText={(v) => handleChange('email', v)}
          placeholder="Email"
          keyboardType="email-address"
          autoCapitalize="none"
        />
        {errors.email ? <Text className="text-danger text-sm mb-3">{errors.email}</Text> : <View className="mb-3" />}

        <TextInput
          className="border border-gray-300 rounded-lg p-3 mb-1"
          value={values.password}
          onChangeText={(v) => handleChange('password', v)}
          placeholder="Contraseña"
          secureTextEntry
        />
        {errors.password ? <Text className="text-danger text-sm mb-3">{errors.password}</Text> : <View className="mb-3" />}

        <TextInput
          className="border border-gray-300 rounded-lg p-3 mb-1"
          value={values.confirmPassword}
          onChangeText={(v) => handleChange('confirmPassword', v)}
          placeholder="Confirmar contraseña"
          secureTextEntry
        />
        {errors.confirmPassword ? <Text className="text-danger text-sm mb-3">{errors.confirmPassword}</Text> : <View className="mb-3" />}

        <Button
          label={loading ? 'Cargando...' : 'Registrarse'}
          onPress={handleSubmit}
          disabled={loading}
        />

        <TouchableOpacity onPress={() => navigation.navigate('Login')} className="mt-4">
          <Text className="text-center text-primary">
            ¿Ya tienes cuenta? <Text className="font-semibold">Inicia sesión</Text>
          </Text>
        </TouchableOpacity>
      </View>
    </SafeAreaView>
  )
}
```

### `Home` — Pantalla principal (dentro de tabs)

Este template incluye una estructura basada en una app de real estate, usando `SafeAreaView`, `ScrollView` horizontal para recomendados, y tarjetas con imágenes. ⚠️ Usar `<Image>` de `react-native` (no `expo-image`) para mejor compatibilidad en Expo Go.

```tsx
// src/screens/Home/index.tsx
import React from 'react'
import { View, Text, ScrollView, TouchableOpacity, Image } from 'react-native'
import { SafeAreaView } from 'react-native-safe-area-context'
import { useAuth } from '@/hooks/useAuth'
import { Ionicons } from '@expo/vector-icons'

const RECOMMENDED = [
  {
    id: '1',
    title: 'Apartamento Soleado',
    location: 'París, Francia',
    price: 900,
    image: 'https://images.unsplash.com/photo-1502672260266-1c1ef2d93688?w=500',
  },
  {
    id: '2',
    title: 'Casa Exclusiva',
    location: 'París, Francia',
    price: 980,
    image: 'https://images.unsplash.com/photo-1512917774080-9991f1c4c750?w=500',
  },
]

const NEW_APARTMENTS = [
  {
    id: '3',
    title: 'Espacio Luminoso',
    location: 'Londres, Inglaterra',
    rooms: 4,
    image: 'https://images.unsplash.com/photo-1493809842364-78817add7ffb?w=300',
  },
  {
    id: '4',
    title: 'Apartamento Moderno',
    location: 'Nueva York, USA',
    rooms: 3,
    image: 'https://images.unsplash.com/photo-1494526585095-c41746248156?w=300',
  },
  {
    id: '5',
    title: 'Casa Acogedora',
    location: 'Madrid, España',
    rooms: 5,
    image: 'https://images.unsplash.com/photo-1430285561322-7808604715df?q=80&w=1170',
  },
  {
    id: '6',
    title: 'Piso con Vistas',
    location: 'Barcelona, España',
    rooms: 2,
    image: 'https://images.unsplash.com/photo-1493809842364-78817add7ffb?w=300',
  }
]

export function Home(): React.JSX.Element {
  const { user, isGuest } = useAuth()
  
  const userName = isGuest ? 'Invitado' : (user?.name || 'Usuario')

  return (
    <SafeAreaView className="flex-1 bg-white">
      <ScrollView className="flex-1" showsVerticalScrollIndicator={false}>
        {/* Header */}
        <View className="px-6 py-4 flex-row justify-between items-center">
          <View className="flex-row items-center">
            <Image 
              source={{ uri: 'https://i.pravatar.cc/150?u=1' }} 
              style={{ width: 48, height: 48, borderRadius: 24, marginRight: 12 }}
            />
            <View>
              <Text className="text-gray-400 text-sm">Buenos días,</Text>
              <Text className="text-xl font-bold text-gray-900">{userName}</Text>
            </View>
          </View>
          <TouchableOpacity className="p-2 bg-gray-50 rounded-full">
            <Ionicons name="notifications-outline" size={24} color="black" />
          </TouchableOpacity>
        </View>

        {/* Recommended Section */}
        <View className="mt-6">
          <View className="px-6 flex-row justify-between items-center mb-4">
            <Text className="text-2xl font-bold text-gray-900">Recomendados</Text>
            <TouchableOpacity>
              <Text className="text-orange-600 font-semibold">Ver todos</Text>
            </TouchableOpacity>
          </View>
          
          <ScrollView 
            horizontal 
            showsHorizontalScrollIndicator={false} 
            contentContainerStyle={{ paddingLeft: 24, paddingRight: 8 }}
          >
            {RECOMMENDED.map((item) => (
              <TouchableOpacity 
                key={item.id} 
                className="mr-4 bg-white rounded-3xl overflow-hidden shadow-sm border border-gray-100"
                style={{ width: 280 }}
              >
                <Image source={{ uri: item.image }} style={{ width: '100%', height: 192 }} />
                <View className="p-4">
                  <Text className="text-lg font-bold text-gray-900" numberOfLines={1}>
                    {item.title}
                  </Text>
                  <Text className="text-gray-400 mb-2">{item.location}</Text>
                  <View className="flex-row items-baseline">
                    <Text className="text-orange-600 text-xl font-bold">€{item.price}</Text>
                    <Text className="text-gray-400 ml-1">/mes</Text>
                  </View>
                </View>
              </TouchableOpacity>
            ))}
          </ScrollView>
        </View>

        {/* New Apartments Section */}
        <View className="mt-10 px-6 mb-10">
          <View className="flex-row justify-between items-center mb-4">
            <Text className="text-2xl font-bold text-gray-900">Nuevos apartamentos</Text>
            <TouchableOpacity>
              <Text className="text-orange-600 font-semibold">Ver todos</Text>
            </TouchableOpacity>
          </View>
          
          {NEW_APARTMENTS.map((item) => (
            <TouchableOpacity 
              key={item.id} 
              className="flex-row bg-white rounded-3xl p-3 shadow-sm border border-gray-50 mb-4 items-center"
            >
              <Image source={{ uri: item.image }} style={{ width: 96, height: 96, borderRadius: 16, marginRight: 16 }} />
              <View className="flex-1">
                <Text className="text-lg font-bold text-gray-900">{item.title}</Text>
                <Text className="text-gray-400 mb-2">{item.location}</Text>
                <View className="flex-row items-center">
                  <View className="bg-orange-50 p-1 rounded-md mr-1">
                    <Ionicons name="home-outline" size={14} color="#ea580c" />
                  </View>
                  <Text className="text-gray-600 text-sm">{item.rooms} habitaciones</Text>
                </View>
              </View>
            </TouchableOpacity>
          ))}
        </View>
      </ScrollView>
    </SafeAreaView>
  )
}
```

### `Profile` — Perfil de usuario (dentro de tabs)

```tsx
// src/screens/Profile/index.tsx
import React from 'react'
import { View, Text } from 'react-native'
import { SafeAreaView } from 'react-native-safe-area-context'
import { Button } from '@/components/Button'
import { useAuth } from '@/hooks/useAuth'

export function Profile(): React.JSX.Element {
  const { user, isGuest, logout } = useAuth()

  if (isGuest) {
    return (
      <SafeAreaView className="flex-1 bg-white">
        <View className="flex-1 justify-center px-6">
          <Text className="text-2xl font-bold text-gray-900 text-center mb-3">Eres un invitado</Text>
          <Text className="text-gray-500 text-center mb-6">
            Inicia sesión o crea una cuenta para acceder a tu perfil con todos los datos.
          </Text>
          <View className="gap-3">
            <Button label="Registrarse" onPress={logout} variant="primary" />
            <Button label="Iniciar sesión" onPress={logout} variant="secondary" />
          </View>
        </View>
      </SafeAreaView>
    )
  }

  return (
    <SafeAreaView className="flex-1 bg-white">
      <View className="flex-1 p-4">
        <Text className="text-2xl font-bold text-gray-900">Perfil</Text>
        <Text className="text-gray-500 mt-2">{user?.name ?? 'Usuario'}</Text>
        <Text className="text-gray-400 text-sm mt-1">{user?.email ?? ''}</Text>

        <View className="mt-10">
          <Button label="Cerrar sesión" variant="danger" onPress={logout} />
        </View>
      </View>
    </SafeAreaView>
  )
}
```

---

## Navegadores base

### `AuthStack.tsx` — Stack de autenticación

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

### `AppTabs.tsx` — Tab navigator principal

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

### `AppStack.tsx` — Stack principal autenticado

```tsx
// src/navigation/stacks/AppStack.tsx
import React from 'react'
import { createNativeStackNavigator } from '@react-navigation/native-stack'
import type { AppStackParamList } from '../navigation-types'
import { AppTabs } from './AppTabs'
// Importar pantallas full-screen aquí:
// import { ProductDetail } from '@/screens/ProductDetail'
// import { Settings } from '@/screens/Settings'

const Stack = createNativeStackNavigator<AppStackParamList>()

export function AppStack() {
  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      <Stack.Screen name="MainTabs" component={AppTabs} />
      {/* Pantallas full-screen (sin tab bar visible): */}
      {/* <Stack.Screen name="ProductDetail" component={ProductDetail} /> */}
      {/* <Stack.Screen name="Settings" component={Settings} /> */}
    </Stack.Navigator>
  )
}
```

### `RootNavigator.tsx` — Entry point con lógica auth

```tsx
// src/navigation/RootNavigator.tsx
import React from 'react'
import { createNativeStackNavigator } from '@react-navigation/native-stack'
import type { RootStackParamList } from './navigation-types'
import { useAuthStore } from '@/store/authStore'
import { AuthStack } from './stacks/AuthStack'
import { AppStack } from './stacks/AppStack'

const Stack = createNativeStackNavigator<RootStackParamList>()

export function RootNavigator() {
  const { token, isLoaded } = useAuthStore()

  if (!isLoaded) return null

  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      {token ? (
        <Stack.Screen name="App" component={AppStack} />
      ) : (
        <Stack.Screen name="Auth" component={AuthStack} />
      )}
    </Stack.Navigator>
  )
}
```

### `navigation/index.ts` — Re-exportar todo

```typescript
// src/navigation/index.ts
export { RootNavigator } from './RootNavigator'
export * from './navigation-types'
export * from './hooks'
```

---

## `App.tsx` — Punto de entrada

```tsx
// src/App.tsx
import '../global.css'
import React, { useEffect } from 'react'
import { NavigationContainer } from '@react-navigation/native'
import { SafeAreaProvider } from 'react-native-safe-area-context'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { verifyInstallation } from 'nativewind'
import { RootNavigator } from '@/navigation'
import { ErrorBoundary } from '@/components/ErrorBoundary'
import { useAuthStore } from '@/store/authStore'
import Toast from 'react-native-toast-message'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,
      retry: 2,
    },
  },
})

function AppBootstrap(): null {
  const loadToken = useAuthStore((s) => s.loadToken)
  useEffect(() => {
    if (__DEV__) {
      verifyInstallation()
    }
    loadToken()
  }, [loadToken])
  return null
}

export default function App(): React.JSX.Element {
  return (
    <SafeAreaProvider>
      <QueryClientProvider client={queryClient}>
        <ErrorBoundary>
          <AppBootstrap />
          <NavigationContainer>
            <RootNavigator />
          </NavigationContainer>
        </ErrorBoundary>
        <Toast />
      </QueryClientProvider>
    </SafeAreaProvider>
  )
}
```

> **Nota:** `<Toast />` se coloca fuera de `<ErrorBoundary>` para que las notificaciones sigan apareciendo incluso si la app falla. `<ErrorBoundary>` envuelve el árbol de navegación para capturar errores de renderizado no controlados. `verifyInstallation()` valida la configuración de Nativewind en modo desarrollo y no afecta producción.

---

## Manejo de errores global

### `ErrorBoundary` — Captura errores de renderizado

```tsx
// src/components/ErrorBoundary/ErrorBoundary.tsx
import React, { Component } from 'react'
import { View, Text } from 'react-native'
import { Button } from '@/components/Button'

type Props = { children: React.ReactNode }
type State = { hasError: boolean }

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false }

  static getDerivedStateFromError(): State {
    return { hasError: true }
  }

  componentDidCatch(error: Error, info: React.ErrorInfo): void {
    // Aquí enviar a servicio de logging (Sentry, Crashlytics, etc.)
    // logger.error('ErrorBoundary', { error, componentStack: info.componentStack })
  }

  private handleReset = (): void => {
    this.setState({ hasError: false })
  }

  render() {
    if (this.state.hasError) {
      return (
        <View className="flex-1 items-center justify-center bg-white px-6">
          <Text className="text-2xl font-bold text-gray-900 mb-2">Algo salió mal</Text>
          <Text className="text-base text-gray-500 text-center mb-6">
            Ha ocurrido un error inesperado. Inténtalo de nuevo.
          </Text>
          <Button label="Reintentar" onPress={this.handleReset} />
        </View>
      )
    }
    return this.props.children
  }
}
```

```typescript
// src/components/ErrorBoundary/index.ts
export { ErrorBoundary } from './ErrorBoundary'
```

### `useToast` — Hook para notificaciones

Usa `react-native-toast-message` como motor. El hook centraliza los tipos de mensaje para que las screens no dependan de la librería directamente.

```typescript
// src/hooks/useToast.ts
import Toast from 'react-native-toast-message'

type ToastType = 'success' | 'error' | 'info'

type ShowToastParams = {
  type: ToastType
  title: string
  message?: string
}

export function useToast() {
  const show = ({ type, title, message }: ShowToastParams): void => {
    Toast.show({
      type,
      text1: title,
      text2: message,
      position: 'top',
      visibilityTime: 3000,
    })
  }

  const showError = (title: string, message?: string): void =>
    show({ type: 'error', title, message })

  const showSuccess = (title: string, message?: string): void =>
    show({ type: 'success', title, message })

  const showInfo = (title: string, message?: string): void =>
    show({ type: 'info', title, message })

  return { show, showError, showSuccess, showInfo }
}
```

### Uso en hooks de datos (patrón recomendado)

Los errores de API se manejan en los hooks, no en las screens:

```typescript
// src/hooks/useProducts.ts (ejemplo)
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { getEntities, addEntity } from '@/services/api'
import { useToast } from '@/hooks/useToast'
import type { Product } from '@/types'

export function useProducts() {
  const { showError, showSuccess } = useToast()
  const queryClient = useQueryClient()

  const products = useQuery({
    queryKey: ['products'],
    queryFn: () => getEntities<Product>('products'),
  })

  const addProduct = useMutation({
    mutationFn: (data: Omit<Product, 'id'>) => addEntity<Product>('products', data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] })
      showSuccess('Producto creado')
    },
    onError: () => {
      showError('Error', 'No se pudo crear el producto')
    },
  })

  return { products, addProduct }
}
```

### Resumen del flujo de errores

```
Error de renderizado (crash)   → ErrorBoundary → pantalla "Algo salió mal" + botón reintentar
Error de API (red, 4xx, 5xx)   → Hook → useToast().showError() → toast visible al usuario
Error de validación (form)     → useForm validate() → mensaje inline bajo el campo
```

---

## Imágenes optimizadas con `expo-image`

**Regla:** Usar siempre `expo-image` en lugar de `<Image>` de React Native. Proporciona caché en disco, placeholders blur, transiciones y rendimiento nativo en listas.

### Uso básico

```tsx
import { Image } from 'expo-image'

// En cualquier componente o screen:
<Image
  source={{ uri: product.imageUrl }}
  placeholder={{ blurhash: product.blurhash }}
  contentFit="cover"
  transition={200}
  className="w-full h-48 rounded-lg"
/>
```

### En listas (FlatList / FlashList)

```tsx
import { Image } from 'expo-image'
import { FlatList, View, Text } from 'react-native'

type ProductCardProps = {
  imageUrl: string
  blurhash?: string
  id: string
  name: string
}

function ProductCard({ imageUrl, blurhash, id, name }: ProductCardProps) {
  return (
    <View className="mb-4">
      <Image
        source={{ uri: imageUrl }}
        placeholder={blurhash ? { blurhash } : undefined}
        contentFit="cover"
        transition={200}
        recyclingKey={id}
        className="w-full h-48 rounded-lg"
      />
      <Text className="text-base font-semibold mt-2">{name}</Text>
    </View>
  )
}
```

> **`recyclingKey`** es obligatorio en listas: evita que al reciclar celdas se muestre brevemente la imagen del item anterior.

### Prefetch (opcional, UX premium)

```tsx
import { Image } from 'expo-image'

// Pre-cargar las primeras imágenes antes de mostrar la lista
useEffect(() => {
  const urls = products.slice(0, 10).map(p => p.imageUrl)
  Image.prefetch(urls)
}, [products])
```

### Props principales

| Prop | Tipo | Descripción |
|---|---|---|
| `source` | `{ uri: string }` | URL de la imagen |
| `placeholder` | `{ blurhash: string }` o `{ thumbhash: string }` | Placeholder mientras carga |
| `contentFit` | `'cover' \| 'contain' \| 'fill' \| 'none'` | Equivalente a CSS `object-fit` |
| `transition` | `number` (ms) | Duración del fade-in al cargar |
| `recyclingKey` | `string` | ID único para reciclaje en listas |
| `cachePolicy` | `'memory-disk' \| 'memory' \| 'disk' \| 'none'` | Estrategia de caché (default: `'disk'`) |

> **Nota backend:** Para que el placeholder blur funcione, el API debe devolver un campo `blurhash` (string) por cada imagen. Se genera server-side al subir la imagen con librerías como `blurhash` (npm) o `kornrunner/blurhash` (PHP/Laravel).

---

## Comandos CLI frecuentes

```bash
# Crear proyecto (pnpm — recomendado)
pnpm create expo-app my-app --template expo-template-blank
cd my-app

# Instalar dependencias
pnpm add nativewind axios @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs
pnpm add react-native-screens react-native-safe-area-context
pnpm add @tanstack/react-query zustand expo-secure-store expo-image react-native-toast-message
pnpm add -D tailwindcss @nativewind/typescript babel-plugin-module-resolver

# Inicializar tailwind
pnpm dlx tailwindcss init

# Instalar dependencias de tests
pnpm add -D jest @testing-library/react-native @testing-library/jest-native @types/jest

# Fallback npm
npx create-expo-app my-app --template expo-template-blank
npm install nativewind axios @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs
npm install react-native-screens react-native-safe-area-context
npm install @tanstack/react-query zustand expo-secure-store
npm install -D tailwindcss @nativewind/typescript babel-plugin-module-resolver
```
