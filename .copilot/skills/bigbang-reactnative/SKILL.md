---
name: bigbang-reactnative
description: Create a complete React Native + Expo + TypeScript + Nativewind project from scratch. Use when asked to "create react native project", "scaffold expo app", "bigbang", "bootstrap react native", "iniciar proyecto", "crear proyecto react native", "setup expo project", "generate base project", or "new mobile app". Executes all setup phases automatically including dependencies, navigation, services, state management, screens, and verification.
author: Julio Alberto Pancorbo Montoro (@juliocodex) — https://github.com/JulioPancorbo
---

# BigBang React Native — Full Project Scaffolding Skill

This skill automates the creation of a complete React Native + Expo + TypeScript + Nativewind project. The main agent acts as **orchestrator only** — each phase is delegated to an **isolated subagent** that reads the docs and creates the files independently.

---

## Architecture — Orchestrator + Subagents

**CRITICAL: The main agent MUST NOT execute phases directly.** The main agent's ONLY job is:
1. Run pre-flight checks (package manager, derive project name from workspace root)
2. Create the progress tracker
3. Dispatch one subagent per phase, sequentially
4. Read each subagent's result (PASS/FAIL)
5. Update progress tracker after each phase
6. Report final result to user, including the mandatory next steps from Step 4

```
Main Agent (Orchestrator) — reads 0 docs, creates 0 project files
│
├─ Pre-flight: verify pnpm, derive project name from workspace root, create progress tracker
│
├─ Subagent → Phase 1: Bootstrap
│    Reads: phase-1-bootstrap.md → project-setup.md, nativewind-theme.md
│    Creates: project dir, tsconfig, babel, tailwind, deps, folder structure
│    Reports: PASS or FAIL + reason
│
├─ Subagent → Phase 2: Services
│    Reads: phase-2-services.md → services-and-api.md
│    Creates: api.ts, auth.ts, storage.ts, logger.ts
│    Reports: PASS or FAIL + reason
│
├─ Subagent → Phase 3: Types & Utils
│    Reads: phase-3-types-utils.md → templates-snippets.md, structure-guide.md
│    Creates: types/, utils/ files
│    Reports: PASS or FAIL + reason
│
├─ Subagent → Phase 4: State & Hooks
│    Reads: phase-4-state-hooks.md → hooks-and-state.md
│    Creates: authStore.ts, useAuth.ts, useFormState.ts, useFetch.ts, useToast.ts
│    Reports: PASS or FAIL + reason
│
├─ Subagent → Phase 5: Navigation
│    Reads: phase-5-navigation.md → navigation-patterns.md, templates-snippets.md
│    Creates: navigation-types.ts, hooks.ts, RootNavigator.tsx, stacks/
│    Reports: PASS or FAIL + reason
│
├─ Subagent → Phase 6: UI
│    Reads: phase-6-ui.md → templates-snippets.md
│    Creates: Button, ErrorBoundary, 5 screens, App.tsx
│    Reports: PASS or FAIL + reason
│
└─ Subagent → Phase 7: Verification
     Reads: phase-7-verification.md → structure-guide.md, testing-ci.md
     Runs: tsc --noEmit, verify-structure.js, creates test example
     Reports: Full checklist + tsc result + anti-pattern scan
```

### Why subagents?

- Reading 10 docs + creating 40+ files sequentially **fills the main context window by phase 3–4**.
- Each subagent starts with a **clean context**: reads only ITS reference file + ITS docs → creates ITS files → reports back.
- The progress tracker (`/memories/session/bigbang-progress.md`) is the **single shared state** between phases.
- If a subagent fails, the main agent can **re-dispatch with failure context** without carrying accumulated baggage.

### Execution Discipline — Mandatory

- Follow the skill and the referenced docs **exactly as written**.
- Execute steps **one by one, in order**. Do not skip, merge, reorder, or compress steps.
- Verify each step against its checklist before moving to the next one.
- Do **not** assume, infer, or invent missing files, conventions, dependencies, or implementation details.
- If a required instruction is missing or ambiguous, **fail clearly** and surface the exact gap instead of improvising.
- A phase is not complete until its verification checklist passes.
- The whole workflow is not complete until the user receives both the completion summary **and** the mandatory next steps from Step 4.

---

## Prerequisites

- **This skill lives inside the React Native template repo** at `.copilot/skills/bigbang-reactnative/`. The workspace root IS the template repo — `docs/` and the skill's `references/` folder are always in sync via git.
- Node.js LTS (v20+)
- Git initialized in the target project

---

## Step 1 — Pre-flight Checks (Main Agent executes directly)

### 1.1 Verify package manager (Regla 0)

```
Run: pnpm --version
- If available → use pnpm for ALL commands (store as PKG_MANAGER=pnpm)
- If NOT available → ask user: "pnpm no está instalado. ¿Quieres instalarlo con npm install -g pnpm? Si no, usaré npm."
- If user declines → store as PKG_MANAGER=npm
```

### 1.2 Derive project name — do NOT ask the user

