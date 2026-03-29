---
title: Estructura del proyecto
version: 2.2
source: REACT_NATIVE_STRUCTURE_GUIDE.md
---

# Estructura y reglas principales

Este documento contiene la arquitectura completa, reglas por carpeta, checklist y referencia rápida. Es la fuente de verdad sobre cómo organizar el proyecto.

---

## Filosofía General

### Principios Fundamentales

1. **Simplicidad sobre complejidad** — Layer-based, no feature-based
2. **Un solo nivel de anidación** — Máximo 2 niveles de profundidad en `src/`
3. **Separación clara de concerns** — Cada carpeta tiene un propósito específico
4. **DRY** — Reutilización centralizada
5. **Escalabilidad sin refactorización** — Crece sin reorganizar
6. **Desarrollador único en mente** — Sin conflictos de merge complejos

### Arquitectura Mental (3 Capas)

```
Presentación (Screens + Components)
          ↓
Servicios (API, Storage, Auth)  ← gestionados por Hooks
          ↓
Utilidades (Helpers, Constants, Types)
```

**La Regla de Oro:** las Screens no hacen llamadas HTTP. El flujo es:

1. **Screen (El Pintor):** solo consume Hooks. No sabe qué es una URL ni Axios.
2. **Hook (El Gerente):** gestiona `useState`/`useEffect`, formatea datos para la vista y llama a Services.
3. **Service (El Obrero):** TypeScript puro, realiza la comunicación HTTP. Sin dependencias de React.

---

## Árbol de Carpetas Completo

```
proyecto-rn/
├── src/
│   ├── screens/              ← Pantallas principales
│   ├── components/           ← Componentes reutilizables
│   ├── navigation/           ← Configuración de navegación
│   ├── services/             ← Lógica de negocio (API, Auth, Storage)
│   ├── hooks/                ← Custom hooks reutilizables
│   ├── store/                ← Gestión de estado global
│   ├── types/                ← Tipos TypeScript globales
│   ├── utils/                ← Funciones auxiliares
│   ├── theme/                ← Referencia a tailwind.config.js
│   ├── assets/               ← Imágenes, íconos, fuentes
│   └── App.tsx               ← Punto de entrada principal
├── tests/
│   ├── __mocks__/
│   ├── unit/
│   └── integration/
├── .env
├── .env.example
├── app.json
├── package.json
├── tsconfig.json
├── babel.config.js
├── metro.config.js
├── jest.config.js
├── tailwind.config.js
└── README.md
```

---

## Reglas por Carpeta

### `src/screens/`

**Propósito:** Pantallas principales de la aplicación.

Reglas:
- ✅ Una carpeta por pantalla, nombre en PascalCase: `Home/`, `Profile/`
- ✅ Archivo principal `index.tsx`
- ✅ Subcomponentes propios en `components/`, hooks propios en `hooks/`
- ✅ Máximo 300 líneas en el componente principal
- ❌ No importar otras screens directamente
- ❌ No hacer llamadas a Axios o fetch directamente
- ❌ No contener lógica de negocio compleja

Estructura interna:
```
Home/
├── index.tsx
├── components/
│   ├── HeroSection.tsx
│   └── FeaturedProducts.tsx
└── hooks/
    └── useFeaturedData.ts
```

### `src/components/`

**Propósito:** Componentes reutilizables usados en 2+ screens.

Reglas:
- ✅ Una carpeta por componente, nombre en PascalCase: `Button/`, `Card/`
- ✅ `index.ts` para re-exportar
- ✅ Componentes puros (sin lógica de negocio)
- ✅ Props bien tipadas
- ✅ Máximo 150 líneas por componente
- ❌ No hacer llamadas a servicios o hooks de API
- ❌ No usar contexto directamente (pasarlo por props)

Estructura:
```
Button/
├── Button.tsx
└── index.ts

ErrorBoundary/
├── ErrorBoundary.tsx   ← Class component (único caso donde se usa class)
└── index.ts
```

