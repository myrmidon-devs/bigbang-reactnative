---
title: Convenciones
version: 1.4
---

# Convenciones de código

Este documento define las reglas de nomenclatura, TypeScript, imports y anti-patrones. Es obligatorio leerlo antes de crear o modificar cualquier archivo.

---

## Convenciones de Nombres

| Tipo | Convención | Ejemplo |
|---|---|---|
| Carpetas | kebab-case | `my-component/` |
| Componentes | PascalCase | `Button.tsx` |
| Funciones / Hooks | camelCase | `useAuth.ts` |
| Tipos | PascalCase | `User`, `Product` |
| Constantes | UPPER_SNAKE_CASE | `API_BASE_URL` |
| Clases Tailwind | kebab-case | `text-primary-500`, `px-4` |

---

## Reglas de TypeScript

- ✅ **OBLIGATORIO** TypeScript en todos los archivos (`.tsx`, `.ts`)
- ✅ **OBLIGATORIO** Tipos explícitos en funciones y sus retornos
- ✅ **OBLIGATORIO** Archivos `src/types/` para tipos globales
- ✅ Usar `type` para definir tipos (no `interface`, salvo extensión OOP)
- ❌ **PROHIBIDO** `any` — usar `unknown` y narrowing si es necesario
- ❌ **PROHIBIDO** Imports con rutas relativas largas (`../../../`)

```typescript
// ✅ Bien
type User = {
  id: string
  name: string
  email: string
}

const fetchUser = async (id: string): Promise<User> => {
  // ...
}

// ❌ Mal
const fetchUser = async (id) => { // sin tipo
  // ...
}

const data: any = response // prohibido
```

---

## Reglas de Archivos

- ✅ Un componente = una carpeta (excepto componentes de una sola línea)
- ✅ Máximo 300 líneas por archivo (componentes: 150 líneas)
- ✅ `index.ts` en cada carpeta para re-exportar
- ✅ Estilos con `className` Nativewind, no en archivos separados
- ✅ Excepciones: animaciones complejas pueden usar `Animated` + Nativewind
- ✅ Usar `expo-image` en lugar de `<Image>` de React Native (caché, blur placeholder, rendimiento en listas)
  - **Excepción:** La pantalla `Home` del template usa `<Image>` de `react-native` para asegurar compatibilidad con Expo Go sin configuración adicional. En proyectos que no usen Expo Go, migrar a `expo-image`.
- ✅ **Todas las screens deben importar y usar `SafeAreaView` desde `react-native-safe-area-context`** como contenedor raíz. Esto respeta el status bar, notch, y dynamic island en todos los dispositivos.
- ❌ No crear archivos `.styles.ts`
- ❌ No usar `StyleSheet.create()`
- ❌ No mezclar Tailwind con StyleSheet
- ❌ No usar `<Image>` de `react-native` directamente (usar `expo-image`, salvo la excepción documentada en Home)

### SafeAreaView en Screens

Todas las screens deben tener esta estructura base:

```tsx
// src/screens/ScreenName/index.tsx
import React from 'react'
import { View, Text, ScrollView } from 'react-native'
import { SafeAreaView } from 'react-native-safe-area-context'

export function ScreenName(): React.JSX.Element {
  return (
    <SafeAreaView className="flex-1 bg-white">
      <ScrollView>
        <View className="p-4">
          <Text className="text-2xl font-bold">Screen Title</Text>
        </View>
      </ScrollView>
    </SafeAreaView>
  )
}
```

ℹ️ `SafeAreaProvider` está configurado en `src/App.tsx` (global). Cada screen usa `SafeAreaView` individualmente para respetar el safe area.

```
Button/
├── Button.tsx   ← Lógica + estilos con className
└── index.ts     ← Re-exporta
```

---

## Reglas de Imports

### ✅ Permitido

```typescript
// Siempre con path alias @/
import { Button } from '@/components/Button'
import { useAuth } from '@/hooks/useAuth'
import { fetchUser } from '@/services/api'
import { isValidEmail } from '@/utils/validators'
import type { User } from '@/types'
import type { AppScreenProps } from '@/navigation'
```

