---
title: Changelog
version: 2.0
---

# Changelog

## 2026-03 v2.0 — Alineación final de skill, references y documentación base

**Motivación:** La skill `bigbang-reactnative`, sus phases de referencia y varios documentos base habían acumulado pequeñas desalineaciones. Ninguna por separado parecía grande, pero en conjunto permitían que el agente generase proyectos incompletos, mezclara convenciones antiguas o no comunicara correctamente el cierre y los siguientes pasos.

**Cambios:**
- `.copilot/skills/bigbang-reactnative/SKILL.md` — Reforzada la ejecución estricta: seguir pasos uno a uno, no inventar nada, validar cada checklist antes de avanzar y obligar a incluir los siguientes pasos al finalizar.
- `.copilot/skills/bigbang-reactnative/SKILL.md` — Derivación automática de `PROJECT_NAME` desde la carpeta raíz; eliminado el comportamiento de pedir nombre/directorio al usuario. Blindado el scaffold en raíz con `create-expo-app .`.
- `.copilot/skills/bigbang-reactnative/references/phase-1-bootstrap.md` — Bootstrap fijado en la raíz del workspace y texto normalizado a inglés en la fase. Añadida edición explícita de `app.json` (`name` y `slug`).
- `.copilot/skills/bigbang-reactnative/references/phase-4-state-hooks.md` — Guest mode y recovery alineados: `isGuest`, `setAsGuest()` y `useToast.ts` ya forman parte de la definición y la recuperación de la fase.
- `.copilot/skills/bigbang-reactnative/references/phase-5-navigation.md` — `RootNavigator` alineado con guest mode usando `token || isGuest`.
- `.copilot/skills/bigbang-reactnative/references/phase-6-ui.md` — Home ya no puede degradarse a placeholder; se exige la plantilla completa con mock data. `App.tsx` documenta correctamente `SafeAreaProvider`, `ErrorBoundary`, `verifyInstallation()`, `Toast` y el orden real de anidación.
- `docs/agent-instructions.md` — Añadida regla de ejecución estricta. Home ya exige la plantilla completa y `App.tsx` se valida con `SafeAreaProvider`, `ErrorBoundary`, `NavigationContainer` y `Toast`.
- `docs/project-setup.md` — Confirmado scaffold siempre en la raíz del repo con `create-expo-app .` y verificación con `expo-doctor`.
- `docs/navigation-patterns.md` y `docs/templates-snippets.md` — `RootNavigator` y `App.tsx` sincronizados con guest mode, `SafeAreaProvider`, `ErrorBoundary`, `Toast` y `verifyInstallation()`.
- `docs/hooks-and-state.md` — `authStore` y `useAuth` alineados con `isGuest` y `setAsGuest()`.
- `docs/services-and-api.md` — Eliminadas referencias obsoletas a `clearToken()` y a `notifications.ts` como parte obligatoria del baseline.
- `docs/structure-guide.md` y `README.md` — Eliminadas referencias ambiguas a RTK/Redux Toolkit como alternativa del template base. El estado global del baseline queda fijado en Zustand para auth.
- `docs/testing-ci.md` — Corregido `setupFilesAfterEnv` y actualizados ejemplos de mocks hacia `expo-secure-store`.

---

## 2026-03 v1.9 — Endurecimiento preventivo de Expo Go + NativeWind v4

**Motivación:** La configuración real validada del template había quedado desalineada respecto a README, instrucciones y skill. Eso permitía dos errores evitables en proyectos nuevos: instalar una línea canary de Expo incompatible con Expo Go y aplicar una configuración incompleta de NativeWind v4 basada en `TailwindProvider`.

