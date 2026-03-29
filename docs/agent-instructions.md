---
title: Instrucciones para el agente IA
version: 2.3
---

# Instrucciones para el agente IA

Este archivo define el flujo operativo exacto que debe seguir un agente IA para crear un proyecto desde cero o incorporar cambios a un proyecto existente usando esta guía.

---

## Regla 0 — Gestor de paquetes

**Antes de cualquier instalación:**

1. Comprueba si `pnpm` está disponible: `pnpm --version`
2. Si está disponible → usa `pnpm` para todo (instalar, ejecutar scripts, etc.)
3. Si **no** está disponible → pregunta al usuario: *"pnpm no está instalado. ¿Quieres instalarlo con `npm install -g pnpm`? Si no, usaré npm."*
4. Si el usuario rechaza → usa `npm` como fallback en todos los comandos

### Regla 0.1 — Expo estable antes del primer arranque

**Nunca** uses `expo@latest` ni plantillas `@latest` de forma acrítica si el objetivo es abrir la app en Expo Go.

1. Crea el proyecto base sin asumir que `latest` es una línea estable.
2. Alinea explícitamente `expo`, `react` y `react-native` a una línea estable del SDK.
3. Si el proyecto se va a abrir en Expo Go, usa una línea compatible con el cliente instalado.
4. Ejecuta `npx expo-doctor@latest` antes del primer `pnpm start`.

---

## Flujo principal — Crear proyecto desde cero

### PASO 1 — Leer `docs/project-setup.md`

**Acción:** Ejecutar los pasos de bootstrap en orden:
1. Crear app con `create-expo-app` usando `.` (punto) como destino — **NUNCA** `create-expo-app nombre-proyecto`, eso crea un subdirectorio. El proyecto siempre se genera en la raíz del workspace.
2. Actualizar `app.json` con el nombre real del proyecto (`name` y `slug`)
3. Renombrar `App.js` → `App.tsx`
4. Crear estructura de carpetas `src/`
5. Instalar dependencias (nativewind, axios, react-navigation)
6. Crear `tsconfig.json` con alias `@/`
7. Crear `babel.config.js` con `module-resolver`
8. Crear `.env.example`

**Verificación obligatoria:**
- [ ] Carpeta `src/` creada con subcarpetas: `screens/ components/ navigation/ services/ hooks/ store/ types/ utils/ theme/ assets/`
- [ ] `tsconfig.json` tiene `"@/*": ["src/*"]` en `paths`
- [ ] `babel.config.js` tiene `module-resolver` apuntando a `./src`
- [ ] `npx tsc --noEmit` no da errores
- [ ] `.env.example` creado

---

### PASO 2 — Leer `docs/structure-guide.md`

**Acción:** Interiorizar las reglas de arquitectura antes de crear ningún archivo.

**Verificación obligatoria:**
- [ ] ¿El proyecto sigue la arquitectura de 3 capas? (Screen → Hook → Service)
- [ ] ¿Cada carpeta tiene un `index.ts` que re-exporta?
- [ ] ¿Las carpetas siguen kebab-case y los componentes PascalCase?
- [ ] ¿No hay ningún `any` en el código?
- [ ] ¿Ningún componente supera las 300 líneas?

---

### PASO 3 — Leer `docs/conventions.md`

**Acción:** Aplicar convenciones de nombres, TypeScript e imports en todos los archivos creados.

**Verificación obligatoria:**
- [ ] ¿Los imports usan `@/` (no rutas relativas `../`)?
- [ ] ¿Los imports están ordenados: React → librerías → locales?
- [ ] ¿Las funciones tienen tipo de retorno explícito?
- [ ] ¿No se usa `StyleSheet.create()` en ningún archivo?

---

### PASO 4 — Leer `docs/nativewind-theme.md`

**Acción:** Configurar Nativewind y el tema visual del proyecto.
1. Configurar `tailwind.config.js` con colores y espaciado del proyecto
2. Añadir `presets: [require('nativewind/preset')]` en `tailwind.config.js`
3. Configurar `babel.config.js` con `['babel-preset-expo', { jsxImportSource: 'nativewind' }]` y `nativewind/babel` en `presets`
4. Crear `metro.config.js` con `withNativeWind(config, { input: './global.css' })`
5. Crear `global.css` e importarlo desde `src/App.tsx`
6. Añadir plugin `@nativewind/typescript` en `tsconfig.json`

