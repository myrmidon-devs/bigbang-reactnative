# Phase 1 — Bootstrap: Project Creation, Dependencies & Configuration

## Docs to Read

1. **`docs/project-setup.md`** — Read the ENTIRE file. It contains:
   - Step-by-step project creation commands
   - Exact `tsconfig.json` content
   - Exact dependency list (`pnpm add` commands)
   - Exact `tailwind.config.js` content
   - Exact `babel.config.js` content
   - Exact `global.d.ts` content
   - Exact `.env.example` content
   - Exact `src/App.tsx` skeleton (will be finalized in Phase 6)

2. **`docs/nativewind-theme.md`** — Read the "Configuración `tailwind.config.js`" section for the complete Tailwind config with custom colors and spacing.

---

## Execution Steps

### Step 1.1 — Create Expo project

> **REGLA CRÍTICA — Siempre en la raíz:**
> El proyecto se crea SIEMPRE en TARGET_DIR (raíz del workspace). NUNCA pasar PROJECT_NAME
> como argumento a `create-expo-app` — eso crearía un subdirectorio. Usar siempre el punto (`.`).
> Tras el scaffold, actualizar el campo `name` y `slug` de `app.json` con PROJECT_NAME.

```bash
# Navegar a TARGET_DIR (raíz del workspace) y crear el proyecto EN ESE DIRECTORIO
cd {TARGET_DIR}

# pnpm (recomendado)
pnpm create expo-app .

# npm (fallback)
npx create-expo-app .
```

A continuación, editar `app.json` para establecer el nombre real del proyecto:
- `"name"` → PROJECT_NAME
- `"slug"` → PROJECT_NAME

Then align the runtime to the stable SDK line documented in `docs/project-setup.md` before the first start. Do NOT leave the project on `expo@latest` or any `@latest` template line if Expo Go compatibility matters.

If already in the target directory and it's empty, execute directly.

### Step 1.2 — Migrate to TypeScript

- Rename `App.js` → `App.tsx` (if App.js exists)
- Create `tsconfig.json` with the exact content from `docs/project-setup.md` Step 2:
  - Must include `"@/*": ["src/*"]` in `paths`
  - Must include `@nativewind/typescript` plugin
  - Must extend `expo/tsconfig.base`

### Step 1.3 — Install ALL dependencies

Follow the install commands from `docs/project-setup.md` Step 3. Do NOT skip any package.

Use `expo install` for Expo ecosystem packages (it resolves the compatible version for the active SDK automatically).
Use the package manager (pnpm/npm) without version pins for non-Expo packages.

**Expo-managed (use `expo install`):**
- expo-secure-store, expo-status-bar, expo-image

**Production (no version pin needed):**
- nativewind, axios
- @react-navigation/native, @react-navigation/native-stack, @react-navigation/bottom-tabs
- react-native-screens, react-native-safe-area-context
- @tanstack/react-query, zustand, react-native-toast-message

**Dev dependencies (no version pin needed):**
- tailwindcss, @nativewind/typescript, babel-plugin-module-resolver, @types/react, @types/react-native

### Step 1.4 — Initialize Tailwind

```bash
pnpm dlx tailwindcss init
```

Then replace `tailwind.config.js` content with the exact config from `docs/project-setup.md` Step 4 (which matches `docs/nativewind-theme.md`). Must include:
- `content: ['./src/**/*.{js,jsx,ts,tsx}']`
- Custom colors: primary, secondary, success, error, warning, danger
- Custom spacing: xs, sm, md, lg, xl

### Step 1.5 — Configure Babel

Create `babel.config.js` with the exact content from `docs/project-setup.md` Step 5:
- `['babel-preset-expo', { jsxImportSource: 'nativewind' }]` in `presets`
- `nativewind/babel` in `presets`
- `module-resolver` with alias `'@': './src'`

### Step 1.6 — Configure Metro + global CSS

Create the files from `docs/project-setup.md`:
- `metro.config.js` with `withNativeWind(config, { input: './global.css' })`
- `global.css` with Tailwind directives

### Step 1.7 — Create `global.d.ts`

In project root (next to `package.json`):
```typescript
/// <reference types="nativewind/types" />
```

### Step 1.8 — Create folder structure

Create ALL subdirectories under `src/`:
```
src/screens/
src/components/
src/navigation/stacks/
src/services/
src/hooks/
src/store/
src/types/
src/utils/
src/assets/images/
src/assets/icons/
src/assets/fonts/
```

### Step 1.9 — Create `.env.example` and `.env`

Create `.env.example` with the content from `docs/project-setup.md` Step 8.
Copy to `.env`.
Ensure `.env` and `.env.local` are in `.gitignore`.

---

## Recovery Key Files

If these files exist, this phase was likely already completed:
- `tsconfig.json` (in project root)
- `babel.config.js` (in project root)
- `tailwind.config.js` (in project root)
- `src/` directory exists with subdirectories

---

## Verification Checklist

- [ ] Project directory exists with `package.json`
- [ ] `tsconfig.json` has `"@/*": ["src/*"]` in `paths`
- [ ] `tsconfig.json` has `@nativewind/typescript` in `plugins`
- [ ] `babel.config.js` has `['babel-preset-expo', { jsxImportSource: 'nativewind' }]` in `presets`
- [ ] `babel.config.js` has `nativewind/babel` in `presets`
- [ ] `babel.config.js` has `module-resolver` with `'@': './src'`
- [ ] `metro.config.js` exists with `withNativeWind`
- [ ] `global.css` exists with Tailwind directives
- [ ] `tailwind.config.js` defines colors: primary, secondary, success, error, warning, danger
- [ ] `tailwind.config.js` uses `nativewind/preset`
- [ ] `global.d.ts` exists with nativewind types reference
- [ ] `.env.example` exists with `EXPO_PUBLIC_API_URL`
- [ ] `src/` folder created with subcarpetas: screens, components, navigation/stacks, services, hooks, store, types, utils, assets/images, assets/icons, assets/fonts
- [ ] All production dependencies installed (check `package.json`)
- [ ] All dev dependencies installed (check `package.json`)
- [ ] `npx expo-doctor@latest` runs without SDK mismatches
- [ ] `npx tsc --noEmit` runs without errors (may have warnings about missing files — that's OK at this stage)