```
TARGET_DIR = workspace root (current directory).
PROJECT_NAME = name of the workspace root folder, converted to kebab-case.
  → e.g. workspace folder "MyAwesomeApp" → PROJECT_NAME = "my-awesome-app"

NEVER ask the user for a project name or a target directory.
NEVER run `create-expo-app {PROJECT_NAME}` — that creates a subdirectory.
ALWAYS run `create-expo-app .` inside TARGET_DIR.
PROJECT_NAME is only used to populate `app.json` fields (`name` and `slug`).
```

### 1.3 Initialize progress tracker

Create a session memory file:

```
Create /memories/session/bigbang-progress.md with:

# BigBang Progress — {PROJECT_NAME}
- Target: {TARGET_DIR}
- Package manager: {PKG_MANAGER}
- [ ] Phase 1: Bootstrap (project, deps, configs, folder structure)
- [ ] Phase 2: Services (api.ts, auth.ts, storage.ts, logger.ts)
- [ ] Phase 3: Types & Utils (models, validators, formatters, constants)
- [ ] Phase 4: State & Hooks (authStore, useAuth, useForm, useFetch, useToast)
- [ ] Phase 5: Navigation (types, hooks, RootNavigator, stacks)
- [ ] Phase 6: UI (Button, ErrorBoundary, 5 screens, App.tsx)
- [ ] Phase 7: Verification (checklist, tsc, anti-patterns, test example)
```

---

## Step 2 — Dispatch Phase Subagents (Main Agent orchestrates)

For each phase 1 through 7, the main agent dispatches a subagent using `runSubagent` with the following prompt template. **Do NOT read docs or create files yourself — the subagent does it.**

### Subagent Prompt Template

Use this EXACT template for each phase, replacing the variables:

```
You are executing Phase {N} of the bigbang-reactnative project scaffolding.

CONTEXT:
- Project name: {PROJECT_NAME}
- Target directory: {TARGET_DIR}
- Package manager: {PKG_MANAGER}
- Workspace root (template repo): {WORKSPACE_ROOT}

YOUR TASK:
1. Read the reference file at: {WORKSPACE_ROOT}/.copilot/skills/bigbang-reactnative/references/phase-{N}-{name}.md
2. The reference file lists which docs/ files to read. Read ALL of them from {WORKSPACE_ROOT}/docs/
3. Execute EVERY step in the reference file in the exact order written. Do NOT skip, merge, reorder, or simplify steps.
4. After EACH step, confirm that the expected output exists and is complete before moving on.
5. Create only the files, code, and configuration explicitly required by the reference file and docs in {TARGET_DIR}.
6. Run the verification checklist at the end of the reference file.
7. If any check fails, fix it immediately and re-verify.
8. If any instruction is missing, contradictory, or ambiguous, do NOT guess or invent a solution. Stop and report the exact gap as a failure.

CONVENTIONS (non-negotiable — apply to EVERY file you create):
- NEVER use `any` type — use `unknown` + narrowing
- NEVER use `StyleSheet.create()` — only className with Nativewind
- NEVER use `console.log` — use logger from services/logger.ts
- NEVER call Axios from screens — screens call hooks, hooks call services
- ALWAYS use `@/` import alias — NEVER relative paths `../`
- ALWAYS use `type` (not `interface`) for type definitions
- ALL functions must have explicit return types
- Zustand for auth state, React Query for server data, NEVER Redux
- expo-secure-store for tokens, NEVER AsyncStorage
- Max 300 lines per screen, 150 per component
- NEVER run `create-expo-app {PROJECT_NAME}` — this creates a subdirectory. ALWAYS run `create-expo-app .` inside TARGET_DIR. PROJECT_NAME is derived from the workspace folder name and only goes in app.json.

REPORT BACK with exactly one of:
- "PHASE {N} COMPLETE — all {X} files created, all checks passed"
- "PHASE {N} FAILED — {specific failure reason}"

Include a brief list of files created.
```

### Phase dispatch order and reference file paths

| Phase | Reference File | Subagent Description |
|-------|---------------|---------------------|
| 1 | `references/phase-1-bootstrap.md` | "Execute Phase 1 Bootstrap" |
| 2 | `references/phase-2-services.md` | "Execute Phase 2 Services" |
| 3 | `references/phase-3-types-utils.md` | "Execute Phase 3 Types Utils" |
| 4 | `references/phase-4-state-hooks.md` | "Execute Phase 4 State Hooks" |
| 5 | `references/phase-5-navigation.md` | "Execute Phase 5 Navigation" |
| 6 | `references/phase-6-ui.md` | "Execute Phase 6 UI" |
| 7 | `references/phase-7-verification.md` | "Execute Phase 7 Verification" |

### After each subagent returns

