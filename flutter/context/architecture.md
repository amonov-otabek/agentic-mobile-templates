# Architecture Context

HARD CONTRACT: file placement, stack choices, and boundary rules here are
non-negotiable for Builders — QA rejects violations. This is the planned foundation
for an empty Flutter project; keep it locked once building starts.

## Stack

| Layer                 | Technology                                              | Role               |
| --------------------- | ------------------------------------------------------- | ------------------ |
| SDK                   | Flutter (stable channel)                               | app framework      |
| Language              | Dart 3 — sound null safety, records, patterns           | all source         |
| Design                | Material 3 (`useMaterial3: true`), light + dark themes  | UI system          |
| State management      | Riverpod (`flutter_riverpod` + `riverpod_generator`)    | app/UI state       |
| Navigation            | `go_router` (declarative routes + redirect guards)      | routing/deep links |
| HTTP client           | `dio` + `retrofit` (typed clients) + `pretty_dio_logger`| API access         |
| Models / serde        | `freezed` + `json_serializable`                         | immutable models   |
| Secure storage        | `flutter_secure_storage`                                | JWT tokens         |
| Local cache (non-secret) | `shared_preferences`                                | prefs, flags       |
| i18n                  | `intl` + `flutter_localizations` (ARB files)            | localized strings  |
| Env / config          | `--dart-define` / `--dart-define-from-file` flavors     | build-time config  |
| Testing               | `flutter_test` (unit/widget), `integration_test` (e2e), `mocktail` | tests   |
| Lint / format         | `very_good_analysis` (or `flutter_lints`), `dart format`| lint/format        |
| Codegen runner        | `build_runner`                                          | freezed/json/riverpod gen |

Package manager: **pub** via `pubspec.yaml` (`flutter pub get`). One lockfile
(`pubspec.lock`) is committed. Codegen output (`*.g.dart`, `*.freezed.dart`) is
generated, not hand-edited.

## Commands

| Action | Command |
| --- | --- |
| Install deps | `flutter pub get` |
| Codegen (one-shot) | `dart run build_runner build --delete-conflicting-outputs` |
| Codegen (watch) | `dart run build_runner watch --delete-conflicting-outputs` |
| Run (dev) | `flutter run --dart-define-from-file=env/dev.json` |
| Analyze (native gate) | `flutter analyze` |
| Format check | `dart format --output=none --set-exit-if-changed .` |
| Unit + widget tests | `flutter test` |
| E2E / integration | `flutter test integration_test` |
| Build APK (release) | `flutter build apk --release --dart-define-from-file=env/prod.json` |
| Build iOS (release) | `flutter build ipa --release --dart-define-from-file=env/prod.json` |

QA and Builders run exactly these — keep them accurate. `flutter analyze` +
`flutter test` are the green-build gate.

## Folder structure

Imports use package-relative paths (`package:app/...`); no deep `../../..` chains
across feature boundaries.

```
lib/
├── main.dart                  ← entry: bootstrap + ProviderScope + MaterialApp.router
├── app.dart                   ← root widget: theme, router, localization wiring
├── core/
│   ├── network/               ← dio instance, interceptors (auth/refresh/logging), api_client
│   ├── theme/                 ← ThemeData (light/dark) + tokens.dart (the design tokens)
│   ├── router/                ← go_router config, routes, redirect guards
│   ├── storage/               ← secure token store, prefs wrapper
│   ├── error/                 ← Failure types, exception → Failure mapping
│   ├── env/                   ← typed access to --dart-define config
│   └── localization/          ← l10n setup, ARB files, generated strings
├── features/<name>/           ← one folder per feature (see Feature shape)
└── shared/                    ← cross-feature widgets, extensions, utils (no feature logic)

test/                          ← mirrors lib/ for unit + widget tests (see Test structure)
integration_test/              ← end-to-end flows only
env/                           ← dev.json / staging.json / prod.json (--dart-define-from-file)
```

### Feature shape (every `features/<name>/`)

```
data/
  models/         ← freezed DTOs + json_serializable (API row shapes)
  datasources/    ← retrofit API definitions (remote), local datasources (optional)
  repositories/   ← repository implementations (map DTO → domain, errors → Failure)
domain/
  entities/       ← plain immutable domain types (freezed)
  repositories/   ← abstract repository interfaces
  usecases/       ← single-purpose business operations (optional, for complex flows)
presentation/
  providers/      ← Riverpod notifiers + state (immutable, freezed)
  screens/        ← route-level widgets (one per screen)
  widgets/        ← feature-local widgets composed by screens
```

Every file a Builder creates must fit this tree; test files follow
**Test structure** below. New top-level folders under `lib/`
require developer approval first.

## Test structure (HARD)

Applies to every project built from this template. QA rejects a task whose test files break any rule below.

