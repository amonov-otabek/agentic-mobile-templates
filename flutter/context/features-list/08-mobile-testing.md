# Feature 08 — Mobile Testing & QA

- Feature ID: FEAT-08
- Status: draft
- Depends on: FEAT-01, FEAT-02, FEAT-03, FEAT-04, FEAT-05, FEAT-06, FEAT-07

> Source of truth for the **test harness**: `flutter_test` for unit + widget tests (with mocked
> repositories and overridden Riverpod providers via `mocktail`), and `integration_test` for
> device-level e2e of critical flows. This spec reflects the CURRENT state honestly and defines the
> harness to add. TO-BUILD on an empty project — nothing is wired yet.

## Current state (nothing wired)

- **No tests exist.** The project is empty (no Dart code, no `test/` or `integration_test/`
  directory). `context/architecture.md` already names the intended commands (`flutter test`,
  `flutter test integration_test`) and the green-build gate (`flutter analyze` + `flutter test`);
  this feature makes them real.
- The quality gates available today are only `flutter analyze` and
  `dart format --output=none --set-exit-if-changed .`.
- FEAT-01–07 ship their own tests as each is built; FEAT-08 establishes the shared harness,
  helpers, and the e2e suite, and backfills coverage where features landed without it.

## Goal

A working test suite for the Flutter app: `flutter_test` + `mocktail` for unit/widget tests (mocked
repositories and `ProviderScope` overrides), plus an `integration_test` e2e flow for the critical
auth path — wired to the commands recorded in `context/architecture.md`.

## Test layout (HARD)

All tests in this feature follow `context/architecture.md` → **Test structure**: `test/` — `main_test.dart`, `app_test.dart`, `flutter_test_config.dart`, `helpers.dart`, folders mirroring `lib/` (`*_test.dart`); e2e in `integration_test/`. Any path elsewhere in this spec that sits outside that layout (`src/**/tests/`, `src/test/`, a root `tests/` or `e2e/`, `*.test.*`, `conftest.py`) describes the current/legacy suite — it is migrated by this feature, never extended.

## Scope

### Unit / widget harness (to add)

- `dev_dependencies`: `flutter_test` (SDK), `mocktail`, and `integration_test` (SDK). Mirror `lib/`
  under `test/`; e2e lives under `integration_test/`.
- `test/helpers.dart` — a `pumpWithProviders(...)` helper that wraps a widget under test in a
  `ProviderScope` (with overrides) + `MaterialApp` (theme + localization delegates), and a fake
  `TokenStore` + fake repositories built with `mocktail`. `test/flutter_test_config.dart` (auto-loaded by Flutter) holds the global setup: a
  `registerFallbackValue` for custom argument types and fake env values.
- Reset secure storage + auth state between tests via provider overrides (no real
  `flutter_secure_storage` plugin calls in unit/widget tests).

### What to test

- `core/network` envelope parser — `{ success:true, data, meta? }` → `ApiResponse<T>` /
  `PaginationMeta`; `{ success:false, error:{ code, message, fields? } }` → typed `Failure`;
  `204` no-body; status → `Failure` mapping (against a mocked `dio` adapter or mocked datasource).
- `core/storage` `TokenStore` — get/set/clear of the access + refresh pair and swallowed-error
  behavior (mocked `flutter_secure_storage`).
- `core/error` mapping — `DioException`/parse/storage exceptions → the right `Failure` variant.
- `features/identity` — the auth repository (token persist/clear, `loadSession`/hydration,
  pending-2FA lifecycle, sign-out calling logout then clearing) and the auth notifier state
  transitions (`unknown → authenticated/unauthenticated`); form validators (accept/reject cases).
- One representative widget per built feature against overridden providers (a fake repository):
  identity `sign_in_form` / `sign_up_form` (validation, submit DTO, `error.fields` surfacing on
  inputs); then a list/form widget per feature as FEAT-03–07 land.

### E2E (`integration_test`, to add)

- `integration_test/` driven by `flutter test integration_test`. Critical flows:
  - sign-in happy path → lands on the dashboard shell tab; sign-out → returns to `/sign-in`.
  - unauthenticated launch → the `go_router` redirect lands on `/sign-in`.
  - locale switch persists; theme follows the OS.
- The backend is mocked/stubbed: point `API_BASE_URL` (via `--dart-define-from-file`) at a local
  mock server returning the `{ success, data, meta? }` envelope, or override the dio adapter — assert
  on rendered envelope-shaped data, never on backend internals.

### Out of scope (by design)

- Visual-regression / golden screenshot diffing, load/perf testing, on-device cloud test farms.
- CI wiring is noted as a follow-up (run `flutter analyze` / `flutter test` / format-check on PR)
  but not built here.

## Consumed API

- None live. Unit/widget tests mock repositories + the network layer; e2e points `API_BASE_URL` at a
  local mock (or overrides the dio adapter) returning the `{ success, data, meta? }` envelope.

## UI / widgets used

- Test utilities in `test/helpers.dart` + `test/flutter_test_config.dart` (`pumpWithProviders` wrapping `ProviderScope` + `MaterialApp`
  + localization); `integration_test/*` flows; existing feature screens/widgets as units under test.

## Acceptance criteria

- [ ] Test layout matches `context/architecture.md` → Test structure: every existing test is moved to its mirrored path and renamed to the stack suffix; nothing test-related remains under the source root; runner config (globs, setup file, e2e dir, coverage root) points at the test folder; the collected-test count is not lower after the move.
- [ ] `flutter_test` + `mocktail` + `integration_test` added as dev deps; `test/` mirrors `lib/` and
      `integration_test/` exists; `flutter test` and `flutter test integration_test` run (commands
      already recorded in `context/architecture.md`).
- [ ] `test/helpers.dart` `pumpWithProviders` mounts a widget in `ProviderScope` (overrides) + `MaterialApp`
      (theme + localization) and resets token + auth state between tests; no real plugin I/O in unit/
      widget tests.
- [ ] `core/network` envelope parser and `core/storage` `TokenStore` are unit-tested (envelope/
      `Failure`/`204`/`meta`/status-mapping; secure-store get/set/clear + swallowed errors).
- [ ] `features/identity` auth repository, auth notifier, and validators are unit-tested (token
      lifecycle, pending-2FA, sign-out; state transitions; accept/reject cases).
- [ ] At least one widget test per built feature against overridden providers; assertions branch on
      `error.code`, never localized message text.
- [ ] An `integration_test` e2e covers sign-in → dashboard, sign-out, the unauthenticated route-guard
      redirect, and a preference (locale/theme) check against a mocked API base.
- [ ] Tokens-only theming, a11y, dart analyze, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