**Cambios:**
- `README.md` — Actualizado de Nativewind v2 a Nativewind v4. Añadida advertencia explícita contra `expo@latest` y plantillas `@latest` cuando el objetivo es Expo Go.
- `.github/copilot-instructions.md` — Reglas no negociables alineadas con Nativewind v4 y bootstrap Expo estable verificado con `expo-doctor`.
- `docs/agent-instructions.md` — Nuevo guardrail de Expo estable y checklist de NativeWind v4 basado en Babel, Metro, `global.css` y `nativewind/preset`.
- `docs/project-setup.md` — Bootstrap reescrito para fijar una línea estable del SDK, configurar `tailwind.config.js`, `babel.config.js`, `metro.config.js`, `global.css` y validar con `expo-doctor`.
- `docs/nativewind-theme.md` — Guía actualizada a Nativewind v4. Eliminadas referencias a `TailwindProvider` como setup base del template.
- `docs/templates-snippets.md` — Plantilla de `App.tsx` alineada con `global.css` + `ErrorBoundary` + `NavigationContainer`, sin `TailwindProvider`.
- `.copilot/skills/bigbang-reactnative/references/phase-1-bootstrap.md` — Fase de bootstrap alineada con Expo estable y NativeWind v4.
- `.copilot/skills/bigbang-reactnative/references/phase-6-ui.md` — Fase de UI alineada con `global.css` y sin `TailwindProvider`.
- `docs/expo-go-sdk54-nativewind-troubleshooting.md` — Eliminada la nota postmortem; el aprendizaje útil se absorbe en la documentación base.
- `output.css` — Eliminado artefacto temporal de diagnóstico.

---

## 2026-03 v1.7 — Revisión de calidad pre-publicación

## 2026-03 v1.8 — Troubleshooting Expo Go + NativeWind

**Motivación:** Se diagnosticó y resolvió una cadena de errores al ejecutar el proyecto en Expo Go Android tras migrar desde una versión canary de Expo a SDK 54 estable. Los síntomas incluían incompatibilidad de Expo Go, errores de Babel, un falso error ESM de Metro en Windows y un crash nativo en Fabric con cast de String a Boolean.

**Cambios:**
- `docs/expo-go-sdk54-nativewind-troubleshooting.md` — Nueva nota técnica con diagnóstico completo, causas raíz, solución aplicada, validaciones realizadas y flujo recomendado de depuración para Expo Go + NativeWind v4.

---

**Motivación:** Auditoría exhaustiva de toda la documentación y skill `bigbang-reactnative` antes de subir a la nube. Se detectaron y corrigieron inconsistencias entre archivos.

**Cambios:**
- `docs/services-and-api.md` — Corregido comentario de `logger.ts`: de `src/utils/logger.ts` a `src/services/logger.ts`. Añadidos `RegisterPayload` type y función `register()` en `auth.ts`.
- `docs/conventions.md` — Corregido import de logger: de `@/utils/logger` a `@/services/logger`.
- `docs/structure-guide.md` — Eliminado `logger.ts` de la estructura `utils/` (solo vive en `services/`). Clarificada referencia a `services/logger.ts`.
- `docs/hooks-and-state.md` — Añadido método `register()` al hook `useAuth()` con su import correspondiente.
- `.gitignore` — Añadido `coverage/` para reportes de tests.
- Skill `bigbang-reactnative` — Movida de `~/.copilot/skills/` al repo en `.copilot/skills/`. Rutas actualizadas.

---

## 2026-03 v1.6 — Diseño responsive obligatorio

**Motivación:** Todas las pantallas deben ser responsive desde el día 0. El proyecto está pensado para móviles, pero debe soportar una posible conversión a web (Expo Web / React Native Web). Sin reglas responsive explícitas, los desarrolladores tienden a usar anchos fijos que se rompen en tablets y web.

**Principios documentados:**
- **Flexbox como base** — `flex-1`, `flex-row`, `flex-wrap`, `gap-*` en todo layout
- **Fracciones, no píxeles** — `w-full`, `w-1/2`, `w-1/3` en lugar de `w-[375px]`
- **Contenido con ancho máximo** — `max-w-lg self-center w-full` para que no se estire en pantallas grandes
- **Breakpoints de Nativewind** — `sm:` (640px), `md:` (768px), `lg:` (1024px), `xl:` (1280px)
- **`useWindowDimensions()`** — para lógica responsive en runtime (ej: columnas dinámicas en FlatList)
- **Preparación para web** — imports estándar de `react-native`, evitar APIs nativas sin fallback web

