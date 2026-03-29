---
title: Testing & CI
version: 2.0
---

# Tests y CI

---

## Estrategia de tests

| Capa | Qué testear | Tipo |
|------|-------------|------|
| `services/` | Llamadas HTTP, parseo de errores, transformaciones | Unit test |
| `utils/` | Validadores, formateadores, helpers puros | Unit test |
| `store/` | Estado inicial, acciones, reset | Unit test |
| `hooks/` | Estados loading/error/data, efectos secundarios | Integration test |
| `components/` | Solo los que tienen lógica condicional o callbacks | Unit test |

**Reglas:**
- ✅ Unit tests para `services/`, `utils/` y `store/`
- ✅ Integration tests para `hooks/` con mocks de servicios
- ✅ E2E con Maestro para flujos críticos completos
- ✅ Mocks en `tests/__mocks__/`
- ❌ No testear detalles de implementación
- ❌ No hacer llamadas reales a la API en tests

---

## Qué NO testear (pérdida de tiempo)

### `src/components/` — componentes visuales puros
Si tienes un `<Button variant="primary">`, no pierdas tiempo escribiendo un test para comprobar si el color de fondo es azul. Si se rompe, lo ves a simple vista en el emulador. Testea solo los componentes que tienen **lógica condicional** (renderizado según props, interacciones que disparan callbacks).

### `src/screens/` — la capa visual
Las pantallas son “tontas”: solo consumen hooks. No testearlas unitariamente. Confiar en **Maestro (E2E)** para verificar que la pantalla entera funciona en conjunto.

### Librerías de terceros
No testees si el mapa de `react-native-maps` carga, ni si `react-navigation` cambia de pantalla correctamente. Los creadores de esas librerías ya han hecho esos tests.

---

## Instalación

```bash
# pnpm (recomendado)
pnpm add -D jest @testing-library/react-native @testing-library/jest-native @types/jest jest-expo

# npm fallback
npm install --save-dev jest @testing-library/react-native @testing-library/jest-native @types/jest jest-expo
```

---

## `jest.config.js`

```javascript
// jest.config.js
module.exports = {
  preset: 'jest-expo',
  setupFilesAfterEnv: ['@testing-library/jest-native/extend-expect'],
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@unimodules/.*|unimodules|sentry-expo|native-base|react-native-svg)',
  ],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/types/**',
    '!src/screens/**',          // cubierto por Maestro E2E
    '!src/components/**/*.tsx', // solo si son visuales puros
  ],
}
```

---

## Estructura de tests

```
tests/
├── __mocks__/
│   ├── expo-secure-store.ts
│   └── @/services/
│       └── api.ts
├── unit/
│   ├── services/
│   │   └── api.test.ts
│   ├── utils/
│   │   └── validators.test.ts
│   ├── store/
│   │   └── authStore.test.ts
│   └── components/
│       └── Button.test.tsx
└── integration/
    └── hooks/
        └── useProducts.test.ts

e2e/                          ← tests Maestro (fuera de tests/)
└── flows/
    ├── login.yaml
    ├── login-error.yaml
    └── product-list.yaml
```

---

## Ejemplos de tests

### Unit test — servicio

```typescript
// tests/unit/services/api.test.ts
import { parseApiError } from '@/services/api'
import axios from 'axios'

describe('parseApiError', () => {
  it('returns message from AxiosError', () => {
    const axiosError = new axios.AxiosError('Not found', undefined, undefined, undefined, {
      status: 404,
      data: {},
      headers: {},
      config: {} as any,
      statusText: 'Not Found',
    })
    expect(parseApiError(axiosError)).toEqual({ message: 'Not found', status: 404 })
  })

  it('returns string for unknown error', () => {
    expect(parseApiError(new Error('fail'))).toEqual({ message: 'Error: fail' })
  })
})
```

### Unit test — validadores