**Verificación obligatoria:**
- [ ] ¿`tailwind.config.js` define colores `primary`, `secondary`, `error`?
- [ ] ¿`tailwind.config.js` usa `nativewind/preset`?
- [ ] ¿`babel.config.js` usa `jsxImportSource: 'nativewind'` y `nativewind/babel` en `presets`?
- [ ] ¿Existe `metro.config.js` con `withNativeWind` apuntando a `global.css`?
- [ ] ¿`src/App.tsx` importa `../global.css`?
- [ ] ¿No hay `StyleSheet.create()` en ningún componente?
- [ ] ¿Un componente de prueba muestra estilos correctamente?

---

### PASO 5 — Leer `docs/navigation-patterns.md`

**Acción:** Crear el sistema de navegación tipado.
1. Crear `src/navigation/navigation-types.ts` con todos los `ParamList` (`RootStackParamList`, `AuthStackParamList`, `AppStackParamList`, `AppTabsParamList`)
2. Crear `src/navigation/hooks.ts` con `useRootNavigation()`, `useAuthNavigation()` y `useAppNavigation()`
3. Crear `src/navigation/RootNavigator.tsx` (con lógica condicional `useAuthStore`)
4. Crear `src/navigation/stacks/AuthStack.tsx` (Welcome, Login, Register)
5. Crear `src/navigation/stacks/AppTabs.tsx` (BottomTabNavigator: Home, Profile)
6. Crear `src/navigation/stacks/AppStack.tsx` (NativeStack que envuelve AppTabs + pantallas full-screen)
7. Crear `src/navigation/index.ts` (re-exporta todo)

**Verificación obligatoria:**
- [ ] ¿Existe `declare global { namespace ReactNavigation { ... } }` en `navigation-types.ts`?
- [ ] ¿Se usa `NavigatorScreenParams` para tipar la relación `AppStack → AppTabs`?
- [ ] ¿Las pantallas importan tipos desde `@/navigation` (path alias)?
- [ ] ¿No hay rutas duplicadas en múltiples archivos?
- [ ] ¿El `RootNavigator` apunta a `AppStack` (no a `AppTabs` directamente)?
- [ ] ¿`RootNavigator` usa `useAuthStore` para decidir entre `AuthStack` y `AppStack`?
- [ ] ¿`AppStack` tiene `MainTabs` como primera screen y registra pantallas full-screen?
- [ ] ¿`AppTabs` usa `@react-navigation/bottom-tabs` con iconos de `@expo/vector-icons`?

---

### PASO 6 — Crear pantallas base

**Acción:** Crear las pantallas mínimas que todo proyecto necesita. Usar las plantillas de `docs/templates-snippets.md`.

1. Crear `src/screens/Welcome/index.tsx` — Pantalla de bienvenida con título y botón "Continuar" → navega a Login
2. Crear `src/screens/Login/index.tsx` — Formulario email/password con `useForm` + `useAuth`, enlace a Register
3. Crear `src/screens/Register/index.tsx` — Formulario nombre/email/password/confirmar con `useForm` + `useAuth`, enlace a Login
4. Crear `src/screens/Home/index.tsx` — Pantalla principal base (contenido placeholder)
5. Crear `src/screens/Profile/index.tsx` — Pantalla de perfil con datos placeholder y botón "Cerrar sesión" (`useAuth().logout`)

**Verificación obligatoria:**
- [ ] ¿Existen las 5 pantallas: Welcome, Login, Register, Home, Profile?
- [ ] ¿Welcome navega a Login al pulsar "Continuar"?
- [ ] ¿Login usa `useForm` + `useAuth` para autenticación?
- [ ] ¿Login tiene enlace a Register y Register tiene enlace a Login?
- [ ] ¿Register valida que las contraseñas coincidan?
- [ ] ¿Profile tiene botón de logout funcional con `useAuth().logout`?
- [ ] ¿Las pantallas usan `SafeAreaView` de `react-native-safe-area-context`?
- [ ] ¿Ninguna pantalla llama a Axios directamente? (todo a través de hooks)

---

### PASO 7 — Leer `docs/services-and-api.md`

**Acción:** Crear la capa de servicios con Axios.
1. Crear `src/services/api.ts` con Axios client, interceptores y `parseApiError`
2. Crear `src/services/auth.ts` si hay autenticación
3. Crear `src/services/logger.ts` (reemplaza `console.log`)

**Verificación obligatoria:**
- [ ] ¿Axios está instalado como dependencia?
- [ ] ¿`api.ts` usa `process.env.EXPO_PUBLIC_API_URL` como base URL?
- [ ] ¿`parseApiError` maneja tanto errores Axios como errores genéricos?
- [ ] ¿No hay `console.log` directo en los servicios?
- [ ] ¿Las funciones de servicio son puras (sin imports de React)?

---

### PASO 8 — Leer `docs/hooks-and-state.md`

