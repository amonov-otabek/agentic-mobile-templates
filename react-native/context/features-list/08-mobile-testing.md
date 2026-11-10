# Feature 08 — Mobile Testing & QA

- Feature ID: FEAT-08
- Status: draft
- Depends on: FEAT-01, FEAT-02, FEAT-03, FEAT-04, FEAT-05, FEAT-06, FEAT-07

> Source of truth for the **test harness**: Jest + React Native Testing Library for
> unit/component tests (with a mocked API client and mocked query hooks), and a device-level
> e2e runner (Maestro preferred, Detox alternative) for critical flows. This spec reflects the
> CURRENT state honestly and defines the harness to add.

## Current state (nothing wired)

- **No test runner is installed or configured.** `package.json` has no `test` script and no
  test deps (`jest`, `jest-expo`, `@testing-library/react-native`, `detox`, or Maestro are
  absent). `context/architecture.md` records this: "There is no test runner wired yet; add
  one (and the command here) when tests are introduced."
- The only quality gates today are `pnpm lint` (`expo lint`), `pnpm exec tsc --noEmit`
  (typecheck), and `pnpm format:check`.
- The identity feature (FEAT-02) is built but currently **untested**; FEAT-03–07 are scaffolds.

## Goal

A working test suite for the Expo app: `jest-expo` + React Native Testing Library for
component/unit tests (mocked API client and mocked query hooks), plus a Maestro (or Detox)
e2e flow for the critical auth path — wired to pnpm scripts and recorded in
`context/architecture.md`.

## Test layout (HARD)

All tests in this feature follow `context/architecture.md` → **Test structure**: `test/` — `main.spec.ts(x)`, `app.spec.ts(x)`, `setup.ts`, `helpers.ts(x)`, folders mirroring `src/` (`*.spec.ts(x)`), `e2e/` (`*.e2e-spec.ts`, own `helpers.ts` allowed). Any path elsewhere in this spec that sits outside that layout (`src/**/tests/`, `src/test/`, a root `tests/` or `e2e/`, `*.test.*`, `conftest.py`) describes the current/legacy suite — it is migrated by this feature, never extended.

## Scope

### Unit / component harness (to add)

- Install `jest-expo` (the Expo-aligned Jest preset), `jest`, `@testing-library/react-native`,
  `@testing-library/jest-native` (matchers), and `react-test-renderer` pinned to the React
  version. Add `jest.config.js` (preset `jest-expo`, `transformIgnorePatterns` for RN/Expo
  packages) and `test`, `test:watch`, `test:coverage` scripts.
- `test/setup.ts` — register jest-native matchers, mock
  `expo-secure-store`, `expo-localization`, and `expo-router` navigation, and reset the
  in-memory token + auth store between tests.
- `test/helpers.tsx` — a `renderWithProviders` helper mounting components with i18n +
  React Query (fresh client, retries off).

### What to test

- `shared/libs/api.ts` — envelope unwrap, typed `ApiError` (status/code/fields), `204` →
  `undefined`, pagination `meta`, bearer header attach (against a mocked `fetch`).
- `shared/libs/storage.ts` — secure-store get/set/remove + the web `localStorage` fallback +
  swallowed-error behavior (mocked `expo-secure-store`).
- `identity/services/auth.service.ts` — token persist/clear, `loadSession` hydration,
  `isAuthenticated`, pending-2FA token lifecycle, `signOut` calling logout then clearing.
- `identity/validators/auth.validator.ts` — Zod schemas (sign-in/up, reset, two-factor,
  change-password) accept/reject cases.
- One representative component per built/feature domain against mocked query hooks
  (`jest.mock("../queries/*.queries")`): identity `sign-in-form` / `sign-up-form` (validation,
  submit DTO, `ApiError.fields` surfacing); then a list/form per feature as FEAT-03–07 land.

### E2E (to add)

- **Maestro** (preferred — YAML flows, no native build harness) or **Detox** (if a fuller
  gray-box harness is wanted). Critical flows:
  - sign-in happy path → lands on the dashboard tab; sign-out → returns to `/home`.
  - unauthenticated launch → `(protected)` redirects to `/sign-in`.
  - locale switch persists; theme follows the OS.
- Backend is mocked/stubbed via a fixture API base (`EXPO_PUBLIC_API_URL` pointed at a local
  mock server returning the `{ success, data, meta? }` envelope) — assert on rendered
  envelope-shaped data, never on backend internals.

### Out of scope (by design)

- Visual-regression/screenshot diffing, load/perf testing, on-device cloud test farms.
- CI wiring is noted as a follow-up (run `test` / `lint` / `tsc` on PR) but not built here.

## Consumed API

- None live. Component tests mock the query hooks + `shared/libs/api.ts`; e2e points
  `EXPO_PUBLIC_API_URL` at a local mock returning the `{ success, data, meta? }` envelope.

## UI / components used

- Test utilities in `test/setup.ts` + `test/helpers.tsx` (`renderWithProviders` wrapping i18n + Query); Maestro
  flows under `test/e2e/` (`<flow>.e2e.yaml`, or Detox `<flow>.e2e-spec.ts`); existing feature components and screens as units
  under test.

## Acceptance criteria

- [ ] Test layout matches `context/architecture.md` → Test structure: every existing test is moved to its mirrored path and renamed to the stack suffix; nothing test-related remains under the source root; runner config (globs, setup file, e2e dir, coverage root) points at the test folder; the collected-test count is not lower after the move.
- [ ] `jest-expo` + RNTL + jest-native installed; `jest.config.js` + `test`/`test:watch`/
      `test:coverage` scripts added; the test command recorded in `context/architecture.md`.
- [ ] `test/setup.ts` mocks `expo-secure-store` / `expo-localization` / `expo-router` and
      resets token + auth store between tests; `renderWithProviders` mounts i18n + Query.
- [ ] `shared/libs/api.ts` and `shared/libs/storage.ts` are unit-tested (envelope/`ApiError`/
      `204`/`meta`/bearer; secure-store + web fallback).
- [ ] `identity/services/auth.service.ts` and `identity/validators/auth.validator.ts` are
      unit-tested (token lifecycle, pending-2FA, sign-out; Zod accept/reject).
- [ ] At least one component test per built feature against mocked hooks/client; assertions
      branch on `error.code`, never localized message text.
- [ ] A Maestro (or Detox) e2e covers sign-in → dashboard, sign-out, the unauthenticated
      route-guard redirect, and a preference (locale/theme) check against a mocked API base.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
