# Feature 01 — Mobile Setup & Scaffold

- Feature ID: FEAT-01
- Status: draft
- Depends on: none

> Source of truth for the **mobile foundation** of this Expo + React Native app. It stands
> up the runnable shell — stack, env, typed envelope/JWT API client, secure-store token
> plumbing, providers, file-based navigation, theming, and i18n — and writes **no auth or
> business logic**. The identity feature lands in FEAT-02.

## Goal

Stand up the application skeleton so the team is ready to write feature code: a strict
TypeScript Expo SDK 56 / React Native 0.85 app (iOS + Android, secondary web) that boots
clean, talks to the backend through one typed envelope-aware `fetch` client with bearer
auth, persists tokens in `expo-secure-store`, navigates via Expo Router file-based route
groups with an auth guard, themes light/dark from NativeWind design tokens, and localizes
through i18next (en/ru/uz).

## Scope

- **Stack & tooling** — Expo SDK ~56 (managed workflow), React Native ~0.85 (Hermes),
  TypeScript 6 (`strict`, `expo/tsconfig.base`), Expo Router 56 (`typedRoutes`,
  `experiments.reactCompiler` on). Path aliases `@/*` → `src/*` and `@/assets/*` →
  `assets/*` (tsconfig). ESLint (`eslint-config-expo`) + Prettier + Husky + lint-staged.
  Package manager: pnpm (`pnpm-lock.yaml` canonical — no second lockfile).
- **Env** — base URL from `EXPO_PUBLIC_API_URL` only (public, inlined at build; default
  `http://localhost:3000`); a physical device needs a LAN IP, Android emulator uses
  `http://10.0.2.2:3000`. No secrets in the client; `.env.example` holds the public var.
- **Typed API client** (`src/shared/libs/api.ts`) — `api.get/getPaginated/post/put/patch/delete`;
  prepends `EXPO_PUBLIC_API_URL` origin (builds the query string by hand, since Hermes'
  `URLSearchParams` is unreliable), attaches the in-memory bearer token
  (`authorization: Bearer <token>`), unwraps the `{ success, data, meta? }` envelope, and
  throws a typed `ApiError(status, code, message, fields?)` on a non-2xx / `false` envelope /
  parse failure. `204` → `data: undefined`. Pagination via `?page&limit` → `getPaginated` →
  `{ data, meta }`.
- **Secure token storage** (`src/shared/libs/storage.ts`) — async `getItem/setItem/removeItem`
  over `expo-secure-store` (Keychain / Keystore) on native, `localStorage` on web; storage
  failures are non-fatal (the in-memory token still drives the session).
- **JWT + refresh plumbing** — in-memory access token (`setAccessToken` / `getAccessToken`
  in the client); the auth service (FEAT-02) persists access + refresh in secure storage and
  rehydrates via `loadSession()` at startup. Refresh-token rotation and forced sign-out on
  `401/403` are the contract the identity feature uses.
- **i18n** (`src/shared/i18n`) — i18next + react-i18next + `expo-localization` detector;
  `messages/{en,ru,uz}.json`; `routing.ts` locale list; all user-facing strings resolve
  through `t(...)` — no hardcoded copy.
- **Theme / tokens** (`src/global.css`) — Tailwind v4 `@theme inline` mapping NativeWind
  color names to CSS variables; OKLCH token palette with `:root` (light) +
  `@media (prefers-color-scheme: dark)` overrides; `metro.config.js`
  (`withNativewind(..., { hexColors: true })`) compiles `oklch()` → hex at build (RN cannot
  parse `oklch()` at runtime). Light/dark auto via `app.json` `userInterfaceStyle: automatic`.
- **Navigation shell** (`src/app/*`, Expo Router file-based) — root `_layout.tsx` mounts
  `<Providers>` and imports `global.css`; `index.tsx` redirects authed→`/dashboard`,
  guest→`/home`. Three route groups: `(public)` (home), `(auth)` (sign-in/up, forgot/reset,
  verify-email, two-factor), `(protected)` (a `Tabs` navigator: dashboard, messages, users,
  notifications, settings).
- **Auth guard** — group `_layout.tsx` files read the auth store status: `(protected)/_layout`
  redirects unauthenticated users to `/sign-in`; `(auth)/_layout` redirects authed users to
  `/dashboard`. Guards are UX-only — the backend remains the authz source of truth (a `401`
  clears the session).
- **Providers** (`src/shared/components/provider/providers.tsx`) — compose once in order
  i18n → TanStack Query → AuthProvider (awaits `loadSession()` before first paint) → toaster;
  mounted in `app/_layout.tsx`.
- **Shared UI kit** (`src/shared/components/ui/*`) — `Button`, `Text`, `Input`, `Field`,
  `Label`, `Card`, `Checkbox`, `Toaster` primitives (cva variants, tokens-only); widgets
  `Icons` (lucide map) + `LocaleSwitch`; `shared/hooks/use-toast`; `shared/libs/{store,utils,validators}`.
- Out of scope: any auth/business logic, feature data wiring, public marketing site beyond a
  single placeholder `home`, tests (FEAT-08).

## Consumed API

- None directly. This feature defines the **client** of the backend envelope contract
  (`/api/v1`, `{ success, data, meta? }`) consumed by every later feature; it makes no real
  endpoint calls itself. See `context/api-context.md`.

## UI / components used

- Route shells (`src/app/**`, `src/layout/{auth-layout,coming-soon}`),
  `shared/components/provider/*`, `shared/components/ui/*` primitives,
  `shared/components/widgets/{icons,locale-switch}`, `shared/hooks/use-toast`, tokens in
  `src/global.css`.

## Acceptance criteria

- [ ] `pnpm install` succeeds; only one lockfile (`pnpm-lock.yaml`) present; `pnpm start`/
      `ios`/`android`/`web` boot Metro and render the shell.
- [ ] `@/*` and `@/assets/*` aliases resolve via tsconfig; no deep `../../..` chains.
- [ ] `shared/libs/api.ts` prepends `EXPO_PUBLIC_API_URL`, attaches the bearer token, unwraps
      `{ success, data, meta? }`, throws typed `ApiError` on failure, and resolves `204` to
      `undefined`; `getPaginated` returns `{ data, meta }`.
- [ ] `shared/libs/storage.ts` reads/writes tokens via `expo-secure-store` on native and
      `localStorage` on web; storage failures do not crash; no secrets bundled (only
      `EXPO_PUBLIC_API_URL`).
- [ ] i18next loads `en/ru/uz`, the `expo-localization` detector + `LocaleSwitch` work, and
      every visible string resolves through `t(...)`.
- [ ] Light/dark theme follows the OS (`userInterfaceStyle: automatic`); all visual values
      come from `global.css` tokens compiled to hex by `metro.config.js` (no raw hex/px in JSX).
- [ ] Expo Router mounts `(public)`, `(auth)`, and `(protected)` (Tabs) groups; `index.tsx`
      redirects authed→`/dashboard`, guest→`/home`.
- [ ] `(protected)/_layout` redirects unauthenticated users to `/sign-in`; `(auth)/_layout`
      keeps authed users out of the auth group.
- [ ] Providers compose i18n → Query → AuthProvider → toaster once and mount in
      `app/_layout.tsx`; `AuthProvider` awaits `loadSession()` before first paint.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