**Acción:** Crear el patrón de hooks y el store de estado global.
1. Crear hooks por entidad con React Query: `useProducts.ts`, `useAuth.ts`
2. Crear `src/store/authStore.ts` con Zustand para el estado de sesión
3. Envolver la app en `QueryClientProvider` en `App.tsx`

**Verificación obligatoria:**
- [ ] ¿Los hooks de datos del servidor usan `useQuery`/`useMutation` de React Query?
- [ ] ¿El estado de sesión (token, usuario) vive en `authStore` con Zustand?
- [ ] ¿Los hooks llaman a servicios (no a Axios directamente)?
- [ ] ¿Las screens obtienen datos del hook, no del servicio?
- [ ] ¿No se usa Redux Toolkit?

---

### PASO 9 — Leer `docs/templates-snippets.md`

**Acción:** Usar las plantillas para crear los primeros archivos del proyecto.
- Las pantallas base ya se crearon en el PASO 6 usando las plantillas de este doc
- `src/types/models.ts` con los modelos del proyecto
- `src/types/index.ts` que re-exporta todo

**Verificación obligatoria:**
- [ ] ¿Cada nuevo archivo sigue el esqueleto de la plantilla correspondiente?
- [ ] ¿Los `index.ts` re-exportan correctamente?
- [ ] ¿`App.tsx` importa `global.css` y usa `QueryClientProvider` + `NavigationContainer` + `RootNavigator`?

---

### PASO 10 — Leer `docs/testing-ci.md` (opcional para MVP)

**Acción:** Configurar tests si el proyecto los requiere.
1. Instalar dependencias de test
2. Crear `jest.config.js`
3. Crear estructura `tests/__mocks__/`

**Verificación obligatoria:**
- [ ] ¿`npm test` (o `pnpm test`) pasa sin errores?

---

## Reglas operativas

- **No crear ningún archivo antes de leer el doc correspondiente.**
- Ante la duda entre dos enfoques, elegir el más simple.
- Si algo no está en la guía, consulta con el usuario antes de inventar convenciones.
- Si identificas una mejora a la guía, anótala en `docs/changelog.md` (no modifiques los otros docs sin consenso).
- No usar `console.log`; usar `src/services/logger.ts`.
- No crear archivos `.styles.ts`.
- No usar `StyleSheet.create()`.
- Estado global: **Zustand** para sesión auth + **React Query** para datos del servidor. No usar Redux Toolkit.
- Iconos y animaciones: seguir la jerarquía de `docs/animations-and-icons.md` (iconos estáticos → `@expo/vector-icons`, animaciones JSON → `lottie-react-native`, animaciones de UI → `react-native-reanimated`).
- Plugins nativos: **antes de instalar cualquier plugin nativo** (cámara, mapas, PDF, background tasks, etc.), consultar `docs/native-plugins.md`. Si la necesidad no está cubierta, proponer una opción al usuario antes de instalar.
- Manejo de errores: **todo proyecto** debe tener `ErrorBoundary` en `App.tsx` + `useToast` hook + `<Toast />` como último hijo. Errores de API van en hooks (`onError`), nunca en screens. No usar `Alert.alert()` directamente.
- Diseño responsive: **todas las pantallas deben ser responsive**. Usar Flexbox, fracciones (`w-1/2`), breakpoints de Nativewind (`sm:`, `md:`, `lg:`), `useWindowDimensions()` para lógica dinámica. No anchos fijos en píxeles. Diseñar para posible conversión a web (Expo Web).

---

## Añadir una nueva screen — Flujo rápido

```
1. Crear carpeta src/screens/NombrePantalla/
2. Crear index.tsx (usar plantilla de docs/templates-snippets.md)
3. Si tiene componentes propios → crear components/ dentro de la carpeta
4. Añadir ruta en navigation/navigation-types.ts:
   - ¿Es un tab? → AppTabsParamList
   - ¿Es pantalla full-screen sin tabs? → AppStackParamList
   - ¿Es pantalla de auth? → AuthStackParamList
5. Registrar la pantalla en el navegador correspondiente:
   - Tab → stacks/AppTabs.tsx
   - Full-screen autenticada → stacks/AppStack.tsx
   - Auth → stacks/AuthStack.tsx
6. Crear hook en src/hooks/useNombreFeature.ts si la pantalla tiene datos
7. Crear función en src/services/api.ts si el hook llama a la API
8. Si la pantalla usa iconos o animaciones → consultar docs/animations-and-icons.md
9. Si la pantalla usa plugins nativos (cámara, mapas, PDF, etc.) → consultar docs/native-plugins.md
```

---

## Checklist de validación final

Antes de dar por terminado el proyecto, ejecutar la validación completa de `docs/structure-guide.md` (sección "Checklist de Validación").
