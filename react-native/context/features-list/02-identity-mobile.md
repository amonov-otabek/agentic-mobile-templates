# Feature 02 — Identity & Auth (Mobile)

- Feature ID: FEAT-02
- Status: draft
- Depends on: FEAT-01 (scaffold: API client, secure storage, providers, route groups, theme, i18n)

> Source of truth for the **mobile identity layer**: the full auth screen surface, session +
> token handling via `expo-secure-store`, the auth store + route guards, and the
> profile/settings/admin data layers. It wires `features/identity/*` to the backend identity
> API; it builds no backend. **The only fully-built feature in the template** — the auth flow
> screens are real; settings and admin screens are still placeholders consuming an
> already-wired query/api layer (see "Built vs to-build" below).

## Goal

Make sign-in / sign-up and the auth surface work end-to-end against the backend `/api/v1`
identity API on device: token persistence in secure storage with rehydration on launch,
guarded route groups, email verification, password reset, and a 2FA challenge — then build
out the profile/settings and admin screens on top of the data layers that already ship.

## Scope

### Built today (real screens + flows)

- **Auth screens** (`(auth)` group → `features/identity/pages/*` rendered inside `auth-layout`):
  - `sign-in-page` / `sign-in-form` — identifier (email/phone) + password, `remember_me`,
    `trust_device`; on `requires_two_fa` the service stashes the pending token in memory and
    the screen routes to `/two-factor`.
  - `sign-up-page` / `sign-up-form` — first/last name + email + password + terms/privacy
    consent.
  - `verify-email-page` / `verify-email-view` — consume an email-link token
    (`/auth/verify-email-link`).
  - `forgot-password-page` / `forgot-password-form` — request a reset email.
  - `reset-password-page` / `reset-password-form` — submit token + new password.
  - `two-factor-page` / `two-factor-form` — complete the challenge with the in-memory
    pending token.
- **Dashboard screen** (`(protected)/dashboard`) — greets the current user (`useMe`) and
  signs out (`useSignOut` → `router.replace('/home')`).
- **Session / token handling** (`services/auth.service.ts`) — persist access + refresh in
  `expo-secure-store` via `shared/libs/storage.ts`; hydrate via `loadSession()` (awaited by
  `AuthProvider` before first paint); in-memory access bearer + in-memory refresh; in-memory
  pending-2FA token across the sign-in → two-factor navigation; `signOut()` calls
  `/auth/logout` then clears tokens.
- **Client auth state** (`store/auth.store.ts`, Zustand + `createSelectors`) — `user`,
  `status` (`idle | authenticated | unauthenticated`), `setUser`, `setStatus`, `clear`;
  selector hooks (`useAuthStore.use.status()`). Route guards in the group `_layout.tsx`
  files read `status`.
- **Validators** (`validators/auth.validator.ts`, Zod) — sign-in/up, forgot/reset, two-factor,
  update-profile, change-password, 2FA-enable code; server field errors (`error.fields`) map
  onto inputs.
- **Data layers already wired** (consumed by the built screens above, and by the
  to-build screens below):
  - `queries/auth.queries.ts` — `useMe`, `useSignIn`, `useSignUp`, `useSignOut`,
    `useVerifyTwoFactor`, `useForgotPassword`, `useResetPassword`, `useVerifyEmail`.
  - `queries/settings.queries.ts` — `useUpdateProfile`, `useChangePassword`, `useSessions`,
    `useRevokeSession`, `useRevokeAllSessions`, `useTwoFaStatus`, `useSetupTwoFa`,
    `useEnableTwoFa`, `useDisableTwoFa`.
  - `queries/admin.queries.ts` — `useAdminUsers` (paginated), `useRoles`,
    `useToggleUserStatus`, `useAssignRole`, `useDeleteUser`.
  - `api/{auth,profile,session,twofa,admin}.api.ts` thin wrappers over the typed client.

### To build on top (screens for the already-wired data layers)

- **Settings screen** (`(protected)/settings` — currently a `ComingSoon` placeholder) →
  new `features/identity/pages/settings-page` + components: profile edit (`useUpdateProfile`),
  password change (`useChangePassword`), 2FA setup/enable/disable (`useTwoFaStatus` →
  `useSetupTwoFa` (QR) → `useEnableTwoFa` (backup codes) → `useDisableTwoFa`), active-session
  list + revoke / revoke-all (`useSessions` / `useRevokeSession` / `useRevokeAllSessions`).