Ver detalle completo en `docs/navigation-patterns.md`.

El `RootNavigator` debe comprobar `useAuthStore` para decidir si mostrar `AuthStack` o `AppStack`, sin depender de `AsyncStorage` directamente.

Estructura:
```
navigation/
├── navigation-types.ts   ← Centro único de tipado
├── hooks.ts              ← Hooks de navegación tipados
├── RootNavigator.tsx     ← Entry point principal (lee authStore para decidir stack)
├── linking.ts            ← Deep linking (opcional)
├── index.ts              ← Re-exporta todo públicamente
└── stacks/
    ├── AuthStack.tsx      ← Welcome, Login, Register
    ├── AppStack.tsx       ← NativeStack: envuelve AppTabs + pantallas full-screen
    └── AppTabs.tsx        ← BottomTabNavigator (Home, Profile, …)
```

> **Jerarquía clave:** `RootNavigator → AppStack → AppTabs`. Las pantallas autenticadas que **no** necesitan tab bar se registran en `AppStack` (ej: ProductDetail, Settings, EditProfile). Las pantallas con tab bar se registran en `AppTabs`.

### `src/services/`

Ver detalle completo en `docs/services-and-api.md`.

Este proyecto usa un patrón genérico CRUD con métodos reutilizables para cualquier entidad del backend, incluyendo rutas anidadas (`/entity/{id}/subEntity/{id}`).

Estructura:
```
services/
├── api.ts           ← Cliente Axios + métodos genéricos CRUD + sub-entidades + upload
├── auth.ts          ← Login, logout, registro y social auth
├── storage.ts       ← Token seguro con expo-secure-store (NO AsyncStorage)
└── logger.ts        ← Logging (no console.log)
```

### `src/store/`

**Propósito:** Estado global reactivo. Usa **Zustand para auth** y **React Query para datos del servidor**. No mezcles ambas librerías para el mismo dato.

Reglas:
- ✅ `authStore.ts` con Zustand — token, usuario, `isGuest`, `loadToken()`, `setAuth()`, `setAsGuest()`, `clearAuth()`
- ✅ Datos del servidor (entidades) → React Query (`useQuery`/`useMutation` en hooks)
- ❌ No guardar listas de entidades en Zustand (eso va en React Query cache)
- ❌ No usar Redux Toolkit en este template

Estructura:
```
store/
└── authStore.ts   ← Estado de sesión (token + user + isGuest + loadToken)
```

Ver detalle completo en `docs/hooks-and-state.md`.

### `src/hooks/`

Ver detalle completo en `docs/hooks-and-state.md`.

Estructura:
```
hooks/
├── useAuth.ts
├── useFetch.ts
├── useStorage.ts
├── useFormState.ts
└── useToast.ts        ← Notificaciones toast estandarizadas
```

### `src/types/`

**Propósito:** Tipos TypeScript globales compartidos.

Reglas:
- ✅ Tipos compartidos entre múltiples archivos
- ✅ Nombres en PascalCase: `User`, `Product`
- ✅ `index.ts` re-exporta todos los tipos
- ✅ Archivos separados por dominio: `models.ts`, `api.ts`
- ✅ Comentarios JSDoc para tipos complejos
- ❌ No tipos locales (esos van en el componente)
- ❌ No lógica, solo tipos

Estructura:
```
types/
├── index.ts      ← export * from todos
├── api.ts        ← Tipos de respuestas API
├── models.ts     ← User, Product, Order, OrderItem
├── navigation.ts ← Tipos de navegación (backup; fuente en navigation-types.ts)
└── services.ts   ← Tipos de servicios
```

Ejemplo `src/types/models.ts`:
```typescript
export type User = { id: string; name: string; email: string; createdAt: Date }
export type Product = { id: string; name: string; price: number; description: string }
export type Order = { id: string; userId: string; items: OrderItem[]; status: 'pending' | 'completed' | 'cancelled' }
export type OrderItem = { productId: string; quantity: number; price: number }
```

