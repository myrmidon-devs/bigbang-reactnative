---
title: Project setup
version: 1.5
purpose: "Pasos mínimos para bootstrap de un proyecto Expo + TypeScript + Nativewind"
---
# Project setup

Pasos para crear un proyecto desde cero. Seguir en orden. El agente debe verificar cada paso antes de pasar al siguiente.

---

## Prerequisitos

- Node.js LTS (v20+)
- Git inicializado en el proyecto
- **`pnpm` recomendado.** Si no está instalado, ver Regla 0 en `docs/agent-instructions.md`.

---

## Paso 1 — Crear el proyecto base

> Objetivo: partir de un proyecto Expo limpio, pero **sin** asumir que `latest` es una línea estable compatible con Expo Go.

> **REGLA CRÍTICA — Siempre en la raíz del workspace:**
> Ejecuta el comando **desde dentro del directorio de destino** usando `.` (punto) como nombre.
> Nunca uses `create-expo-app my-app` — eso crea un subdirectorio `my-app/` y el proyecto
> quedaría anidado fuera de la raíz. Los archivos de scaffolding (package.json, src/, app.json…)
> deben vivir en la raíz del workspace junto a `docs/`, `.copilot/`, etc.

```bash
# pnpm (recomendado) — crea el proyecto en el directorio actual
pnpm create expo-app .

# npm (fallback)
npx create-expo-app .
```

Tras el scaffold, editar `app.json` para poner el nombre real del proyecto en los campos `name` y `slug`.

### Regla crítica de Expo Go

- No instales `expo@latest` a ciegas.
- No uses plantillas `@latest` como fuente de verdad del SDK.
- Fija una línea estable de SDK antes del primer arranque. Verifícala con `expo-doctor` y asegúrate de que el proyecto arranca en Expo Go sin errores de incompatibilidad.
---

## Paso 2 — Migrar a TypeScript

Renombra `App.js` a `App.tsx`. Crea `tsconfig.json`:

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/navigation": ["src/navigation/index.ts"]
    },
    "plugins": [
      {
        "name": "@nativewind/typescript"
      }
    ]
  },
  "include": ["src", "*.ts", "*.tsx", "global.d.ts", "nativewind-env.d.ts"]
}
```

---

## Paso 3 — Instalar dependencias

> Usa `expo install` para paquetes del ecosistema Expo: el CLI resolverá automáticamente las versiones compatibles con el SDK activo del proyecto. Para paquetes sin integración Expo, instala sin pin de versión y verifica compatibilidad con `expo-doctor` al final.

```bash
# Paquetes del ecosistema Expo (versión resuelta automáticamente por expo install)
# pnpm (recomendado)
pnpm expo install expo-secure-store expo-status-bar expo-image

# npm (fallback)
npx expo install expo-secure-store expo-status-bar expo-image

# Paquetes de navegación y UI
# pnpm
pnpm add nativewind axios
pnpm add @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs
pnpm add react-native-screens react-native-safe-area-context
pnpm add @tanstack/react-query zustand react-native-toast-message

# npm (fallback)
npm install nativewind axios
npm install @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs
npm install react-native-screens react-native-safe-area-context
npm install @tanstack/react-query zustand react-native-toast-message

# Dev dependencies
# pnpm
pnpm add -D tailwindcss @nativewind/typescript babel-plugin-module-resolver @types/react @types/react-native

# npm
npm install -D tailwindcss @nativewind/typescript babel-plugin-module-resolver @types/react @types/react-native
```

---

## Paso 4 — Inicializar Tailwind

```bash
# pnpm
pnpm dlx tailwindcss init

