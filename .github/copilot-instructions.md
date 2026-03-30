# Copilot Instructions — React Native + Expo + TypeScript + Nativewind

Este workspace es un **template base** para proyectos React Native.
Antes de cualquier acción, lee `docs/agent-instructions.md`. Define el flujo completo con verificaciones obligatorias.

---

## Reglas no negociables

- **Estilos:** Nativewind v4 siempre. Nunca `StyleSheet.create()`. Nunca estilos inline en píxeles fijos.
- **Expo:** Nunca instalar `expo@latest` ni usar plantillas `@latest` sin fijar una línea estable de SDK compatible con Expo Go. Verifica con `expo-doctor` antes del primer arranque.
- **Bootstrap:** Si se crea el proyecto desde este template, se genera **siempre en la raíz del workspace** con `create-expo-app .`. Nunca crear un subdirectorio adicional. El nombre del proyecto se deriva de la carpeta raíz y se refleja en `app.json` (`name` y `slug`).
- **Arquitectura 3 capas:** `Screen → Hook → Service`. La screen nunca llama a Axios directamente.
- **Estado global:** Zustand para sesión auth. React Query (`useQuery`/`useMutation`) para datos del servidor. Sin Redux.
- **HTTP:** Axios desde `src/services/api.ts`. Nunca `fetch` directo.
- **Imports:** Siempre alias `@/`. Nunca rutas relativas `../`. Orden: React → librerías → locales.
- **TypeScript:** Sin `any`. Todas las funciones con tipo de retorno explícito.
- **Logs:** Nunca `console.log`. Usar `src/services/logger.ts`.
- **Errores de API:** Siempre en el hook (`onError`). Nunca en la screen. Nunca `Alert.alert()` directo. Todo proyecto debe tener `useToast` + `<Toast />` y `ErrorBoundary` en `App.tsx`.
- **Gestor de paquetes:** `pnpm` preferido. Si no está disponible, preguntar antes de usar `npm`.
- **Componentes:** Máximo 300 líneas por archivo. Sin archivos `.styles.ts`.
- **App shell:** `App.tsx` debe usar `SafeAreaProvider` como provider global, `QueryClientProvider`, `ErrorBoundary`, `NavigationContainer`, `RootNavigator` y `<Toast />` como último hijo. En desarrollo, `verifyInstallation()` de Nativewind va dentro de `if (__DEV__)`.
- **SafeAreaView:** Todas las screens deben estar envueltas en `<SafeAreaView>` para respetar el status bar, notch, y dynamic island. `SafeAreaProvider` está en `App.tsx` (global).
- **Guest mode:** El template soporta "Entrar como invitado" desde Login. Usar `useAuth().loginAsGuest()` para acceder a tabs sin autenticación. Screens pueden checar `isGuest` para adaptar contenido (ej: Profile muestra UI diferente para invitados).

---

## Doc a leer según la tarea

| Tarea | Doc obligatorio |
|---|---|
| Crear proyecto desde cero | `docs/agent-instructions.md` (flujo completo PASO 1–10) |
| Nueva pantalla o feature | `docs/agent-instructions.md` (sección "Añadir una nueva screen") |
| Estilos / Tailwind | `docs/nativewind-theme.md` |
| Navegación / rutas | `docs/navigation-patterns.md` |
| Servicios / Axios / API | `docs/services-and-api.md` |
| Hooks / React Query / Zustand | `docs/hooks-and-state.md` |
| Plantillas de archivos | `docs/templates-snippets.md` |
| Iconos o animaciones | `docs/animations-and-icons.md` |
| Plugin nativo (cámara, mapas, PDF, etc.) | `docs/native-plugins.md` — leer **antes** de instalar |
| Naming, TypeScript, imports | `docs/conventions.md` |
| Tests | `docs/testing-ci.md` |

---

## Flujo para añadir una feature nueva

```
1. Añadir tipo en src/types/models.ts
2. Añadir función en src/services/api.ts
3. Crear hook en src/hooks/useFeature.ts (useQuery/useMutation si es dato de servidor)
4. Crear carpeta src/screens/FeatureName/index.tsx  ← usar plantilla de templates-snippets.md
5. Registrar ruta en src/navigation/navigation-types.ts
6. Añadir screen al stack en src/navigation/stacks/
```

---

## Separación de docs

| Carpeta | Contenido |
|---|---|
| `docs/` | Convenciones del stack — iguales para todos los proyectos |
| `docs-project/` | Especificaciones del proyecto concreto (no incluida en el template) |

Cuando exista `docs-project/`, leer primero sus archivos (`brief.md`, `screens.md`, `models.md`, `api.md`) para saber **qué** construir, y luego `docs/` para saber **cómo** construirlo.

---

## Ante la duda

- Si algo no está cubierto en los docs → preguntar al usuario antes de inventar convenciones.
- Si identificas una mejora → anotarla en `docs/changelog.md`. No modificar otros docs sin consenso.
