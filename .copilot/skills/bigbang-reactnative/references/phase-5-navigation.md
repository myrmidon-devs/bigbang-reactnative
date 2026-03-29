# Phase 5 — Navigation: Typed Navigation System

## Docs to Read

1. **`docs/navigation-patterns.md`** — Read the ENTIRE file. It contains the complete code for:
   - `navigation-types.ts` — All ParamLists, typed props, and `declare global`
   - `hooks.ts` — Typed navigation hooks (useRootNavigation, useAuthNavigation, useAppNavigation)
   - `RootNavigator.tsx` — Entry point with auth logic
   - `stacks/AuthStack.tsx` — Welcome, Login, Register
   - `stacks/AppStack.tsx` — NativeStack wrapping AppTabs + full-screen pages
   - `stacks/AppTabs.tsx` — BottomTabNavigator with Home, Profile
   - `index.ts` — Re-exports everything
   - Navigation hierarchy explanation

2. **`docs/templates-snippets.md`** — Read the sections:
   - `navigation-types.ts` — Same content as above (cross-reference)
   - Navigators base section (AuthStack, AppTabs, AppStack, RootNavigator, index.ts)

---

## Execution Steps

### Step 5.1 — Create `src/navigation/navigation-types.ts`

Copy the COMPLETE type definitions from `docs/navigation-patterns.md` section `navigation-types.ts`:

Must include ALL of these:
- `RootStackParamList` — Auth and App
- `AuthStackParamList` — Welcome, Login, Register
- `AppStackParamList` — MainTabs (with NavigatorScreenParams), commented-out full-screen pages
- `AppTabsParamList` — Home, Profile
- `RootScreenProps<T>`, `AuthScreenProps<T>`, `AppStackScreenProps<T>`, `AppTabsScreenProps<T>`
- `declare global { namespace ReactNavigation { interface RootParamList extends RootStackParamList {} } }`

**CRITICAL:** The `declare global` block MUST exist for useNavigation autocompletion to work.
**CRITICAL:** `AppStackParamList` MUST use `NavigatorScreenParams<AppTabsParamList>` for the MainTabs route.

### Step 5.2 — Create `src/navigation/hooks.ts`

Copy from `docs/navigation-patterns.md` section `hooks.ts`:
- `useRootNavigation()` — typed for RootStackParamList
- `useAuthNavigation()` — typed for AuthStackParamList
- `useAppNavigation()` — typed for AppStackParamList

All must import types from `./navigation-types` (relative within navigation is OK).

### Step 5.3 — Create `src/navigation/RootNavigator.tsx`

Copy from `docs/navigation-patterns.md` section `RootNavigator.tsx`:
- Creates `NativeStackNavigator<RootStackParamList>`
- Reads `useAuthStore()` for `token`, `isGuest`, and `isLoaded`
- Returns `null` while `!isLoaded`
- Shows `AuthStack` when neither token nor guest mode is active, `AppStack` when `token || isGuest`
- `screenOptions={{ headerShown: false }}`

**CRITICAL:** RootNavigator points to AppStack, NOT to AppTabs directly.

### Step 5.4 — Create `src/navigation/stacks/AuthStack.tsx`

Copy from `docs/navigation-patterns.md` / `docs/templates-snippets.md`:
- `NativeStackNavigator<AuthStackParamList>`
- Screens: Welcome, Login, Register
- `headerShown: false`
- Imports screens from `@/screens/Welcome`, `@/screens/Login`, `@/screens/Register`

### Step 5.5 — Create `src/navigation/stacks/AppStack.tsx`

Copy from `docs/navigation-patterns.md` / `docs/templates-snippets.md`:
- `NativeStackNavigator<AppStackParamList>`
- First screen: `MainTabs` → component={AppTabs}
- Full-screen pages commented out (ProductDetail, Settings, EditProfile)
- `headerShown: false`

### Step 5.6 — Create `src/navigation/stacks/AppTabs.tsx`

Copy from `docs/navigation-patterns.md` / `docs/templates-snippets.md`:
- `BottomTabNavigator<AppTabsParamList>`
- Tabs: Home, Profile
- Uses `Ionicons` from `@expo/vector-icons` for tab icons
- `tabBarActiveTintColor: '#007AFF'`

### Step 5.7 — Create `src/navigation/index.ts`

```typescript
export { RootNavigator } from './RootNavigator'
export * from './navigation-types'
export * from './hooks'
```

---

## Recovery Key Files

If these files exist, this phase was likely already completed:
- `src/navigation/navigation-types.ts`
- `src/navigation/RootNavigator.tsx`
- `src/navigation/stacks/AppTabs.tsx`

---

## Verification Checklist

- [ ] `src/navigation/navigation-types.ts` exists with ALL 4 ParamLists
- [ ] `declare global { namespace ReactNavigation { ... } }` exists in navigation-types.ts
- [ ] `NavigatorScreenParams` is used to connect AppStack → AppTabs
- [ ] `src/navigation/hooks.ts` exists with 3 typed hooks
- [ ] `src/navigation/RootNavigator.tsx` uses `useAuthStore` to decide Auth vs App
- [ ] `RootNavigator` supports guest mode with `token || isGuest`
- [ ] `RootNavigator` points to `AppStack` (NOT AppTabs directly)
- [ ] `src/navigation/stacks/AuthStack.tsx` has Welcome, Login, Register
- [ ] `src/navigation/stacks/AppStack.tsx` has MainTabs as first screen
- [ ] `src/navigation/stacks/AppTabs.tsx` has Home, Profile with Ionicons
- [ ] `src/navigation/index.ts` re-exports RootNavigator, types, and hooks
- [ ] ALL screen imports use `@/screens/` path alias
- [ ] Navigation hierarchy: Root → AppStack → AppTabs (3 levels)
- [ ] No duplicate route definitions across files
- [ ] No relative imports (`../`) from screens to navigation (all use `@/navigation`)
