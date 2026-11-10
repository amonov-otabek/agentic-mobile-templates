# Project Standards

Implementation rules for all code. The planned standard for this Flutter mobile
app (foundation for an empty project). Treat as locked.

## Engineering mindset

1. **Think first** — read the task block and relevant context before writing code.
2. **Scope is sacred** — build what the task says; nothing more, nothing less.
3. **Every change is verifiable** — if it can't be tested or observed, it isn't done.
4. **Clean over clever** — boring, readable widgets and notifiers win.
5. **One thing at a time** — never mix refactoring with feature work.
6. **Failures are expected** — handle errors as first-class flow (typed `Failure`s).
7. **Fix root causes** — never patch symptoms or wrap broken code.

## Dart / Flutter

- **Effective Dart** is the baseline; `very_good_analysis` (or `flutter_lints`) is the
  enforced ruleset. `flutter analyze` must be clean — warnings are failures.
- **Sound null safety** — no `!` to silence the analyzer; narrow with `if`/`case` or
  provide real defaults. Avoid `late` unless initialization is provably before use.
- **No `dynamic`** — use precise types, generics, or sealed/`freezed` unions.
- `dart format` is canonical; run it before committing (CI checks it).
- Prefer `const` constructors and `const` widgets wherever possible (rebuild cost).
- Explicit return types on public methods and providers; avoid leaking inference
  across library boundaries.

## Immutability

- All models, entities, and notifier state are **`freezed`** (or `const` value types).
- Never mutate — produce a new value via `copyWith`. No growing/clearing shared lists
  in place; build a new collection.
- Widget fields are `final`; widgets are `const` where their inputs allow.

## State management discipline (Riverpod)

- One notifier owns one feature's state surface; state is a single immutable
  `freezed` type (e.g. `AsyncValue<T>` or an explicit `{data, loading, error}` union).
- Widgets `watch` state and call notifier methods; they never construct repositories
  or perform I/O themselves.
- Side effects (navigation, snackbars) are triggered from the widget via `ref.listen`,
  not from inside the notifier.
- Providers are declared with the generator where practical; dependencies are injected
  via `ref`, never via global singletons.

## Layered feature rules

- **Presentation (widgets/screens/providers):** render state, dispatch intents. No
  business logic, no networking, no storage access.
- **Domain (entities/interfaces/usecases):** pure Dart; depends on nothing concrete.
- **Data (models/datasources/repositories):** owns all I/O. Repositories map DTO →
  domain entity and exceptions → `Failure`. All `dio`/`retrofit`/storage lives here.
- **Dependency direction:** `presentation → domain → data`. Domain never imports
  Flutter or `dio`.

## Errors

- Repositories convert every exception (`DioException`, parse errors, storage errors)
  into a typed `Failure` (`core/error`). The UI renders `Failure` states.
- Never surface raw exceptions, stack traces, or backend internals to the user.
- No empty `catch`; expected failures (validation, not-found, conflict, offline) are
  flow, mapped to specific `Failure` variants.
- Log diagnostic context (prefixed `[feature/operation]`) — never log tokens or PII.

## API consumption

- All requests go through the shared `dio` instance and `retrofit` datasources.
- Every response is parsed through `freezed`/`json_serializable` models; assert on the
  envelope's `error.code`, not message text (messages are localized server-side).
- The auth interceptor owns token attach + single-flight refresh; features never
  manage tokens by hand.

## Security

- JWTs live only in `flutter_secure_storage`. Never in `shared_preferences`, logs, or
  source. No secrets compiled into the binary — config via `--dart-define`.
- Validate/clamp anything used in a request (ids, query params) before sending.
- Respect platform secure-flag / no-screenshot needs on sensitive screens when scoped.

## Accessibility

- Touch targets ≥ 48×48 logical px; `Semantics` labels on icon-only controls; images
  have semantic labels where meaningful.
- Text scales with the OS setting — no fixed-height containers that clip scaled text.
- Color contrast ≥ 4.5:1 against its background; never use color as the only signal.
- Visible focus/selection states; controls disabled while their async action runs.

## i18n

- All user-facing strings resolve through `intl` ARB files (`core/localization`) — no
  literals in widgets. Format dates/numbers/currency through `intl`, not by hand.

## Testing

- **Where tests live (HARD):** every test file goes in `test/` (`main_test.dart`, `app_test.dart`, `flutter_test_config.dart`, `helpers.dart`) + `integration_test/`, at the path that
  mirrors its source file — see `context/architecture.md` → **Test structure**. Never beside a
  source file, never inside a module/feature folder. `setup` keeps the suite
  self-contained: tests run on a clean checkout with no developer `.env`.
- `flutter_test` for unit (notifiers, repositories, mappers) and widget tests;
  `mocktail` for mocks; `integration_test` for critical e2e flows.
- New behavior ships with a happy-path test plus at least one edge/error case
  (e.g. a `Failure` path). Pre-existing tests stay green — a task that breaks them is
  not done.

## Comments, TODOs, dependencies

- Comments explain *why*, where the code can't. No TODO/FIXME/HACK without a linked task.
- Prefer existing dependencies; a new pub package requires developer approval (note it
  in QA notes) and must be sound-null-safe and maintained.
- Before using any third-party package, load its installed skill if present or verify
  against the installed version's docs — APIs drift, training data goes stale.
- pub is the package manager (see `context/architecture.md`); commit one `pubspec.lock`.