**Cambios:**
- `docs/nativewind-theme.md` (v1.2) — Añadida sección completa "Diseño Responsive": reglas de layout (7 items), tabla de breakpoints de Nativewind, patrón de layout adaptativo (ScreenWrapper con max-w-lg), patrón de grid responsive (cards con breakpoints), patrón de FlatList con columnas dinámicas (useWindowDimensions), notas de preparación para Expo Web.
- `docs/conventions.md` (v1.4) — Añadida sección "Diseño Responsive": 7 reglas positivas (Flexbox, porcentajes, max-w, useWindowDimensions, breakpoints, diseño para web) y 2 prohibiciones (anchos fijos, Dimensions.get estático).
- `docs/structure-guide.md` — Añadida subsección "Responsive" en el checklist de validación (6 items: responsive en todas las pantallas, Flexbox, max-w-lg, useWindowDimensions, no Dimensions.get estático, breakpoints).
- `docs/agent-instructions.md` — Añadida regla operativa: todas las pantallas deben ser responsive, con referencia a Flexbox, breakpoints, useWindowDimensions y preparación para web.

---

## 2026-03 v1.5 — Manejo de errores global (ErrorBoundary + Toast)

**Motivación:** Todo proyecto necesita un sistema de errores unificado desde el día 0. Sin él, los errores se pierden silenciosamente o se muestran de forma inconsistente (Alert.alert en un sitio, console.log en otro, nada en otro).

**Patrón implementado — 3 niveles de error:**
- **Errores de renderizado (crash)** → `ErrorBoundary` (class component) → pantalla "Algo salió mal" + botón reintentar
- **Errores de API (red, 4xx, 5xx)** → hooks con React Query `onError` → `useToast().showError()` → toast visible
- **Errores de validación (formulario)** → `useForm` validate → mensajes inline bajo campos

**Cambios:**
- `docs/templates-snippets.md` (v1.5) — Añadidas plantillas: `ErrorBoundary` (class component con fallback UI), `useToast` hook (wrapper tipado sobre react-native-toast-message con showError/showSuccess/showInfo), ejemplo de uso en hooks de datos con onSuccess/onError. Actualizado `App.tsx` con `<ErrorBoundary>` + `<Toast />`. Actualizado comando CLI con `react-native-toast-message`.
- `docs/structure-guide.md` (v2.2) — Añadida carpeta `ErrorBoundary/` en estructura de components. Añadido `useToast.ts` en estructura de hooks. Añadida sección "Manejo de Errores" en checklist de validación. Actualizada tabla de referencia rápida.
- `docs/conventions.md` (v1.3) — Añadida sección "Manejo de Errores": reglas de ErrorBoundary, useToast, prohibición de Alert.alert directo, prohibición de catch vacíos.
- `docs/agent-instructions.md` (v2.3) — Añadida regla operativa: todo proyecto debe tener ErrorBoundary + useToast.
- `docs/project-setup.md` (v1.4) — Añadido `react-native-toast-message` a las dependencias de instalación (pnpm y npm).

---

## 2026-03 v1.4 — Imágenes optimizadas con `expo-image`

**Motivación:** Las listas con imágenes son un caso de uso muy frecuente. El `<Image>` nativo de React Native no tiene caché en disco, no soporta placeholders blur, y provoca parpadeo y jank en scroll. `expo-image` resuelve todos estos problemas sin coste adicional de configuración.

**Cambios:**
- `docs/conventions.md` (v1.2) — Añadida regla: usar `expo-image` en lugar de `<Image>` de React Native. Añadida prohibición explícita de `<Image>` nativo.
- `docs/templates-snippets.md` (v1.4) — Añadida sección completa "Imágenes optimizadas con `expo-image`": uso básico con `blurhash` + `contentFit` + `transition`, patrón en FlatList con `recyclingKey`, prefetch para UX premium, tabla de props principales, nota sobre generación de blurhash en backend. Actualizado comando CLI con `expo-image`.
- `docs/project-setup.md` (v1.3) — Añadido `expo-image` a las dependencias de instalación (pnpm y npm).

---

## 2026-03 v1.3 — AppStack: pantallas full-screen fuera de tabs

**Problema detectado:** La arquitectura v1.2 solo contemplaba pantallas dentro de AuthStack (no autenticado) o AppTabs (autenticado con tab bar). Faltaba soporte para pantallas autenticadas **sin tab bar** (ej: ProductDetail, Settings, EditProfile).