- **Admin users screen** (`(protected)/users` — currently a `ComingSoon` placeholder) →
  new `features/identity/pages/users-page` + components: paginated user list (`useAdminUsers`),
  toggle-active (`useToggleUserStatus`), role assignment (`useRoles` + `useAssignRole`),
  delete (`useDeleteUser`). Gate the tab/screen on `PublicUser.role === 'admin'`.
- Out of scope: refresh-token auto-rotation interceptor on `401` (the plumbing exists; the
  silent-retry interceptor is a follow-up), OAuth providers, devices/security center, push
  registration (FEAT-06).

## Consumed API

Base `/api/v1`, envelope `{ success, data, meta? }`. Branch on `error.code` / `error.fields`
(messages are i18n-localized server-side).

- **Auth:** `POST /auth/sign-in` (2FA → `requires_two_fa` + `pending_token`, else `tokens`)
  · `POST /auth/sign-up` · `POST /auth/refresh` (`{ refresh_token }` → rotated pair)
  · `POST /auth/logout` (`{ refresh_token }`).
- **Verify / password:** `POST /auth/verify-email-link` · `POST /auth/forgot-password`
  · `POST /auth/reset-password` · `PATCH /auth/password`.
- **2FA:** `GET /2fa/status` · `POST /2fa/setup` (secret + QR) · `POST /2fa/enable`
  (backup codes) · `POST /2fa/verify` (challenge → `AuthResponse`) · `DELETE /2fa/disable`.
- **Profile:** `GET /profile/me` · `PATCH /profile/me`.
- **Sessions:** `GET /sessions?limit=50` · `DELETE /sessions/:id` · `DELETE /sessions`.
- **Admin (role-gated):** `GET /admin/users?page&limit` · `GET /admin/users/:id`
  · `PATCH /admin/users/:id/toggle-active` · `PATCH /admin/users/:id/role` (`{ role_id }`)
  · `DELETE /admin/users/:id` · `GET /admin/roles`.

## UI / components used

- Built: `features/identity/components/*` (`auth-provider`, `sign-in-form`, `sign-up-form`,
  `two-factor-form`, `forgot-password-form`, `reset-password-form`, `verify-email-view`);
  `layout/auth-layout`.
- To build: `settings-form`/`password-form`/`two-factor-card`/`sessions-list`,
  `users-list`/`roles-picker` under `features/identity/components/*`.
- Shared primitives: `Button`, `Input`, `Field`, `Label`, `Card`, `Text`, `Checkbox`,
  `Toaster`; `shared/hooks/use-toast`; `Icons`; `(auth)` / `(protected)` route groups.

## Acceptance criteria

- [ ] Sign-up and sign-in work against the live API; the access token is in memory and the
      access + refresh pair is persisted in `expo-secure-store` and rehydrated on launch
      (`loadSession()` in `AuthProvider` before first paint).
- [ ] On `requires_two_fa`, the pending token is stashed in memory and `/two-factor`
      completes auth (`/2fa/verify` → tokens).
- [ ] `(protected)` redirects unauthenticated users to `/sign-in`; `(auth)` redirects authed
      users to `/dashboard`; `index.tsx` routes by status.
- [ ] Email-link verification, forgot → reset, and (settings) change-password flows work.
- [ ] Settings screen edits profile, lists + revokes sessions (single + all), and runs the
      full 2FA cycle: status → setup (QR) → enable (backup codes) → disable.
- [ ] Admin users screen lists (paginated), toggles active, assigns a role, and deletes —
      visible only to `role === 'admin'`; non-admins cannot reach it.
- [ ] Server field errors (`error.fields`) map onto the right form inputs; assertions branch
      on `error.code`, never message text.
- [ ] Sign-out calls `/auth/logout`, clears secure storage + the store, and returns to `/home`.
- [ ] All copy + validation errors localized (en/ru/uz); auth state lives in the Zustand
      store; mutations invalidate the relevant query keys.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
