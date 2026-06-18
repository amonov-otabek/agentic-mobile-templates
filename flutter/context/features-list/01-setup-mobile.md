# Feature 01 — Mobile Setup & Scaffold

- Feature ID: FEAT-01
- Status: draft
- Depends on: none

> Source of truth for the **mobile foundation** of this Flutter + Dart 3 app. It stands up
> the runnable shell — stack, env/config, typed envelope/JWT API client, secure-store token
> plumbing with a refresh interceptor, `go_router` shell with an auth redirect guard, Material 3
> theme/tokens, i18n, and the `ProviderScope` — and writes **no auth or business logic**.
> This is a TO-BUILD spec on a currently empty project (no Dart code exists yet). The identity
> feature lands in FEAT-02.

## Goal

Stand up the application skeleton so the team is ready to write feature code: a strict Dart 3
(sound null safety) Flutter (stable channel) app (iOS + Android) that boots clean, talks to the
backend through one typed `dio` + `retrofit` client that unwraps the `{ success, data, meta? }`
envelope and attaches a bearer token, persists JWTs in `flutter_secure_storage` with a
single-flight `401` refresh interceptor, navigates via `go_router` (auth flow + a protected
shell route) behind a redirect guard, themes light/dark from Material 3 design tokens, and
localizes through `intl` (en/ru/uz) — all wrapped in a Riverpod `ProviderScope`.

## Scope

### Planned structure (to build)

- **Stack & tooling** — Flutter (stable), Dart 3 (records, patterns, sound null safety),
  Material 3 (`useMaterial3: true`), `flutter_riverpod` + `riverpod_generator`, `go_router`,
  `dio` + `retrofit` + `pretty_dio_logger`, `freezed` + `json_serializable`,
  `flutter_secure_storage`, `shared_preferences`, `intl` + `flutter_localizations`,
  `very_good_analysis` (or `flutter_lints`). Package manager **pub** (one committed
  `pubspec.lock`). Codegen via `build_runner` (`dart run build_runner build
  --delete-conflicting-outputs`); `*.g.dart` / `*.freezed.dart` are generated, never
  hand-edited. Path style: package-relative (`package:app/...`) — no deep `../../..` chains.
- **Entry & root** — `lib/main.dart` bootstraps and wraps the app in `ProviderScope`;
  `lib/app.dart` is the root widget building `MaterialApp.router` (theme + router +
  localization wiring).
- **Env / config** (`lib/core/env/`) — typed access to `--dart-define` / `--dart-define-from-file`
  values; `env/{dev,staging,prod}.json` hold the per-flavor `API_BASE_URL` (and similar). The
  API base URL is **never** hardcoded in source. Run dev with
  `flutter run --dart-define-from-file=env/dev.json`.
- **Typed API client** (`lib/core/network/`) — a configured `dio` instance (base URL from
  `core/env`, timeouts, `pretty_dio_logger`) plus `retrofit` datasource definitions per feature.
  An envelope parser turns every response into `ApiResponse<T>` (`{ data, meta? }`) or maps
  `{ success:false, error:{ code, message, fields? } }` → typed `Failure`; `204` → no body;
  list responses expose `PaginationMeta { page, limit, total, totalPages }`. Envelope/error/
  pagination types are declared once here and imported (never re-declared per feature).
- **Secure token storage** (`lib/core/storage/`) — a `TokenStore` over `flutter_secure_storage`
  (Keychain / Keystore) for the access + refresh JWT pair, plus a `shared_preferences` wrapper
  for non-secret prefs/flags. Tokens never touch prefs, logs, or source.
- **JWT + refresh interceptor** (`lib/core/network/`) — an auth interceptor attaches
  `Authorization: Bearer <access>` to `/api/v1` calls (except documented public ones), and on a
  `401` performs a **single-flight** refresh via the refresh token, retries the original request
  once, and on refresh failure clears `TokenStore` and routes to login (`go_router` redirect).
  The interceptor owns all token attach/refresh — features never manage tokens by hand.
- **Error layer** (`lib/core/error/`) — `Failure` sealed/`freezed` types (`ValidationFailure`,
  `AuthFailure`, `ForbiddenFailure`, `NotFoundFailure`, `ConflictFailure`, `ServerFailure`,
  `NetworkFailure`) and the exception → `Failure` mapping (status table from `api-context.md`).
- **i18n** (`lib/core/localization/`) — `flutter_localizations` + `intl` ARB files
  (`app_en.arb`, `app_ru.arb`, `app_uz.arb`); generated lookup; locale resolution follows the OS
  with a user override. Every user-facing string resolves through the generated `l10n` lookup —
  no literals in widgets.