# npm
npx tailwindcss init
```

Reemplaza el contenido de `tailwind.config.js`:

```javascript
module.exports = {
  content: ['./src/**/*.{js,jsx,ts,tsx}'],
  presets: [require('nativewind/preset')],
  theme: {
    extend: {
      colors: {
        primary: '#007AFF',
        secondary: '#5AC8FA',
        success: '#34C759',
        error: '#FF3B30',
        warning: '#FF9500',
        danger: '#FF3B30',
      },
      spacing: {
        xs: '4px',
        sm: '8px',
        md: '16px',
        lg: '24px',
        xl: '32px',
      },
    },
  },
  plugins: [],
}
```

---

## Paso 5 — Configurar Babel (path alias + Nativewind)

Crea o actualiza `babel.config.js`:

```javascript
module.exports = function (api) {
  api.cache(true)
  return {
    presets: [['babel-preset-expo', { jsxImportSource: 'nativewind' }], 'nativewind/babel'],
    plugins: [
      [
        'module-resolver',
        {
          alias: {
            '@': './src',
          },
        },
      ],
    ],
  }
}
```

---

## Paso 6 — Configurar Metro + CSS global

Crea `metro.config.js`:

```javascript
const { getDefaultConfig } = require('expo/metro-config')
const { withNativeWind } = require('nativewind/metro')

const config = getDefaultConfig(__dirname)

module.exports = withNativeWind(config, { input: './global.css' })
```

Crea `global.css` en la raíz del proyecto:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

## Paso 7 — Crear `global.d.ts`

En la raíz del proyecto (junto a `package.json`):

```typescript
// global.d.ts
/// <reference types="nativewind/types" />
```

---

## Paso 8 — Crear estructura de carpetas

```bash
mkdir -p src/screens src/components src/navigation/stacks src/services src/hooks src/store src/types src/utils src/assets/images src/assets/icons src/assets/fonts
```

Estructura resultante:

```
src/
├── screens/
├── components/
├── navigation/
│   └── stacks/
├── services/
├── hooks/
├── store/
├── types/
├── utils/
├── assets/
│   ├── images/
│   ├── icons/
│   └── fonts/
└── App.tsx
```

---

## Paso 9 — `.env.example`

```bash
# .env.example
EXPO_PUBLIC_API_URL=https://api.tu-dominio.com/v1
EXPO_PUBLIC_APP_ENV=development
```

Crear también `.env` local (no subir a git):

```bash
cp .env.example .env
```

Añadir a `.gitignore`:

```
.env
.env.local
```

---

## Paso 10 — `src/App.tsx`

```tsx
import '../global.css'
import React, { useEffect } from 'react'
import { NavigationContainer } from '@react-navigation/native'
import { SafeAreaProvider } from 'react-native-safe-area-context'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { RootNavigator } from '@/navigation'
import { useAuthStore } from '@/store/authStore'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,
      retry: 2,
    },
  },
})

function AppBootstrap() {
  const loadToken = useAuthStore((s) => s.loadToken)
  useEffect(() => { loadToken() }, [loadToken])
  return null
}

export default function App() {
  return (
    <SafeAreaProvider>
      <QueryClientProvider client={queryClient}>
        <AppBootstrap />
        <NavigationContainer>
          <RootNavigator />
        </NavigationContainer>
      </QueryClientProvider>
    </SafeAreaProvider>
  )
}
```

ℹ️ **SafeAreaProvider** envuelve toda la app en un solo alto para que todas las screens respeten el safe area del dispositivo (status bar, notch, dynamic island). Luego cada screen usa individualmente `<SafeAreaView>` (ver `docs/conventions.md` para la convención).

---

## Comandos de desarrollo

```bash
# pnpm (recomendado)
pnpm start     # Levanta Expo + Metro bundler
pnpm android   # Compila y abre en emulador Android
pnpm ios       # Compila y abre en simulador iOS

# npm (fallback)
npm run start
npm run android
npm run ios
```

---

## Verificación final del setup

- [ ] `npx tsc --noEmit` pasa sin errores
- [ ] `npx expo-doctor@latest` pasa sin mismatches del SDK
- [ ] `pnpm start` (o `npm start`) levanta el bundler sin errores
- [ ] Path alias `@/` resuelve correctamente
- [ ] `tailwind.config.js` usa `nativewind/preset`
- [ ] `babel.config.js` usa `jsxImportSource: 'nativewind'` y `nativewind/babel` en `presets`
- [ ] `metro.config.js` existe con `withNativeWind`
- [ ] `global.css` existe y `src/App.tsx` lo importa
- [ ] Carpeta `src/` creada con todas las subcarpetas
- [ ] `.env.example` creado y `.env` en `.gitignore`

---

> Siguientes pasos: `docs/structure-guide.md` → `docs/navigation-patterns.md` → `docs/templates-snippets.md`