**Solución:** Se añade una capa `AppStack` (NativeStackNavigator) entre `RootNavigator` y `AppTabs`. `AppStack` contiene los tabs como primera screen (`MainTabs`) y permite registrar pantallas full-screen que se pushean encima de los tabs.

**Cambios:**
- `docs/navigation-patterns.md` (v1.3) — Añadido `AppStackParamList` con `NavigatorScreenParams` para tipar la relación con AppTabs. Añadido `AppStack.tsx` como nuevo archivo con plantilla completa. Actualizado `RootNavigator` para apuntar a `AppStack` (no a `AppTabs`). Añadido hook `useAppNavigation()`. Añadida tabla de decisión "¿Dónde va cada pantalla?" (tab vs full-screen vs sub-tab vs auth). Actualizados ejemplos con 4 opciones de uso (tabs, auth, full-screen con hook, full-screen con props). Actualizada sección "Anidar stacks dentro de un tab" con nota aclaratoria vs AppStack.
- `docs/templates-snippets.md` (v1.3) — Añadida plantilla de `AppStack.tsx`. Actualizado `navigation-types.ts` con `AppStackParamList` + `NavigatorScreenParams` + `AppStackScreenProps`. Actualizado `RootNavigator.tsx` para importar `AppStack` en lugar de `AppTabs`.
- `docs/structure-guide.md` (v2.1) — Actualizada estructura de `navigation/stacks/` con `AppStack.tsx`. Añadida nota sobre la jerarquía `RootNavigator → AppStack → AppTabs`. Actualizada tabla de referencia rápida.
- `docs/agent-instructions.md` (v2.2) — Actualizado PASO 5 con 7 pasos (antes 6), incluyendo creación de `AppStack.tsx`. Añadidas verificaciones: `NavigatorScreenParams`, `AppStack` como target de `RootNavigator`, `MainTabs` como primera screen. Actualizado "Flujo rápido" con guía de decisión (tab/full-screen/auth) para nuevas pantallas.

**Flujo de navegación documentado:**
```
RootNavigator (useAuthStore)
├── No autenticado → AuthStack
│   ├── Welcome   → botón "Continuar" → Login
│   ├── Login     → formulario + enlace a Register
│   └── Register  → formulario + enlace a Login
└── Autenticado → AppStack (NativeStack)
    ├── MainTabs → AppTabs (BottomTabNavigator)
    │   ├── Home      → pantalla principal
    │   └── Profile   → datos usuario + logout
    ├── ProductDetail  ← full-screen, sin tab bar
    ├── Settings       ← full-screen, sin tab bar
    └── EditProfile    ← full-screen, sin tab bar
```

---

## 2026-03 v1.2 — Pantallas base y navegación con tabs

**Cambios:**
- `docs/templates-snippets.md` (v1.2) — Añadidas 9 plantillas de pantallas base: Welcome (bienvenida con botón continuar), Login (formulario con useForm + useAuth), Register (formulario con validación de contraseñas), Home (pantalla base dentro de tabs), Profile (con botón logout). Añadidas plantillas de navegadores: AuthStack (Welcome → Login ↔ Register), AppTabs (BottomTabNavigator con Ionicons), RootNavigator (lógica condicional con useAuthStore). Actualizada plantilla de navigation-types.ts con AuthStackParamList, AppTabsParamList y RootStackParamList. Actualizada plantilla de App.tsx con QueryClientProvider y AppBootstrap para loadToken. Actualizados comandos CLI con @react-navigation/bottom-tabs, @tanstack/react-query, zustand y expo-secure-store.
- `docs/navigation-patterns.md` (v1.2) — Actualizada estructura de carpetas (AuthStack.tsx, AppTabs.tsx en lugar de AppStack.tsx). Actualizado navigation-types.ts como fuente de verdad con 3 ParamList (Root, Auth, AppTabs) + BottomTabScreenProps. Actualizado RootNavigator con lógica useAuthStore. Reemplazado AppStack por AuthStack + AppTabs. Añadida sección de cómo añadir tabs y anidar stacks dentro de tabs. Actualizados hooks de navegación (useAuthNavigation). Actualizados ejemplos de uso en pantallas.
- `docs/agent-instructions.md` (v2.1) — Actualizado PASO 5 (navegación) con AuthStack, AppTabs y verificaciones de bottom-tabs. Añadido nuevo PASO 6 "Crear pantallas base" con instrucciones y checklist de verificación (5 pantallas obligatorias, validaciones de formulario, SafeAreaView). Renumerados pasos 6-9 a 7-10.
- `docs/project-setup.md` (v1.2) — Añadidas dependencias: @react-navigation/bottom-tabs, @tanstack/react-query, zustand, expo-secure-store. Actualizado App.tsx con QueryClientProvider y AppBootstrap.
- `TO-DO.md` — Marcados como completados: "Añadir registro y login funcional" y "Crear pantallas base como registro, login, home y profile".