- **Theme / tokens** (`lib/core/theme/`) — `tokens.dart` (the only place raw `Color`/spacing/
  radius literals live: `AppColors`, `AppSpacing`, `AppRadius`, `AppFonts`) and `app_theme.dart`
  (light + dark `ThemeData` via `ColorScheme.fromSeed` + token overrides + a `TextTheme` and an
  `AppStatusColors` `ThemeExtension` for success/warning). Widgets read
  `Theme.of(context).colorScheme.*` / `textTheme.*` — never restate hex.
- **Navigation shell** (`lib/core/router/`) — a `go_router` config with: an auth flow branch
  (`/sign-in`, `/sign-up`, `/verify-email`, `/forgot-password`, `/reset-password`,
  `/two-factor`), a protected `ShellRoute` (or `StatefulShellRoute`) hosting the bottom-nav
  tabs (dashboard, messages, notifications, settings, plus role-gated admin), and a top-level
  `redirect` guard reading auth state: unauthenticated → `/sign-in`, authed-in-auth-flow →
  the shell home. The guard is UX-only; the backend stays the authz source of truth.
- **Riverpod wiring** (`ProviderScope` in `main.dart`) — core providers exposed (dio/api client,
  `TokenStore`, router, current locale/theme-mode) so features inject via `ref`, never via global
  singletons. An auth-state provider feeds the router redirect; FEAT-02 fills its logic.
- **Shared widget kit** (`lib/shared/widgets/`) — primitives the features compose: a primary/
  secondary `AppButton`, `AppTextField`/`AppField`, `AppCard`, an async-state scaffold
  (loading skeleton / empty / `Failure` + retry), a locale switcher. Tokens-only, `const`
  where possible; shared widgets registered in `context/ui-context.md`.
- **Out of scope:** any auth/business logic, real feature data wiring, web/desktop targets,
  tests (FEAT-08).

## Consumed API

- None directly. This feature defines the **client** of the backend envelope contract
  (`/api/v1`, `{ success, data, meta? }` / `{ success:false, error:{ code, message, fields? } }`)
  consumed by every later feature; it makes no real endpoint calls itself. See
  `context/api-context.md`.

## UI / widgets used

- `lib/app.dart` (root `MaterialApp.router`), `lib/core/router/*` route definitions, the
  protected `ShellRoute` bottom-nav scaffold, `lib/core/theme/*` (`tokens.dart` + `app_theme.dart`),
  and `lib/shared/widgets/*` primitives (`AppButton`, `AppTextField`/`AppField`, `AppCard`,
  the async-state scaffold, the locale switcher).

## Acceptance criteria

- [ ] `flutter pub get` succeeds; one committed `pubspec.lock`; `dart run build_runner build
      --delete-conflicting-outputs` generates `*.g.dart`/`*.freezed.dart`; `flutter run
      --dart-define-from-file=env/dev.json` boots the shell on iOS + Android.
- [ ] Imports are package-relative (`package:app/...`); no deep `../../..` chains across feature
      boundaries; `flutter analyze` and `dart format --output=none --set-exit-if-changed .` are clean.
- [ ] `core/network` exposes one configured `dio` + `retrofit` client; the envelope parser yields
      `ApiResponse<T>` / `PaginationMeta` on success and a typed `Failure` on
      `{ success:false, error }`; `204` carries no body.
- [ ] `core/storage` persists the access + refresh JWT pair via `flutter_secure_storage`; no token
      ever lands in `shared_preferences`, logs, or source; the API base URL comes from `core/env`
      (`--dart-define`), not source.
- [ ] The auth interceptor attaches `Authorization: Bearer <token>`, performs single-flight refresh
      on `401`, retries the original request once, and on refresh failure clears tokens and routes
      to `/sign-in`.
- [ ] `intl` loads en/ru/uz from ARB, locale follows the OS with an override via the locale switcher,
      and every visible string resolves through the generated `l10n` lookup.
- [ ] Light/dark `ThemeData` both ship from `ColorScheme.fromSeed` + tokens; all color/spacing/
      radius/text come from `core/theme` (`tokens.dart`); no inline `Color(0x...)` or magic numbers
      in widgets.
- [ ] `go_router` mounts the auth flow branch and the protected `ShellRoute` (bottom-nav tabs); the
      top-level `redirect` sends unauthenticated users to `/sign-in` and keeps authed users out of
      the auth flow.
- [ ] `ProviderScope` wraps the app in `main.dart`; core providers (api client, `TokenStore`, router,
      locale/theme, auth-state) are injected via `ref`, with no global singletons.
- [ ] Tokens-only theming, a11y, dart analyze, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