```
lib/core/network/api_client.dart                       → test/core/network/api_client_test.dart
lib/features/auth/data/repositories/auth_repository_impl.dart
  → test/features/auth/data/repositories/auth_repository_impl_test.dart
lib/features/auth/presentation/screens/sign_in_screen.dart
  → test/features/auth/presentation/screens/sign_in_screen_test.dart
```

```
test/
├── helpers/                   ← pump_app.dart, mocks.dart, fixtures.dart (no _test suffix)
├── app_test.dart              ← root widget: theme, router, localization wiring
├── core/                      ← mirrors lib/core: network/, router/, storage/, error/, …
├── features/<name>/           ← mirrors the feature: data/, domain/, presentation/
└── shared/                    ← mirrors lib/shared
integration_test/              ← <flow>_test.dart (end-to-end only)
```

1. **One location only.** Every test file lives under `test/` — never next to a source file, never inside a module/feature folder, never in a second test folder.
2. **Mirror the source tree.** `test/` mirrors `lib/` one-to-one. Same folder names, same file names, plus the `_test.dart` suffix — `flutter test` only discovers `*_test.dart` (`*_spec.dart` is not used).
3. **One spec per source file**, at its mirrored path. Moving or renaming a source file moves or renames its spec in the same task.
4. **No catch-all spec.** Logic shared across the app is tested at the mirrored path of the file that owns it — never in a "global" / "misc" / "utils" spec file.
5. **Shared test code** lives in `test/helpers/` (`pump_app.dart` with ProviderScope overrides, mocktail mocks, fixtures). Files there have no `_test.dart` suffix. Specs never import from other specs.
6. **End-to-end tests** live only in `integration_test/` (the folder Flutter requires), named `<flow>_test.dart` and mirroring the feature name. Unit/integration specs never go there, and e2e specs never go anywhere else.
7. **Runner config follows the folder.** `flutter test` runs `test/`; `flutter test integration_test` runs e2e; coverage (`--coverage`) measures `lib/`. Generated files (`*.g.dart`, `*.freezed.dart`) get no spec.
8. **Legacy layout is drift, not precedent.** If existing code or a feature spec places tests elsewhere (e.g. a module's own `tests/` folder), new and modified tests still go to the mirrored path; moving the old ones is its own task — surface the drift to the developer, never silently mass-move files.

## Boundary rules (HARD)

1. **Widgets render; notifiers hold logic.** Widgets read state and dispatch intents
   only — no API calls, no business logic, no `dio` inside a widget.
2. **Repositories own all data access.** Every API/storage call goes through a
   repository; presentation never touches `dio`, `retrofit`, or storage directly.
3. **No raw DTOs in the UI.** Data-layer models map to domain entities; widgets bind
   to entities/state, never to raw JSON or response DTOs.
4. **Models are immutable.** All models/entities/state are `freezed` (or `const`
   value types); never mutate — produce a new value with `copyWith`.
5. **Tokens-only theming.** Every color, text style, spacing, and radius comes from
   `core/theme` (ThemeData + `tokens.dart`). No inline hex, no magic numbers.
6. **Features do not import each other's internals.** Cross-feature reuse goes
   through `shared/` or a domain interface — not by reaching into another feature's
   `data`/`presentation`.
7. **One direction of dependency:** `presentation → domain → data`. Domain depends on
   nothing concrete; data implements domain interfaces.

## Cross-cutting invariants

- **Config via `--dart-define`** (typed in `core/env`); the API base URL is never
  hardcoded in source.
- **Tokens in secure storage only** — `flutter_secure_storage`; never `shared_preferences`,
  never logs.
- **Auth interceptor** attaches the bearer token and performs single-flight refresh on
  `401`, retrying the original request once; a failed refresh clears tokens and routes
  to login.
- **Response envelope (consumed):** the app parses the backend envelope —
  - success: `{ success: true, data, meta? }`
  - error: `{ success: false, error: { code, message, fields? } }`
  - lists: `data` is the items array; `meta: { page, limit, total, totalPages }`.
  See `context/api-context.md` for the contract.
- **All errors become typed `Failure`s** at the repository boundary; the UI renders
  `Failure` states, never raw exceptions or stack traces.
- **Every user-facing string** is localized via `intl` ARB — no literals in widgets.

## Decisions log

Locked decisions agents must not re-litigate.

- **Riverpod over Bloc/Provider** — compile-safe DI, testable notifiers, fine-grained
  rebuilds; standardized across features (Bloc was considered and rejected for this
  template to keep one state model).
- **`go_router` over Navigator 1.0** — declarative routes, typed params, a single
  redirect for auth gating and deep links.
- **`dio` + `retrofit` over bare `http`** — interceptors (auth/refresh/logging),
  typed generated clients, cancellation, and consistent error mapping.
- **Clean-ish layering (`data`/`domain`/`presentation`)** — feature-first folders with
  a one-way dependency rule; not full DDD, just enough to keep UI off the network.
- **`freezed` everywhere for data** — immutability and `copyWith` are mandatory, not
  optional.
