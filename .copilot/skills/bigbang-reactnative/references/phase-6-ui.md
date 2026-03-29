# Phase 6 — UI: Components, Screens & App.tsx

## Docs to Read

1. **`docs/templates-snippets.md`** — Read the ENTIRE file. It contains the complete code for:
   - `Button` component (Button.tsx + index.ts)
   - `ErrorBoundary` component (class component + index.ts)
   - `useToast` hook (already created in Phase 4, cross-reference)
   - All 5 base screens: Welcome, Login, Register, Home, Profile
   - `App.tsx` — Final version with all providers
   - `expo-image` usage pattern

---

## Execution Steps

### Step 6.1 — Create `src/components/Button/Button.tsx`

Copy from `docs/templates-snippets.md` section "Componente reutilizable — Button":
- `Variant` type: 'primary' | 'secondary' | 'danger'
- `ButtonProps` type with label, onPress, variant, disabled
- Uses `TouchableOpacity` with `className` (Nativewind)
- Variant colors mapped to Tailwind classes (bg-primary, bg-secondary, bg-danger)

### Step 6.2 — Create `src/components/Button/index.ts`

```typescript
export { Button } from './Button'
```

### Step 6.3 — Create `src/components/ErrorBoundary/ErrorBoundary.tsx`

Copy from `docs/templates-snippets.md` section "ErrorBoundary":
- **Class component** (this is the ONE exception where class components are allowed)
- Catches rendering errors with `getDerivedStateFromError` and `componentDidCatch`
- Shows fallback UI with "Algo salió mal" message and retry button
- Uses `Button` from `@/components/Button`

### Step 6.4 — Create `src/components/ErrorBoundary/index.ts`

```typescript
export { ErrorBoundary } from './ErrorBoundary'
```

### Step 6.5 — Create `src/screens/Welcome/index.tsx`

Copy from `docs/templates-snippets.md` section "Welcome — Pantalla de bienvenida":
- Uses `SafeAreaView` from `react-native-safe-area-context`
- Shows app title and description
- Button "Continuar" navigates to Login
- Uses typed navigation from `@/navigation`

### Step 6.6 — Create `src/screens/Login/index.tsx`

Copy from `docs/templates-snippets.md` section "Login — Inicio de sesión":
- Uses `useForm` from `@/hooks/useFormState` for form management
- Uses `useAuth` from `@/hooks/useAuth` for login
- Validates email with `isValidEmail` from `@/utils/validators`
- Validates password length >= 8
- Shows inline errors under fields
- **INCLUDES "Entrar como invitado" button** that calls `loginAsGuest()`
- Link to Register screen
- Uses `SafeAreaView`

### Step 6.7 — Create `src/screens/Register/index.tsx`

Copy from `docs/templates-snippets.md` section "Register — Registro":
- Uses `useForm` with fields: name, email, password, confirmPassword
- Uses `useAuth` for register
- Validates: name required, email valid, password >= 8, passwords match
- Shows inline errors under fields
- Link to Login screen
- Uses `SafeAreaView`

### Step 6.8 — Create `src/screens/Home/index.tsx`

Copy the **COMPLETE code** from `docs/templates-snippets.md` section "Home — Pantalla principal (dentro de tabs)". Do NOT simplify or create a placeholder — the template includes real mock data and a full layout:
- `RECOMMENDED` and `NEW_APARTMENTS` mock data arrays (hardcoded constants at the top)
- `SafeAreaView` + outer `ScrollView` (vertical)
- Header: avatar image, greeting with `user?.name` / `'Invitado'` (via `useAuth`), notifications icon
- Horizontal `ScrollView` for "Recomendados" with image cards showing title, location and price
- Vertical list "Nuevos apartamentos" with thumbnail + room count using `Ionicons`
- Uses `Image` from `react-native` (NOT `expo-image`) for Expo Go compatibility
- Uses `Ionicons` from `@expo/vector-icons`
- Uses `useAuth` from `@/hooks/useAuth` to read `user` and `isGuest`

### Step 6.9 — Create `src/screens/Profile/index.tsx`

Copy from `docs/templates-snippets.md` section "Profile — Perfil de usuario":
- Uses `useAuth` to get user data, isGuest flag, and logout function
- **IF guest mode (isGuest === true):** Show different UI with message "Eres un invitado" + buttons to create account or logout
- **IF authenticated:** Shows user name and email + "Cerrar sesión" button with `variant="danger"` calling `logout`
- Uses `SafeAreaView`

### Step 6.10 — Create `src/App.tsx`

Copy from `docs/templates-snippets.md` section "App.tsx — Punto de entrada":
Must include ALL of these providers/components in the correct nesting order:
1. `QueryClientProvider` (outermost data layer)
2. `ErrorBoundary`
3. `AppBootstrap` (loads token on mount)
4. `NavigationContainer`
5. `RootNavigator`
6. `<Toast />` (last child, renders on top)

**Nativewind v4 setup in App.tsx:**
- Import `../global.css` at the top of the file
- Do NOT add `TailwindProvider`

**AppBootstrap component:**
- Reads `useAuthStore((s) => s.loadToken)` 
- Calls `loadToken()` in `useEffect`
- Returns `null`

**QueryClient config:**
- `staleTime: 1000 * 60 * 5` (5 minutes)
- `retry: 2`

---

## Recovery Key Files

If these files exist, this phase was likely already completed:
- `src/screens/Home/index.tsx`
- `src/screens/Login/index.tsx`
- `src/components/Button/Button.tsx`
- `src/App.tsx`

---

## Verification Checklist

- [ ] `src/components/Button/Button.tsx` exists with variant support
- [ ] `src/components/Button/index.ts` re-exports Button
- [ ] `src/components/ErrorBoundary/ErrorBoundary.tsx` exists as class component
- [ ] `src/components/ErrorBoundary/index.ts` re-exports ErrorBoundary
- [ ] All 5 screens exist: Welcome, Login, Register, Home, Profile
- [ ] Welcome navigates to Login on "Continuar" press
- [ ] Login uses `useForm` + `useAuth` (NOT Axios directly)
- [ ] Login has link to Register
- [ ] Register uses `useForm` + `useAuth` (NOT Axios directly)
- [ ] Register validates passwords match
- [ ] Register has link to Login
- [ ] Profile has logout button calling `useAuth().logout`
- [ ] All screens use `SafeAreaView` from `react-native-safe-area-context` (except Home which uses ScrollView)
- [ ] NO screen calls Axios directly — all through hooks
- [ ] NO screen uses `Alert.alert()` — errors handled in hooks
- [ ] All styles use `className` (no `StyleSheet.create()`)
- [ ] `src/App.tsx` imports `../global.css`
- [ ] `src/App.tsx` has: QueryClientProvider + ErrorBoundary + NavigationContainer + RootNavigator + Toast
- [ ] `App.tsx` has `AppBootstrap` component that calls `loadToken()` on mount
- [ ] `<Toast />` is the last child (renders on top of everything)
