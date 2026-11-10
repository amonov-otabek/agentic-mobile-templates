# Project Standards

Implementation rules for all code. Derived from the `react-native` (Expo) reference.
Mobile client project — pair with `context/ui-rules.md` + `context/ui-context.md` for UI
work and `context/api-context.md` for the consumed backend. Treat as locked.

## Engineering mindset

1. **Think first** — read the task block and relevant context before writing code.
2. **Scope is sacred** — build what the task says; nothing more, nothing less.
3. **Every change is verifiable** — if it can't be observed in the running app, it isn't done.
4. **Clean over clever** — boring, readable components win.
5. **One thing at a time** — never mix refactoring with feature work.
6. **Failures are expected** — handle loading/empty/error states as first-class flow.
7. **Fix root causes** — never patch symptoms or wrap broken code.

## TypeScript

- **File names (HARD):** PascalCase for pages, components, layouts and providers; camelCase for hooks (`useX.ts`) and every other module; lowercase/kebab-case folders; generated and framework-reserved files keep their names — full table in `context/ui-rules.md` → File naming.
- Strict mode (`strict: true`, via `expo/tsconfig.base`). No `any` — use precise types
  or `unknown` + narrowing. No `as` to silence errors.
- Use the path aliases (`@/*`, `@/assets/*`) — not deep relative `../../..` chains.
- Explicit return types on exported functions and hooks.
- Mirror backend contracts in `features/<name>/types/*` — keep them in sync with the
  API; never hand-duplicate a shared type in two places.
- Validate all external input (forms, deep-link params) with Zod at the boundary.

## Component structure

- **Function components only**, named exports. One component concern per file
  (`200–400` lines typical, `800` max — split when it grows).
- Props are typed interfaces; prefer composition over boolean-flag explosions.
- Use the shared `ui/*` primitives (`Button`, `Input`, `Field`, `Card`, `Text`,
  `Label`, `Checkbox`) — do not re-implement them. Variants come from `cva`.
- **No business logic in components.** A component renders props/state and dispatches
  to hooks; fetching, persistence, and multi-step flows live elsewhere (see below).
- React Compiler is on — do not add `useMemo`/`useCallback` purely for referential
  stability; add them only for genuinely expensive work.

## Hooks for logic

- **Server state → TanStack Query** in `queries/*`. Queries use stable `queryKey`
  factories (e.g. `authKeys`); mutations update the cache via `setQueryData` /
  `invalidateQueries`. Components consume these hooks, never `api/*` directly.
- **Side effects / orchestration → `services/*`** (token persistence, sign-out
  sequencing, 2FA challenge state). Services are framework-agnostic functions.
- **Client state → Zustand** in `store/*`, exposed via `createSelectors`
  (`useStore.use.field()`). Keep stores minimal; never mirror server data into them.

## Navigation patterns

- **File-based (Expo Router).** A route is a file under `src/app/`; route groups
  `(public)`/`(auth)`/`(protected)` model the auth boundary. Use `typedRoutes`.
- Route screens stay thin: import a feature `pages/*` component, render it in a layout.
- **Guards live in group `_layout.tsx`** and read the auth store status
  (`<Redirect href=… />`). Do not guard inside individual screens.
- Navigate with `useRouter()` (`push`/`replace`) and `<Link>`; pass dynamic data via
  route params, not module-level singletons (the in-memory 2FA token is the documented
  exception).

## Data fetching & errors

- All HTTP goes through `shared/libs/api.ts`; a failed envelope throws `ApiError`
  (`status`, `code`, `fields`). Surface `error.fields` back into form field errors and
  show a toast (`shared/hooks/useToast`) for the general message.
- Every async surface handles **loading / empty / error / success** explicitly — no
  silent spinners that never resolve. Disable controls while their action is pending
  (`Button` `loading` prop).
- Never swallow errors. Localize the user-facing message via i18n; never show a raw
  stack or backend internal.

## Validation

- **Zod** schemas in `validators/*`, one per input surface; reuse field schemas from
  `shared/libs/validators.ts`. Parse with `safeParse`, then `formatZodErrors` to the
  `{ field: string[] }` shape (matches the backend `error.fields`).

## Styling

- **Tokens only** (NativeWind classes bound to `global.css` theme tokens). Zero raw
  hex, pixel sizes, or magic spacing in `className`. Compose with `cn()`.
- Light/dark is automatic via theme tokens — never branch on color scheme to hardcode
  a color. (The rare SVG icon `color` prop is the documented exception, since NativeWind
  className can't drive SVG fill.)

## Accessibility

- Set `accessibilityRole` / `accessibilityLabel` / `accessibilityState` on interactive
  elements (the `Button`/`Checkbox` primitives already do — match the pattern).
- Touch targets ≥ 44×44 (primitives default to `h-11`); inputs have labels via `Field`.
- Respect reduced motion for Reanimated transitions; never convey state by color alone.

## i18n

- All user-facing strings resolve through `react-i18next` keys (`t('…')`) with entries
  in `shared/i18n/messages/{en,ru,uz}.json`. No hardcoded copy in JSX. Add a key to all
  three locale files when you add a string.

## Security

- Tokens only in `expo-secure-store` (native) via `shared/libs/storage.ts`; never in
  plain AsyncStorage or app state files. Never log tokens, passwords, or PII.
- `EXPO_PUBLIC_*` is bundled into the app — never put a secret there; it is for public
  config (the API base URL) only.

## Testing

- **Where tests live (HARD):** every test file goes in `test/` (`main`, `app`, `setup`, `helpers`, `e2e/`), at the path that
  mirrors its source file — see `context/architecture.md` → **Test structure**. Never beside a
  source file, never inside a module/feature folder. `setup` keeps the suite
  self-contained: tests run on a clean checkout with no developer `.env`.
- No runner is wired yet. When introduced, new behavior ships with a happy-path test
  plus at least one edge/error case; pre-existing tests stay green. Record the test
  command in `context/architecture.md`.

## Comments, TODOs, dependencies

- Comments explain *why*, where the code can't. No TODO/FIXME/HACK without a linked task.
- Prefer existing dependencies; a new one (especially native modules) requires developer
  approval — note it in QA notes and confirm Expo SDK 56 compatibility.
- Before using any third-party library, verify against the installed version's docs —
  RN/Expo APIs drift fast. Do not introduce a second lockfile.