### `src/utils/`

**Propósito:** Funciones auxiliares puras y constantes.

Reglas:
- ✅ Funciones puras reutilizables, tipadas completamente
- ✅ Constantes globales (UPPER_SNAKE_CASE)
- ❌ No lógica compleja (eso va en services)
- ❌ No dependencias de React ni side effects
- ❌ No `console.log` (usar logger de services/logger.ts)

Estructura:
```
utils/
├── constants.ts   ← API_BASE_URL, TIMEOUT, etc.
├── validators.ts  ← isValidEmail, isValidPassword
├── formatters.ts  ← formatCurrency, formatDate
└── helpers.ts     ← Funciones auxiliares
```

Ejemplo:
```typescript
// constants.ts
export const API_BASE_URL = process.env.EXPO_PUBLIC_API_URL
export const API_TIMEOUT = 10000
export const MAX_RETRY_ATTEMPTS = 3

// validators.ts
export const isValidEmail = (email: string): boolean =>
  /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)

export const isValidPassword = (password: string): boolean =>
  password.length >= 8

// formatters.ts
export const formatCurrency = (amount: number, currency = 'USD'): string =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(amount)

export const formatDate = (date: Date): string =>
  date.toLocaleDateString('es-ES')
```

### `src/theme/`

**Propósito:** Referencia a la configuración de Tailwind CSS.

Reglas:
- ✅ `tailwind.config.js` en la **raíz** del proyecto (NO en `src/`)
- ✅ Colores y espaciado personalizados en `tailwind.config.js`
- ❌ No crear archivos TypeScript de colores/spacing
- ❌ No usar estilos inline en lugar de clases Tailwind

Ver instalación y configuración completa en `docs/nativewind-theme.md`.

### `src/assets/`

**Propósito:** Recursos estáticos.

Reglas:
- ✅ Carpetas por tipo: `images/`, `icons/`, `fonts/`
- ✅ Nombres en kebab-case
- ✅ Optimizar imágenes antes de añadirlas
- ✅ Preferir librerías de iconos nativas (no SVGs pesados)
- ❌ No archivos sin usar

Estructura:
```
assets/
├── images/
│   ├── hero.png
│   └── logo.png
├── icons/
└── fonts/
    └── Roboto-Regular.ttf
```

---

## Referencia Rápida — Qué Va Dónde

| Necesidad | Ubicación | Ejemplo |
|---|---|---|
| Componente reutilizable | `components/` | `Button`, `Card`, `ErrorBoundary` |
| Pantalla de app | `screens/` | `Home`, `Profile` |
| Tipos de rutas | `navigation/navigation-types.ts` | `RootStackParamList` |
| Hooks de nav tipados | `navigation/hooks.ts` | `useRootNavigation` |
| Configuración stacks | `navigation/stacks/` | `AuthStack.tsx`, `AppStack.tsx`, `AppTabs.tsx` |
| Lógica de API | `services/` | `api.ts` |
| Lógica compartida | `hooks/` | `useAuth`, `useProducts`, `useToast` |
| Validaciones | `utils/validators.ts` | `isValidEmail` |
| Constantes | `utils/constants.ts` | `API_BASE_URL` |
| Tipos globales | `types/` | `User`, `Product` |
| Configuración Tailwind | `tailwind.config.js` (raíz) | Colores, espaciado |
| Imágenes/Iconos | `assets/` | `images/`, `icons/` |

---

## Cuándo Romper las Reglas

Hay casos puntuales donde las reglas pueden relajarse:

- ✅ Componente de una sola línea → puede estar inline
- ✅ Tipo usado solo en un lugar → puede estar en el componente
- ✅ Constante muy específica → puede estar en el archivo
- ✅ Utility muy corta → puede estar en el componente

**Regla general:** estos casos deben ser la excepción, no la norma.

---

## Checklist de Validación Completo

