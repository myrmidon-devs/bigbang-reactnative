---
title: Nativewind & Tailwind
version: 1.3
---

# Nativewind v4 y Tailwind CSS

Nativewind lleva Tailwind CSS a React Native mediante clases de utilidad en `className`. Es la única forma de aplicar estilos en este proyecto.

Esta guía está escrita para **Nativewind v4**. No uses configuraciones heredadas de v2 o v3 como `TailwindProvider`, porque en este template la integración correcta depende de Babel, Metro, el preset de Tailwind y la importación explícita de `global.css`.

---

## Instalación Completa

```bash
# pnpm (recomendado)
pnpm add nativewind
pnpm add -D tailwindcss @nativewind/typescript
pnpm dlx tailwindcss init

# npm (fallback)
npm install nativewind
npm install -D tailwindcss @nativewind/typescript
npx tailwindcss init
```

Si el proyecto se va a abrir en Expo Go, no uses `expo@latest` ni plantillas `@latest` como fuente de verdad. Primero fija una línea estable del SDK y después instala Nativewind.

---

## Configuración `tailwind.config.js`

En la **raíz** del proyecto (NO en `src/`):

```javascript
module.exports = {
  content: ['./src/**/*.{js,jsx,ts,tsx}'],
  presets: [require('nativewind/preset')],
  theme: {
    extend: {
      colors: {
        primary: '#007AFF',
        secondary: '#5856D6',
        success: '#34C759',
        danger: '#FF3B30',
        warning: '#FF9500',
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

## `babel.config.js` — JSX + Nativewind

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

`nativewind/babel` va en `presets`, no en `plugins`.

---

## `metro.config.js` — integrar Nativewind con Metro

```javascript
const { getDefaultConfig } = require('expo/metro-config')
const { withNativeWind } = require('nativewind/metro')

const config = getDefaultConfig(__dirname)