```typescript
// tests/unit/utils/validators.test.ts
import { isValidEmail, isValidPassword } from '@/utils/validators'

describe('isValidEmail', () => {
  it('validates correct emails', () => {
    expect(isValidEmail('user@example.com')).toBe(true)
  })

  it('rejects invalid emails', () => {
    expect(isValidEmail('not-an-email')).toBe(false)
    expect(isValidEmail('')).toBe(false)
  })
})

describe('isValidPassword', () => {
  it('requires at least 8 characters', () => {
    expect(isValidPassword('1234567')).toBe(false)
    expect(isValidPassword('12345678')).toBe(true)
  })
})
```

---

### Unit test — Zustand store

Los stores son código puro (sin React), por lo que son los más rápidos y simples de testear. Testear siempre: estado inicial, cada acción, y el reset.

```typescript
// tests/unit/store/authStore.test.ts
import { useAuthStore } from '@/store/authStore'

// Resetear el store entre tests para evitar contaminación
beforeEach(() => {
  useAuthStore.setState({ token: null, user: null })
})

describe('authStore', () => {
  it('starts unauthenticated', () => {
    const { token, user } = useAuthStore.getState()
    expect(token).toBeNull()
    expect(user).toBeNull()
  })

  it('sets token and user on login', () => {
    const mockUser = { id: '1', name: 'Ana', email: 'ana@example.com' }

    useAuthStore.getState().setAuth('jwt-token-abc', mockUser)

    const { token, user } = useAuthStore.getState()
    expect(token).toBe('jwt-token-abc')
    expect(user).toEqual(mockUser)
  })

  it('clears token and user on logout', () => {
    useAuthStore.setState({ token: 'jwt-token-abc', user: { id: '1', name: 'Ana', email: 'ana@example.com' } })

    useAuthStore.getState().clearAuth()

    const { token, user } = useAuthStore.getState()
    expect(token).toBeNull()
    expect(user).toBeNull()
  })
})
```

**Reglas:**
- Siempre llamar a `useAuthStore.setState(initialState)` en `beforeEach` para aislar tests.
- Acceder al store vía `useAuthStore.getState()` — nunca con `renderHook`, el store es puro.
- No mockear Zustand; testear la implementación real.

---

### Unit test — componente con lógica condicional

Solo testear componentes que tienen **lógica observable**: renderizado condicional, callbacks, estados de error/loading. No testear componentes que son markup estático.

```typescript
// tests/unit/components/Button.test.tsx
import React from 'react'
import { render, fireEvent } from '@testing-library/react-native'
import { Button } from '@/components/Button'

describe('Button', () => {
  it('renders label correctly', () => {
    const { getByText } = render(<Button label="Guardar" onPress={() => {}} />)
    expect(getByText('Guardar')).toBeTruthy()
  })

  it('calls onPress when tapped', () => {
    const onPress = jest.fn()
    const { getByText } = render(<Button label="Enviar" onPress={onPress} />)

    fireEvent.press(getByText('Enviar'))

    expect(onPress).toHaveBeenCalledTimes(1)
  })

  it('does not call onPress when disabled', () => {
    const onPress = jest.fn()
    const { getByText } = render(<Button label="Enviar" onPress={onPress} disabled />)

    fireEvent.press(getByText('Enviar'))

    expect(onPress).not.toHaveBeenCalled()
  })

  it('shows loading indicator and hides label when loading', () => {
    const { getByTestId, queryByText } = render(
      <Button label="Enviar" onPress={() => {}} loading />
    )

    expect(getByTestId('button-loading')).toBeTruthy()
    expect(queryByText('Enviar')).toBeNull()
  })
})
```

---

### Integration test — hook