### Estructura General
- [ ] Carpetas en kebab-case
- [ ] Archivos de componentes en PascalCase
- [ ] Archivos de funciones/hooks en camelCase
- [ ] Todos los archivos tienen extensión `.ts` o `.tsx`
- [ ] Existe `index.ts` en cada carpeta principal
- [ ] No hay carpetas vacías

### TypeScript
- [ ] No hay `any` type
- [ ] Funciones tienen tipo de retorno explícito
- [ ] Props tipadas con `type` (no `interface` para consistencia)
- [ ] Tipos globales en `src/types/`
- [ ] Imports con rutas absolutas (`@/`)

### Componentes
- [ ] Props tipadas
- [ ] Sin lógica de negocio
- [ ] Sin llamadas a API
- [ ] Estilos en `className` (Nativewind)
- [ ] No hay archivos `.styles.ts`
- [ ] No usa `StyleSheet.create()`

### Screens
- [ ] Una pantalla por carpeta
- [ ] Usa hooks para lógica
- [ ] NO usa Axios directamente
- [ ] Importa componentes reutilizables desde `@/components`
- [ ] Tipos de navegación importados desde `@/navigation`

### Navegación
- [ ] `navigation-types.ts` es el único lugar donde se definen rutas
- [ ] Todos los imports usan `@/navigation`
- [ ] Hooks tipados en `navigation/hooks.ts`
- [ ] `RootNavigator.tsx` limpio sin lógica de negocio
- [ ] No hay duplicación de tipos de rutas
- [ ] Path alias configurado en `tsconfig.json`
- [ ] No hay imports relativos (`../`) en pantallas

### Servicios
- [ ] Sin dependencias de React
- [ ] Sin hooks
- [ ] Funciones puras con tipos de retorno explícitos
- [ ] Error handling completo
- [ ] Tipos exportados
- [ ] No usa `console.log` (usar logger)

### Hooks
- [ ] Nombre empieza con `use`
- [ ] Reutilizable e independiente de componentes
- [ ] Actúa como puente entre Screens y Services
- [ ] Sin lógica de UI
- [ ] Sin side effects innecesarios
- [ ] Agrupa lógica por entidad (ej: `useProducts` tiene todo lo de productos)

### Importes
- [ ] Usa `@/` path alias
- [ ] No imports relativos (`../`)
- [ ] No imports circulares
- [ ] Orden: React → librerías → alias `@/` → locales

### Nativewind y Estilos
- [ ] Todos los estilos usan `className`
- [ ] No hay archivos `.styles.ts`
- [ ] No usa `StyleSheet.create()`
- [ ] No mezcla `className` con `style` innecesariamente
- [ ] `tailwind.config.js` configurado en raíz
- [ ] Colores personalizados en `tailwind.config.js`

### Responsive
- [ ] Todas las pantallas son responsive (funcionan en móvil, tablet y web)
- [ ] Layouts usan Flexbox (`flex-1`, `flex-row`, `flex-wrap`), no anchos fijos
- [ ] Contenido limitado con `max-w-lg self-center w-full` en pantallas grandes
- [ ] Listas y grids usan `useWindowDimensions()` para columnas dinámicas
- [ ] No se usa `Dimensions.get('window')` estático
- [ ] Breakpoints de Nativewind (`sm:`, `md:`, `lg:`) aplicados donde el layout cambia

### Manejo de Errores
- [ ] `ErrorBoundary` envuelve la app en `App.tsx`
- [ ] `<Toast />` renderizado como último hijo en `App.tsx`
- [ ] `useToast` hook centraliza notificaciones (no usar `Alert.alert` directo)
- [ ] Errores de API se manejan en hooks (`onError`), no en screens
- [ ] Errores de formulario se manejan inline con `useForm`

### Testing
- [ ] Unit tests para `services/` y `utils/`
- [ ] Integration tests para `hooks/` con mocks
- [ ] Mocks en `tests/__mocks__/`