```
IF result contains "COMPLETE":
   → Confirm the phase explicitly reported that all checks passed
  → Update /memories/session/bigbang-progress.md: mark phase [x]
  → Proceed to next phase

IF result contains "FAILED":
  → Read the failure reason
  → Re-dispatch the SAME phase with additional context:
    "Previous attempt failed because: {failure_reason}.
     The project already exists at {TARGET_DIR}.
     Check which files from this phase already exist. Create only missing ones.
     Re-read the reference file and the specific doc section that failed.
       Do not invent missing conventions or unstated behavior.
     Fix and retry. Report PASS or FAIL."
  → If second attempt also fails → STOP and report to user:
    "La fase {N} falló después de 2 intentos. Error: {reason}.
     Revisa manualmente {TARGET_DIR} y los docs correspondientes."
```

---

## Step 3 — Recovery from Interrupted Execution

If the skill is invoked but a previous run was interrupted:

1. Check if `/memories/session/bigbang-progress.md` exists
2. If it exists → read it to find TARGET_DIR, PKG_MANAGER, and first uncompleted phase (`[ ]`)
3. Ask user: "Se encontró un progreso previo para {PROJECT_NAME}. ¿Retomar desde Fase {N}?"
4. If yes → skip pre-flight, dispatch subagent for that phase with:
   ```
   "The project already exists at {TARGET_DIR}. This is a RESUME from an interrupted run.
    Check which files from this phase already exist. Create only missing ones.
    Run the full verification checklist. Report PASS or FAIL."
   ```
5. Continue the dispatch loop from there

---

## Conventions — Non-Negotiable Rules

These rules are embedded in every subagent prompt above. They also serve as a quick reference for the main agent during verification.

### TypeScript
- **NEVER** use `any` — use `unknown` + type narrowing if needed
- **ALL** functions must have explicit return types
- Use `type` (not `interface`) for type definitions
- All files must be `.ts` or `.tsx`

### Styles
- **NEVER** use `StyleSheet.create()` — only `className` with Nativewind
- **NEVER** create `.styles.ts` files
- **NEVER** use fixed pixel widths for containers
- All layouts must be responsive (Flexbox, fractions, breakpoints)

### Expo & NativeWind
- **NEVER** bootstrap the project around `expo@latest` or an Expo template `@latest` when Expo Go compatibility matters
- **ALWAYS** align the project to the stable Expo SDK line documented in `docs/project-setup.md` before the first run
- **ALWAYS** validate the SDK line with `npx expo-doctor@latest` before starting the app for the first time
- **ALWAYS** use the validated NativeWind v4 setup for this template: Tailwind preset + Babel preset config + Metro `withNativeWind` + `global.css`
- **NEVER** add `TailwindProvider` in `App.tsx` for this template baseline

### Architecture (3 Layers)
- **Screen** → only renders, consumes hooks. NEVER calls Axios directly
- **Hook** → manages state, calls services. Bridge between Screen and Service
- **Service** → pure TypeScript, HTTP calls with Axios. No React dependencies

### Imports
- **ALWAYS** use `@/` path alias — NEVER relative paths `../`
- Order: React → third-party libs → `@/` local imports
- No circular imports

### Logging & Errors
- **NEVER** use `console.log` — use `src/services/logger.ts`
- **NEVER** use `Alert.alert()` directly — use `useToast` hook
- API errors handled in hooks (`onError`), NEVER in screens
- `ErrorBoundary` wraps the app in `App.tsx`

### Components
- Max 300 lines per screen file, 150 per component file
- One component = one folder with `index.ts` re-export
- Use `expo-image` instead of `<Image>` from react-native

### State Management
- **Zustand** for auth session state (token, user)
- **React Query** for server data (entities, lists)
- **NEVER** use Redux Toolkit
- **NEVER** store entity lists in Zustand (that goes in React Query cache)

### Storage
- **ALWAYS** use `expo-secure-store` for tokens — NEVER AsyncStorage

---

## Step 4 — Post-Completion (Main Agent)

After all 7 phases report COMPLETE:

1. Confirm `/memories/session/bigbang-progress.md` has all phases marked `[x]`
2. Report final status to the user:
   - Number of phases completed (should be 7/7)
   - Phase 7 verification results (tsc, anti-patterns)
   - Any warnings from subagents
3. The final user-facing message is **NOT complete** if it only contains a summary. It MUST also include the next steps below.
4. Suggest next steps:
   ```
   Base project created successfully. Next steps:
   1. Test the base project: run `pnpm start` (or `npm start` if pnpm is not available)
      and verify that the app starts without errors on the simulator/device.
   2. Configure `.env` with the real URL of your Laravel API:
      - Local development: EXPO_PUBLIC_API_URL=http://localhost:8000/api
      - Production:        EXPO_PUBLIC_API_URL=https://your-domain.com/api
   3. Create a `docs-project/` folder with the concrete project documents
      (screens, models, endpoints, brief, and any project-specific specification)
      and/or start building and designing screens directly.
      The `.github/copilot-instructions.md` file ensures the agent follows
      all stack rules and best practices on every action.
   ```

---

## Author

Created by **Julio Alberto Pancorbo Montoro** ([@juliocodex](https://github.com/JulioPancorbo)).