```typescript
// tests/integration/hooks/useProducts.test.ts
import { renderHook, waitFor } from '@testing-library/react-native'
import { useProducts } from '@/hooks/useProducts'
import * as apiService from '@/services/api'

jest.mock('@/services/api')

const mockProducts = [
  { id: '1', name: 'Producto A', price: 10 },
  { id: '2', name: 'Producto B', price: 20 },
]

describe('useProducts', () => {
  it('loads products on mount', async () => {
    ;(apiService.getProducts as jest.Mock).mockResolvedValue(mockProducts)

    const { result } = renderHook(() => useProducts())

    expect(result.current.loading).toBe(true)

    await waitFor(() => {
      expect(result.current.loading).toBe(false)
    })

    expect(result.current.products).toEqual(mockProducts)
    expect(result.current.error).toBeNull()
  })

  it('sets error on failure', async () => {
    ;(apiService.getProducts as jest.Mock).mockRejectedValue(new Error('Network error'))

    const { result } = renderHook(() => useProducts())

    await waitFor(() => {
      expect(result.current.loading).toBe(false)
    })

    expect(result.current.error).toBe('Network error')
  })
})
```

### Mock de service

```typescript
// tests/__mocks__/@/services/api.ts
export const getProducts = jest.fn()
export const getUser = jest.fn()
export const parseApiError = jest.fn((err: unknown) => ({ message: String(err) }))
```

---

## E2E con Maestro

Maestro es la herramienta para tests de extremo a extremo en Expo. Usa sintaxis YAML, no requiere código nativo adicional, y funciona con Development Builds sobre simulador o dispositivo físico.

**Cuándo usarlo:** flujos críticos que involucran múltiples pantallas — login, checkout, formularios complejos, navegación entre tabs.

### Instalación

```bash
# macOS / Linux
curl -Ls "https://get.maestro.mobile.dev" | bash

# Windows (PowerShell)
iwr get.maestro.mobile.dev/windows -useb | iex

# Verificar instalación
maestro --version
```

### Estructura de carpetas

Los flows de Maestro viven fuera de `tests/` para separar E2E de los tests unitarios:

```
e2e/
└── flows/
    ├── login.yaml
    ├── login-error.yaml
    └── product-list.yaml
```

### Ejemplo — flujo de login exitoso

```yaml
# e2e/flows/login.yaml
appId: com.miempresa.miapp   # bundleIdentifier de app.json
---
- launchApp
- assertVisible: "Iniciar sesión"

- tapOn: "Email"
- inputText: "ana@example.com"

- tapOn: "Contraseña"
- inputText: "password123"

- tapOn: "Entrar"

- assertVisible: "Bienvenida, Ana"
- assertNotVisible: "Iniciar sesión"
```

### Ejemplo — flujo de login con error

```yaml
# e2e/flows/login-error.yaml
appId: com.miempresa.miapp
---
- launchApp
- tapOn: "Email"
- inputText: "usuario@invalido.com"

- tapOn: "Contraseña"
- inputText: "wrongpassword"

- tapOn: "Entrar"

- assertVisible: "Credenciales incorrectas"
```

### Ejemplo — listado y detalle de producto

```yaml
# e2e/flows/product-list.yaml
appId: com.miempresa.miapp
---
- launchApp
- tapOn: "Productos"
- assertVisible: "Producto A"
- scrollDown
- assertVisible: "Producto Z"
- tapOn: "Producto A"
- assertVisible: "Detalle del producto"
```

### Ejecutar tests

```bash
# Un flow concreto
maestro test e2e/flows/login.yaml

# Todos los flows
maestro test e2e/flows/

# Dispositivo específico
maestro --device <device-id> test e2e/flows/login.yaml
```

### Integración en CI

```yaml
# fragmento para añadir en bitbucket-pipelines.yml
- step:
    name: E2E Maestro
    script:
      - maestro test e2e/flows/ --format junit --output e2e-results.xml
    artifacts:
      - e2e-results.xml
```

**Reglas:**
- Usar texto visible en pantalla (`assertVisible`) — no selectores CSS ni IDs internos.
- Si necesitas seleccionar por ID en lugar de texto, añadir `testID` al componente: `<View testID="login-form">`.
- Maestro no sustituye los unit tests; cubre flujos completos, no lógica de negocio.
- No añadir lógica condicional compleja en los YAML — si un flow es muy largo, dividirlo en varios archivos.

