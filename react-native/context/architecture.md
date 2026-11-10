# Architecture Context

HARD CONTRACT: file placement, stack choices, and boundary rules here are
non-negotiable for Builders — QA rejects violations. Derived from the `react-native`
(Expo) reference template; keep it locked.

## Stack

| Layer | Technology | Role |
| --- | --- | --- |
| Runtime | React Native 0.85 (Hermes) on iOS / Android, `react-native-web` on web | app runtime |
| Framework | Expo SDK 56 (managed workflow) | tooling, native APIs, build |
| Language | TypeScript 6 — `strict: true`, `expo/tsconfig.base` | all source |
| Navigation | Expo Router 56 (file-based, `typedRoutes`) | routing + deep links |
| Server state | TanStack Query 5 | data fetching, cache |
| Client state | Zustand 5 (+ `createSelectors` helper) | auth + UI state |
| Styling | NativeWind 5 (Tailwind v4 tokens) + `class-variance-authority` | tokens-only styling |
| HTTP | typed `fetch` client (`shared/libs/api.ts`) over the response envelope | API access |
| Secure storage | `expo-secure-store` (Keychain / Keystore); `localStorage` on web | token persistence |
| Validation | Zod 4 | form + input validation |
| i18n | `i18next` + `react-i18next` + `expo-localization` (en / ru / uz) | localization |
| Icons | `lucide-react-native` (central `Icons` map) | iconography |
| Images | `expo-image` | image rendering |
| Gestures / animation | `react-native-gesture-handler`, `react-native-reanimated` 4, `react-native-worklets` | interaction |
| Quality | ESLint (`eslint-config-expo`) + Prettier, Husky + lint-staged | lint / format / hooks |

Package manager: a `pnpm-lock.yaml` is present (pnpm is canonical for this template).
React Compiler is enabled (`experiments.reactCompiler`) — do not hand-add `useMemo`/
`useCallback` purely for referential stability.

## Commands

| Action | Command |
| --- | --- |
| Start (Metro / dev menu) | `pnpm start` (`expo start`) |
| Run on iOS | `pnpm ios` (`expo start --ios`) |
| Run on Android | `pnpm android` (`expo start --android`) |
| Run on web | `pnpm web` (`expo start --web`) |
| Lint (native gate) | `pnpm lint` (`expo lint`) |
| Typecheck | `pnpm exec tsc --noEmit` |
| Format | `pnpm format` (`prettier --write`) |
| Format check | `pnpm format:check` |

QA and Builders run exactly these — keep them accurate. There is no test runner wired
yet; add one (and the command here) when tests are introduced.

## Folder structure

TypeScript path aliases (tsconfig): `@/*` → `src/*`, `@/assets/*` → `assets/*`.
Routing is the `src/app/` tree (Expo Router); product code lives in `src/features/`
and `src/shared/`.

```
src/
├── global.css                 ← Tailwind v4 + NativeWind theme tokens (single source)
├── app/                       ← Expo Router file-based routes (the navigation tree)
│   ├── _layout.tsx            ← root Stack; mounts <Providers>; imports global.css
│   ├── index.tsx              ← entry redirect (authed → dashboard / guest → home)
│   ├── (public)/              ← unauthenticated marketing/home group
│   ├── (auth)/                ← sign-in/up, forgot/reset, verify-email, two-factor (guarded: redirect if authed)
│   └── (protected)/           ← Tabs navigator (guarded: redirect to /sign-in if not authed)
├── layout/                    ← shared screen shells (auth-layout, coming-soon)
├── features/<name>/           ← one folder per domain feature (see Feature shape)
└── shared/
    ├── components/
    │   ├── ui/                ← primitives: button, input, field, label, card, text, checkbox, toaster
    │   ├── provider/          ← Providers, query-provider, i18n-provider
    │   └── widgets/           ← icons (lucide map), locale-switch
    ├── hooks/                 ← cross-feature hooks (useToast)
    ├── i18n/                  ← index, detector, routing, messages/{en,ru,uz}.json
    └── libs/                  ← api (typed client), storage (secure), store (zustand), utils (cn), validators

test/                          ← mirrors src/ (see Test structure)
app.json                       ← Expo config (plugins, scheme, icons, experiments)
metro.config.js                ← NativeWind metro (hexColors: oklch → hex at build)
.env.example                   ← EXPO_PUBLIC_API_URL
```

### Feature shape (every `features/<name>/`)

```
api/          ← thin functions calling the typed api client (one file per resource)
queries/      ← TanStack Query hooks (useX query/mutation) — the screen-facing data API
services/     ← orchestration / side effects (token persistence, multi-step flows)
store/        ← zustand store(s) for feature client state (optional)
validators/   ← Zod schemas, one per input surface
types/        ← TS interfaces mirroring backend contracts
components/    ← feature components (forms, views) — render + local interaction
pages/        ← screen-level composition (a page per route screen)
utils/        ← optional helpers
```

Route screens in `src/app/**` stay thin — they import a feature `pages/*` component
and render it inside a layout. New top-level folders require developer approval first.

## Test structure (HARD)

Every project built from this template uses this exact test layout. QA rejects a task whose test files break any rule below.

```
test/
├── main.spec.tsx              ← bootstrap: root _layout + Providers
├── app.spec.tsx               ← navigation: route groups + auth guards
├── setup.ts                   ← self-contained test env (runner setup file)
├── helpers.tsx                ← render-with-providers, fixtures, mocks
├── features/<name>/           ← mirrors the feature: api/, queries/, services/, …
├── layout/                    ← mirrors src/layout
├── shared/                    ← mirrors src/shared: components/, hooks/, libs/, i18n/
└── e2e/                       ← <flow>.e2e-spec.ts
```

Source → spec mapping:

```
src/app/_layout.tsx                               → test/main.spec.tsx (root providers)
src/features/auth/validators/signIn.ts            → test/features/auth/validators/signIn.spec.ts
src/shared/libs/storage.ts                        → test/shared/libs/storage.spec.ts
```

1. **One location only.** Every test file lives under `test/` at the project root — never next to a source file, never inside a module/feature folder, never in a second test folder.
2. **Fixed root files — nothing else at the root.** `main.spec.tsx` covers bootstrap only: root `_layout.tsx` mounts the provider stack. `app.spec.tsx` covers navigation only: the entry redirect and `(auth)` / `(protected)` guards. `setup.ts` makes the suite self-contained: RNTL matchers, global mocks (fetch/API client, storage, secure-store, expo-router, i18n), test env values — loaded by the runner as its setup file; no developer `.env` is required. `helpers.tsx` holds render-with-providers (router, query client, i18n, theme), fixtures and mocks — no assertions. Besides these four, the root holds only mirrored folders and `e2e/`.
3. **Mirror the source tree.** Every other spec sits at the path of the file it tests: same folder names, same file name, plus the `.spec.ts` / `.spec.tsx` suffix (`*.test.*` is not used). Never put a test file under `src/app/` — Expo Router would treat it as a route.
4. **One spec per source file.** Moving or renaming a source file moves or renames its spec in the same task.
5. **No catch-all spec.** Logic shared across the app is tested at the mirrored path of the file that owns it — never in a "global" / "misc" / "utils" spec.
6. **Specs import only source code and the helpers file** — never another spec, never a fixture defined inside another spec.
7. **End-to-end tests** live only in `test/e2e/`, named `<flow>.e2e-spec.ts` (or Maestro flows `<flow>.e2e.yaml` if Maestro is chosen). Unit/integration specs never go there; the e2e folder may hold one `helpers` file of its own (route/bridge mocks, launch fixtures).
8. **Runner config follows the folder.** No test runner is wired yet. When one is added (`jest-expo` + React Native Testing Library): `roots: ['<rootDir>/test']`, match `*.spec.{ts,tsx}` excluding `test/e2e/`, `setupFiles: ['<rootDir>/test/setup.ts']`, coverage measures `src/`; then add the commands to the table above. If the config disagrees with this section, fix the config in the same task.
9. **Same tree in every template.** The root file names (`main`, `app`, `setup`, `helpers`, `e2e/`) are identical across all stacks, so a developer moving between a TS and a Python API finds the same layout. A change to this rule is applied to every template at once.
10. **Legacy layout is drift, not precedent.** If existing code or a feature spec places tests elsewhere (e.g. a module's own `tests/` folder or `*.test.ts` files), new and modified tests still follow this section; moving the old ones is its own task — surface the drift to the developer, never silently mass-move files.

## Boundary rules (HARD)

1. **Screens orchestrate, components render.** `app/**` route files and `pages/*`
   compose sections and wire navigation; they do not inline section markup or business
   logic. Visual sections live in `components/<feature>/*`.
2. **Logic lives in hooks/services, not components.** Data fetching goes through
   `queries/*` (TanStack Query); side effects and multi-step flows go through
   `services/*`. Components call hooks — they never call `fetch` or `api` directly.
3. **One typed API client.** All HTTP goes through `shared/libs/api.ts`; `api/*`
   functions wrap it. No bare `fetch` in components, pages, or screens.
4. **Tokens-only styling.** Every color / size / radius comes from a theme token via
   NativeWind classes (`bg-primary`, `text-muted-foreground`, `rounded-lg`). No raw
   hex, no inline magic numbers. See `context/ui-context.md`.
5. **Features do not import each other's internals.** Cross-feature sharing goes
   through `shared/*`. No reaching into another feature's `api`/`store`/`services`.
6. **Immutability everywhere** — never mutate state or props in place; return new
   objects (Zustand `set`, Query cache updates).
7. **Validate at the boundary** — every form/input parses through a Zod schema
   (`validators/*`) before it reaches a service or the API.

## Cross-cutting invariants

- **Route guards** read the auth store status (`useAuthStore.use.status()`):
  `(protected)/_layout` redirects to `/sign-in`, `(auth)/_layout` redirects authed
  users to `/dashboard`.
- **Token lifecycle**: `AuthProvider` awaits `loadSession()` (hydrate secure storage)
  before first paint; the access token is held in memory and attached as
  `Authorization: Bearer` by the api client.
- **Response envelope** (`shared/libs/api.ts`): success `{ success, data, meta? }`,
  failure `{ success: false, error: { code, message, fields? } }` → thrown as
  `ApiError`. See `context/api-context.md`.
- **i18n**: all user-facing copy resolves through `react-i18next` keys; no hardcoded
  strings in JSX.
- **Theme**: `oklch` tokens in `global.css`; `metro.config.js` compiles them to hex at
  build time (RN cannot parse `oklch()` at runtime). Light/dark via
  `prefers-color-scheme` + `userInterfaceStyle: automatic`.

## Decisions log

Locked decisions agents must not re-litigate.

- **Expo managed workflow** — native config via `app.json` plugins; no bare native
  project edits unless a config plugin can't express it.
- **File-based routing (Expo Router)** — the `app/` tree IS the navigation; route
  groups `(public)`/`(auth)`/`(protected)` model the auth boundary.
- **TanStack Query for server state, Zustand for client state** — do not duplicate
  server data into Zustand; the auth store holds only session status + the user object.
- **NativeWind tokens over StyleSheet** — styling is utility classes bound to theme
  tokens; `cn()` (clsx + tailwind-merge) composes them.
- **Typed envelope client** — one `fetch` wrapper owns the success/error envelope and
  bearer auth; features never re-implement HTTP.