module.exports = withNativeWind(config, { input: './global.css' })
```

---

## `global.css` — directivas Tailwind

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Importa este archivo una sola vez desde `src/App.tsx`:

```tsx
import '../global.css'
```

---

## `tsconfig.json` — tipos de Nativewind

```json
{
  "compilerOptions": {
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

## `src/App.tsx` — sin `TailwindProvider`

```tsx
import '../global.css'

export default function App() {
  return (
    <>{/* resto de la app */}</>
  )
}
```

En Nativewind v4, con Expo, la integración correcta del template se apoya en `global.css`, Babel, Metro y el preset de Tailwind. `TailwindProvider` no forma parte del setup base aquí.

---

## Verificación mínima antes del primer `pnpm start`

1. `tailwind.config.js` incluye `presets: [require('nativewind/preset')]`.
2. `babel.config.js` usa `['babel-preset-expo', { jsxImportSource: 'nativewind' }]`.
3. `nativewind/babel` está en `presets`.
4. `metro.config.js` existe y usa `withNativeWind`.
5. `global.css` existe.
6. `src/App.tsx` importa `../global.css`.
7. `npx expo-doctor@latest` no reporta mismatches del SDK.

> **`verifyInstallation()` en modo desarrollo:**
> `src/App.tsx` llama a `verifyInstallation()` de `nativewind` dentro de `if (__DEV__)`.
> Esto valida en consola que Babel, Metro y el preset están configurados correctamente.
> Se ejecuta solo en desarrollo y no afecta el bundle de producción. No eliminar.

---

## Guía de Uso

### ✅ Correcto — usar `className` directamente

```tsx
import { View, Text } from 'react-native'

export function Card() {
  return (
    <View className="bg-white rounded-lg p-4 shadow">
      <Text className="text-lg font-bold text-gray-900">Título</Text>
      <Text className="text-sm text-gray-500 mt-1">Subtítulo</Text>
    </View>
  )
}
```

### ❌ Incorrecto — no usar `StyleSheet`

```tsx
// ❌ NO HACER ESTO
const styles = StyleSheet.create({
  card: { backgroundColor: 'white', borderRadius: 8, padding: 16 },
})
```

---

## Patrones Comunes

### Clases Condicionales

```tsx
type CardProps = { isActive: boolean }

export function Card({ isActive }: CardProps) {
  return (
    <View className={`p-4 rounded-lg ${isActive ? 'bg-primary' : 'bg-gray-200'}`}>
      <Text className={isActive ? 'text-white' : 'text-gray-700'}>
        Contenido
      </Text>
    </View>
  )
}
```

### Espaciado Consistente

```tsx
// ✅ Valores de tailwind.config.js
<View className="px-md py-lg gap-4">
  <Text>Item 1</Text>
  <Text>Item 2</Text>
</View>

// ❌ No mezclar con StyleSheet
<View style={{ paddingHorizontal: 16 }}>
```

### Dark Mode

```tsx
<View className="bg-white dark:bg-gray-900 p-4">
  <Text className="text-gray-900 dark:text-white font-medium">
    Texto adaptable
  </Text>
</View>
```

---

## Casos de Excepción

Solo salir de Nativewind cuando sea estrictamente necesario:

```tsx
// ✅ Animaciones complejas — aquí sí se puede usar Animated API
import Animated, { useAnimatedStyle, withSpring } from 'react-native-reanimated'

export function AnimatedCard() {
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: withSpring(1.1) }],
  }))

  return (
    // className para estilos base, style solo para la animación
    <Animated.View className="bg-white rounded-lg p-4" style={animatedStyle}>
      <Text className="text-gray-900">Contenido animado</Text>
    </Animated.View>
  )
}

// ❌ NO hacer esto sin razón
<View className="p-4" style={{ marginBottom: 10 }}>
  {/* Mejor: className="p-4 mb-2.5" */}
</View>
```

---

## Diseño Responsive

Todas las pantallas **deben ser responsive por defecto**. El proyecto está pensado para móviles, pero debe soportar una posible conversión a web (Expo Web / React Native Web).

### Reglas de Layout

- ✅ **Flexbox siempre** — usar `flex-1`, `flex-row`, `flex-wrap`, `gap-*` como base de todo layout
- ✅ **Porcentajes y fracciones** — usar `w-full`, `w-1/2`, `w-1/3` en lugar de anchos fijos
- ✅ **Contenido centrado con ancho máximo** — en pantallas grandes, limitar el ancho con `max-w-lg self-center w-full`
- ✅ **`useWindowDimensions()`** — para lógica responsive en runtime (ej: columnas en FlatList)
- ✅ **Breakpoints de Nativewind** — usar prefijos `sm:`, `md:`, `lg:`, `xl:` para adaptar layout
- ❌ **No anchos fijos en píxeles** para contenedores (ej: ~~`w-[375px]`~~). Solo permitido para iconos, avatares o elementos decorativos pequeños
- ❌ **No usar `Dimensions.get('window')` estático** — usar `useWindowDimensions()` que se actualiza en rotación y resize

### Breakpoints de Nativewind

| Prefijo | Min Width | Uso típico |
|---|---|---|
| (sin prefijo) | 0px | Móvil (base) |
| `sm:` | 640px | Móvil grande / tablet vertical |
| `md:` | 768px | Tablet |
| `lg:` | 1024px | Tablet horizontal / desktop |
| `xl:` | 1280px | Desktop ancho |

### Patrón — Layout Adaptativo

```tsx
// ✅ Contenido centrado que no se estira en pantallas grandes
export function ScreenWrapper({ children }: { children: React.ReactNode }) {
  return (
    <View className="flex-1 bg-white dark:bg-gray-900">
      <View className="flex-1 w-full max-w-lg self-center px-4">
        {children}
      </View>
    </View>
  )
}
```

### Patrón — Grid Responsive

```tsx
// ✅ Cards que se adaptan: 1 columna en móvil, 2 en tablet, 3 en desktop
export function ProductGrid({ products }: { products: Product[] }) {
  return (
    <View className="flex-row flex-wrap gap-4 px-4">
      {products.map((product) => (
        <View key={product.id} className="w-full sm:w-[48%] lg:w-[31%]">
          <ProductCard product={product} />
        </View>
      ))}
    </View>
  )
}
```

### Patrón — FlatList con Columnas Dinámicas

```tsx
import { useWindowDimensions, FlatList } from 'react-native'

export function AdaptiveList({ data }: { data: Product[] }) {
  const { width } = useWindowDimensions()
  const numColumns = width >= 1024 ? 3 : width >= 640 ? 2 : 1

  return (
    <FlatList
      key={numColumns} // fuerza re-render al cambiar columnas
      data={data}
      numColumns={numColumns}
      renderItem={({ item }) => (
        <View className="flex-1 p-2">
          <ProductCard product={item} />
        </View>
      )}
    />
  )
}
```

### Preparación para Web (Expo Web)

- ✅ Usar `react-native` imports estándar (compatibles con React Native Web)
- ✅ Evitar APIs nativas sin fallback web (ej: `NativeModules` directos)
- ✅ Los breakpoints de Nativewind funcionan igual en web via media queries
- ✅ `useWindowDimensions()` funciona tanto en native como en web

---

## Reglas Absolutas

- ✅ Estilos **siempre** via `className`
- ✅ Colores y espaciado personalizados en `tailwind.config.js`
- ✅ Dark mode con prefijo `dark:`
- ❌ No `StyleSheet.create()`
- ❌ No archivos `.styles.ts`
- ❌ No mezclar `className` con `style` salvo animaciones