---

## Pipeline CI

Elige la configuración según dónde esté alojado el repositorio.

### Opción A — Bitbucket Pipelines

```yaml
# bitbucket-pipelines.yml
image: node:20

pipelines:
  branches:
    main:
      - step:
          name: Type check, lint y tests
          caches:
            - node
          script:
            - npm install -g pnpm
            - pnpm install
            - pnpm exec tsc --noEmit
            - pnpm run lint
            - pnpm test -- --coverage --watchAll=false
          artifacts:
            - coverage/**

    develop:
      - step:
          name: Type check, lint y tests
          caches:
            - node
          script:
            - npm install -g pnpm
            - pnpm install
            - pnpm exec tsc --noEmit
            - pnpm run lint
            - pnpm test -- --coverage --watchAll=false
```

### Opción B — GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install pnpm
        run: npm install -g pnpm

      - name: Install dependencies
        run: pnpm install

      - name: Type check
        run: pnpm exec tsc --noEmit

      - name: Lint
        run: pnpm run lint

      - name: Test
        run: pnpm test -- --coverage --watchAll=false

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/
```

---

## Renovate — Actualización automática de dependencias

Renovate monitoriza el `package.json` del proyecto y abre PRs automáticamente cuando detecta versiones nuevas de las dependencias. Funciona con Bitbucket en modo self-hosted ejecutándose desde Bitbucket Pipelines.

**Flujo:**
1. Renovate escanea `package.json` según el schedule configurado
2. Detecta una versión nueva disponible en npm
3. Abre un PR con el cambio en `package.json` y `pnpm-lock.yaml`
4. El pipeline de CI corre los tests automáticamente sobre ese PR
5. Se revisa y mergea manualmente (o con automerge para patches)

### Archivos necesarios

```
renovate.json          ← configuración de Renovate (raíz del proyecto)
bitbucket-pipelines.yml ← ya existente, añadir el schedule de Renovate
```

### `renovate.json`

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "schedule": ["before 6am on monday"],
  "packageRules": [
    {
      "groupName": "Expo SDK",
      "matchPackagePrefixes": ["expo", "@expo/"],
      "automerge": false
    },
    {
      "groupName": "React Navigation",
      "matchPackagePrefixes": ["@react-navigation/"],
      "automerge": false
    },
    {
      "groupName": "TanStack Query",
      "matchPackageNames": ["@tanstack/react-query"],
      "automerge": false
    },
    {
      "groupName": "Dev dependencies",
      "matchDepTypes": ["devDependencies"],
      "matchUpdateTypes": ["patch", "minor"],
      "automerge": true
    }
  ]
}
```

> La agrupación es clave en proyectos Expo: sin ella, una actualización del SDK abriría ~20 PRs individuales (uno por cada paquete `expo-*`). Con los grupos definidos, se reducen a 3-4 PRs máximo por ciclo.

### Añadir el schedule en `bitbucket-pipelines.yml`

```yaml
# Añadir esta sección al bitbucket-pipelines.yml existente
custom:
  renovate:
    - step:
        name: Renovate dependency updates
        image: renovate/renovate:latest
        script:
          - renovate --platform bitbucket --token $BITBUCKET_TOKEN $BITBUCKET_REPO_FULL_NAME
```

La variable `BITBUCKET_TOKEN` debe configurarse en **Bitbucket → Repository settings → Repository variables** con un token de acceso personal con permisos de lectura/escritura sobre el repositorio.

### Reglas

- No mergear PRs de Renovate sin revisar el changelog de la librería, especialmente en actualizaciones de Expo SDK.
- Las actualizaciones de `devDependencies` en versiones patch/minor pueden tener `automerge: true` — son de bajo riesgo.
- Si un PR de Renovate falla en CI, no forzar el merge — investigar la incompatibilidad primero.
- Las actualizaciones de Expo SDK deben testearse en dispositivo físico además del simulador antes de mergear.

