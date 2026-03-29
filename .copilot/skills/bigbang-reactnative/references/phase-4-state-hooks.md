# Phase 4 — State & Hooks: Zustand Store + Custom Hooks

## Docs to Read

1. **`docs/hooks-and-state.md`** — Read the ENTIRE file. It contains the complete code for:
   - `authStore.ts` — Zustand store for auth session
   - `useAuth.ts` — Hook connecting authStore + auth service
   - `useFetch.ts` — Generic fetch hook (non-React-Query)
   - `useFormState.ts` — Generic form management hook with validation
   - Usage patterns and React Query setup

2. **`docs/templates-snippets.md`** — Read the section `useToast` for the toast notification hook.

---

## Execution Steps

### Step 4.1 — Create `src/store/authStore.ts`

Copy the COMPLETE Zustand store code from `docs/hooks-and-state.md` section "Zustand para auth":
- `AuthUser` type with id, name, email, role_id
- `AuthStore` type with: token, user, isGuest, isLoaded, setAuth, setAsGuest, clearAuth, loadToken
- `useAuthStore` created with `create<AuthStore>()`
- `setAsGuest()` method sets isGuest=true without token or user (for guest mode)
- `clearAuth()` resets token, user, and isGuest
- `loadToken()` reads from `getToken()` (from storage service) and sets `isLoaded: true`
- Must import `getToken` from `@/services/storage`

### Step 4.2 — Create `src/hooks/useAuth.ts`

Copy the COMPLETE code from `docs/hooks-and-state.md` section "useAuth":
- Imports from `@/store/authStore` and `@/services/auth`
- `login()`, `loginGoogle()`, `logout()` functions
- `loginAsGuest()` function for guest mode (no API call, just sets store state)
- `logout()` must call `queryClient.clear()` to wipe React Query cache
- Returns: token, user, isGuest, isAuthenticated, login, loginGoogle, register, loginAsGuest, logout

### Step 4.3 — Create `src/hooks/useFormState.ts`

Copy the COMPLETE code from `docs/hooks-and-state.md` section "useForm":
- Generic `useForm<T>()` hook
- Accepts `initialValues`, `onSubmit`, `validate` (optional)
- Returns: values, errors, loading, handleChange, handleSubmit
- `handleSubmit` runs validation first, then calls onSubmit

### Step 4.4 — Create `src/hooks/useFetch.ts`

Copy the COMPLETE code from `docs/hooks-and-state.md` section "useFetch":
- Generic `useFetch<T>()` hook
- Accepts a function returning `Promise<T>` and deps array
- Returns: data, isLoading, error
- Handles cleanup with `mounted` flag

### Step 4.5 — Create `src/hooks/useToast.ts`

Copy the COMPLETE code from `docs/templates-snippets.md` section "useToast":
- Uses `react-native-toast-message` under the hood
- `ToastType`: 'success' | 'error' | 'info'
- Returns: show, showError, showSuccess, showInfo
- All methods properly typed with explicit return types

---

## Recovery Key Files

If these files exist, this phase was likely already completed:
- `src/store/authStore.ts`
- `src/hooks/useAuth.ts`
- `src/hooks/useFormState.ts`
- `src/hooks/useToast.ts`

---

## Verification Checklist

- [ ] `src/store/authStore.ts` exists with Zustand store (NOT Redux)
- [ ] `authStore` has: token, user, isGuest, isLoaded, setAuth, setAsGuest, clearAuth, loadToken
- [ ] `loadToken()` reads from `@/services/storage` (not AsyncStorage directly)
- [ ] `src/hooks/useAuth.ts` exists, calls services from `@/services/auth` (NOT Axios directly)
- [ ] `useAuth().logout()` clears both authStore AND React Query cache
- [ ] `src/hooks/useFormState.ts` exists with generic useForm hook
- [ ] `src/hooks/useFetch.ts` exists with generic useFetch hook
- [ ] `src/hooks/useToast.ts` exists, wraps react-native-toast-message
- [ ] ALL hooks start with `use` prefix
- [ ] ALL hooks call services, NEVER Axios directly
- [ ] NO `console.log` in any hook
- [ ] ALL functions have explicit return types
- [ ] NO Redux or Redux Toolkit imports anywhere