### ❌ Prohibido

```typescript
// Rutas relativas largas
import { Button } from '../../../components/Button'

// Imports circulares
// authSlice → authService → authSlice ← NO

// Desde services hacia componentes
import { Button } from '@/components' // en services: NO

// any explícito
const x: any = value
```

### Orden de imports recomendado

```typescript
// 1. React y librerías de React
import React, { useState, useEffect } from 'react'
import { View, Text } from 'react-native'

// 2. Librerías de terceros
import axios from 'axios'

// 3. Alias @/ propios
import { useAuth } from '@/hooks/useAuth'
import type { User } from '@/types'

// 4. Relativos locales (máximo 1 nivel, excepcional)
import { HeroSection } from './components/HeroSection'
```

---

## Configuración de Path Aliases

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

### `babel.config.js` (para Metro / Expo)

```javascript
module.exports = {
  presets: ['babel-preset-expo'],
  plugins: [
    [
      'module-resolver',
      {
        root: ['./src'],
        alias: {
          '@': './src',
        },
      },
    ],
  ],
}
```

Instalar el plugin con pnpm:

```bash
pnpm add -D babel-plugin-module-resolver
```

---

## Manejo de Errores

- ✅ `ErrorBoundary` en `App.tsx` envolviendo `NavigationContainer` (captura errores de renderizado)
- ✅ `useToast()` hook para notificaciones al usuario (success, error, info)
- ✅ Errores de API → se manejan en hooks con `onError` de React Query → `useToast().showError()`
- ✅ Errores de formulario → inline en `useForm` → texto bajo el campo
- ❌ No usar `Alert.alert()` directamente en screens (centralizar en `useToast`)
- ❌ No silenciar errores con `catch` vacíos
- ❌ No mostrar errores técnicos al usuario (ej: "AxiosError: 500")

---

## Diseño Responsive

- ✅ **Todas las pantallas deben ser responsive** — no se acepta layout que solo funcione en un tamaño
- ✅ Usar **Flexbox** (`flex-1`, `flex-row`, `flex-wrap`, `gap-*`) como base de todo layout
- ✅ Usar **porcentajes y fracciones** (`w-full`, `w-1/2`, `w-1/3`) en lugar de anchos fijos en píxeles
- ✅ Limitar ancho de contenido con `max-w-lg self-center w-full` en pantallas grandes
- ✅ Usar `useWindowDimensions()` para lógica responsive en runtime (nunca `Dimensions.get()` estático)
- ✅ Usar breakpoints de Nativewind (`sm:`, `md:`, `lg:`, `xl:`) para adaptar layout según ancho
- ✅ Diseñar pensando en futura conversión a **web** (Expo Web / React Native Web)
- ❌ No usar anchos fijos en píxeles para contenedores (~~`w-[375px]`~~)
- ❌ No usar `Dimensions.get('window')` estático (se rompe en rotación/resize)

---

## Anti-patrones Generales

```typescript
// ❌ Importar relativamente desde una screen
import { useRootNavigation } from '../../../navigation/hooks'
// ✅ Correcto
import { useRootNavigation } from '@/navigation'

// ❌ Llamar a Axios desde una screen
import axios from 'axios'
export function HomeScreen() {
  useEffect(() => { axios.get('/products') }, []) // NO
}
// ✅ Correcto: la screen llama al hook, el hook llama al service
const { products } = useProducts()

// ❌ Console.log en producción
console.log('data', data)
// ✅ Usar el logger del proyecto
import { logger } from '@/services/logger'
logger.info('data', data)

// ❌ Screen sin SafeAreaView (título detrás del reloj)
export function Home() {
  return (
    <ScrollView className="flex-1">
      <Text className="text-2xl">Home</Text>
    </ScrollView>
  )
}
// ✅ Screen con SafeAreaView (respeta status bar)
export function Home() {
  return (
    <SafeAreaView className="flex-1">
      <ScrollView>
        <Text className="text-2xl">Home</Text>
      </ScrollView>
    </SafeAreaView>
  )
}

