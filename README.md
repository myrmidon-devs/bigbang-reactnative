<!-- README maestro — Guía de proyectos React Native -->

# Guía de proyectos React Native + Expo + TypeScript + Nativewind

---

## Stack obligatorio

| Tecnología                  | Rol                                                       |
| ---------------------------- | --------------------------------------------------------- |
| React Native + Expo          | Framework base                                            |
| TypeScript                   | Obligatorio en todos los archivos                         |
| Nativewind v4                | Estilos (Tailwind CSS) — no `StyleSheet.create()`      |
| Axios                        | Cliente HTTP — no `fetch` directo                      |
| React Navigation             | Navegación tipada                                        |
| TanStack Query (React Query) | Cache y estado de servidor (sustituye RxJS Subjects)      |
| Zustand                      | Estado global de sesión auth (sustituye BehaviorSubject) |
| expo-secure-store            | Almacenamiento seguro del token (no AsyncStorage)         |

**Gestor de paquetes:** `pnpm` preferido. Si no está disponible, preguntar al usuario si quiere instalarlo. Si rechaza, usar `npm`.

**Expo y Expo Go:** no usar `expo@latest` ni plantillas `@latest` de forma ciega cuando el objetivo es abrir la app en Expo Go. El bootstrap debe fijar una línea estable de SDK compatible con el cliente de Expo Go que se vaya a usar y validarla con `expo-doctor`.

---

## Instrucciones para el agente IA

**Lee primero `docs/agent-instructions.md`.** Ese archivo define el flujo completo paso a paso con verificaciones obligatorias por cada doc.

Orden de lectura y ejecución:

| # | Doc                             | Cuándo leer                             |
| - | ------------------------------- | ---------------------------------------- |
| 0 | `docs/agent-instructions.md`  | Antes de cualquier acción               |
| 1 | `docs/project-setup.md`       | Para crear el proyecto base              |
| 2 | `docs/structure-guide.md`     | Para entender la arquitectura            |
| 3 | `docs/conventions.md`         | Para aplicar convenciones de código     |
| 4 | `docs/nativewind-theme.md`    | Para configurar estilos                  |
| 5 | `docs/navigation-patterns.md` | Para crear navegación tipada            |
| 6 | `docs/templates-snippets.md`  | Para crear pantallas base y navegadores  |
| 7 | `docs/services-and-api.md`    | Para capa de servicios con Axios         |
| 8 | `docs/hooks-and-state.md`     | Para hooks y estado global               |
| 9 | `docs/testing-ci.md`          | Para configurar tests (opcional en MVP)  |

---

## Separación de documentación

Este repo contiene **dos capas de docs** que **nunca deben mezclarse**:

| Carpeta | Contenido | Quién la mantiene |
| --- | --- | --- |
| `docs/` | **Convenciones del stack** — cómo escribir el código, patrones, plantillas, arquitectura. Igual para todos los proyectos. | Equipo base (este repo) |
| `docs-project/` | **Especificaciones del proyecto concreto** — qué construir, pantallas propias, modelos de datos, endpoints reales. Única por proyecto. | El equipo de cada proyecto |

### Flujo de uso del template

```
1. Clonar este repo
2. Crear carpeta docs-project/ con los ficheros del proyecto concreto
   ├── brief.md      ← Qué hace la app, contexto del negocio
   ├── screens.md    ← Inventario de pantallas a construir
   ├── models.md     ← Entidades/tipos propios (User, Order, etc.)
   └── api.md        ← Endpoints del backend (rutas, payloads, auth)
3. Llamar al agente: "Crea el proyecto siguiendo los docs"
   → El agente lee docs-project/ (qué construir)
     + docs/ (cómo construirlo)
```

> `docs-project/` **no se incluye en este repo**. Cada proyecto lo crea desde cero. La carpeta está en `.gitignore` del template para evitar que documentación de un cliente se mezcle con las convenciones base.

---

## Arquitectura en 3 capas (Regla de Oro)

```
Screen (pintor)     →  Solo renderiza, consume hooks
      ↓
Hook (gerente)      →  Gestiona estado y llama servicios
                       useQuery / useMutation (React Query) para datos del servidor
                       useAuthStore (Zustand) para estado de sesión
      ↓
Service (obrero)    →  Llama a la API con Axios, puro TypeScript
                       Métodos genéricos CRUD + sub-entidades para rutas Laravel
```

**La screen nunca llama a Axios directamente.** Si necesitas datos, crea un hook. Si el hook necesita datos, crea una función en `services/api.ts`.

---

## Estructura de carpetas

```
src/
├── screens/          Pantallas (una carpeta por pantalla)
├── components/       Componentes usados en 2+ pantallas
├── navigation/       Tipado y stacks de React Navigation
│   └── stacks/
├── services/         API, auth, logger — TypeScript puro
├── hooks/            Custom hooks (puente Screen↔Service)
├── store/            Estado global (RTK o Zustand, uno solo)
├── types/            Tipos globales compartidos
├── utils/            Validators, formatters, constants
├── assets/           images/ icons/ fonts/
└── App.tsx           import de global.css + providers + NavigationContainer
```

---

## Flujo para añadir una feature nueva

```
1. Añadir tipo en src/types/models.ts
2. Añadir función en src/services/api.ts (usar métodos genéricos: getEntity, addSubEntity...)
3. Crear hook en src/hooks/useFeature.ts (con useQuery/useMutation si es dato de servidor)
4. Crear carpeta en src/screens/FeatureName/index.tsx
5. Registrar ruta en src/navigation/navigation-types.ts
6. Añadir screen al stack en src/navigation/stacks/
```

---

## Índice de documentos

- [docs/agent-instructions.md](docs/agent-instructions.md) — Flujo y verificaciones para agentes
- [docs/project-setup.md](docs/project-setup.md) — Bootstrap paso a paso
- [docs/structure-guide.md](docs/structure-guide.md) — Arquitectura y reglas por carpeta
- [docs/conventions.md](docs/conventions.md) — Naming, TypeScript, imports
- [docs/nativewind-theme.md](docs/nativewind-theme.md) — Estilos con Tailwind
- [docs/navigation-patterns.md](docs/navigation-patterns.md) — Navegación tipada
- [docs/services-and-api.md](docs/services-and-api.md) — Servicios con Axios
- [docs/hooks-and-state.md](docs/hooks-and-state.md) — Hooks y estado global
- [docs/templates-snippets.md](docs/templates-snippets.md) — Plantillas de archivos
- [docs/testing-ci.md](docs/testing-ci.md) — Tests y CI/CD
- [docs/changelog.md](docs/changelog.md) — Historial de cambios

### Para desarrolladores

- [docs/angular-to-react-native.md](docs/angular-to-react-native.md) — Guía de transición desde Ionic/Angular (lectura humana)

---

*Última revisión: Marzo 2026 · v1.2*

---

## Autor

Creado por **Julio Alberto Pancorbo Montoro** ([@juliocodex](https://github.com/JulioPancorbo)).

---

## ☕ Apóyame

Si este template te ha ahorrado horas de setup, puedes invitarme a un café. ¡Se agradece mucho!

[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20Me%20a%20Coffee-juliocodex-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/juliocodex)