**Flujo de navegación documentado:**
```
RootNavigator (useAuthStore)
├── No autenticado → AuthStack
│   ├── Welcome   → botón "Continuar" → Login
│   ├── Login     → formulario + enlace a Register
│   └── Register  → formulario + enlace a Login
└── Autenticado → AppTabs (BottomTabNavigator)
    ├── Home      → pantalla principal
    └── Profile   → datos usuario + logout
```

---

## 2026-03 v1.1 — Migración completa desde REACT_NATIVE_STRUCTURE_GUIDE.md

**Cambios:**
- `README.md` — Reescrito como guía directiva para agentes IA con tabla de orden de lectura, arquitectura en 3 capas y flujo para nuevas features.
- `docs/agent-instructions.md` — Reescrito con flujo paso a paso (Pasos 1-9), verificaciones obligatorias por cada doc y flujo rápido para añadir screens.
- `docs/project-setup.md` — Expandido con `tsconfig.json` completo, `babel.config.js` con module-resolver, `global.d.ts`, `.env.example`, y verificación final.
- `docs/structure-guide.md` — Migración completa: arquitectura de 3 capas, árbol de carpetas, reglas por las 10 carpetas (`screens/ components/ navigation/ services/ hooks/ store/ types/ utils/ theme/ assets/`), referencia rápida, checklist de validación y "cuando romper las reglas".
- `docs/conventions.md` — Migración completa: tabla de nombres, reglas TS con ejemplos ✅/❌, reglas de archivos, imports permitidos/prohibidos, `tsconfig.json` paths, `babel.config.js`, anti-patrones.
- `docs/nativewind-theme.md` — Migración completa: instalación con `@nativewind/typescript`, `tailwind.config.js` con colores y espaciado, `TailwindProvider`, patrones correctos/incorrectos, clases condicionales, dark mode, spacing, excepciones (Animated).
- `docs/navigation-patterns.md` — Migración completa: estructura de carpetas, `navigation-types.ts` completo con `declare global`, hooks tipados, `RootNavigator`, `AppStack`, `index.ts` re-exports, patrones de uso en pantallas, anti-patrones, tabla de beneficios.
- `docs/services-and-api.md` — Migración completa: estructura, `api.ts` con Axios + interceptores + `parseApiError`, `auth.ts`, `logger.ts`, todas las reglas.
- `docs/hooks-and-state.md` — Migración completa: reglas, `useProducts`, `useForm<T>`, ejemplo Login screen, RTK store (authSlice + store.ts), Zustand alternativo, test de hook.
- `docs/templates-snippets.md` — Expandido: Button con variantes, `navigation-types.ts`, `api.ts`, `types/models.ts`, `types/index.ts`, `App.tsx` completo, `Home` screen base, comandos CLI.
- `docs/testing-ci.md` — Expandido: tabla de estrategia por capa, `jest.config.js` con path alias, estructura `tests/`, unit test de servicio, unit test de validators, integration test de hook, mock de service, GitHub Actions YAML completo con pnpm.

**Objetivo:** Los 11 docs reemplazan completamente a `REACT_NATIVE_STRUCTURE_GUIDE.md` (puede eliminarse).

---

## 2026-03 v1.0 — Versión inicial

- Creación del sistema de documentación con `README.md` + 11 docs en `docs/`.

