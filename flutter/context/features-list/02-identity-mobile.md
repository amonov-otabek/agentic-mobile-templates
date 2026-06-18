# Feature 02 — Identity & Auth (Mobile)

- Feature ID: FEAT-02
- Status: draft
- Depends on: FEAT-01 (scaffold: dio/retrofit API client, secure `TokenStore`, ProviderScope, go_router shell + redirect guard, theme, i18n)

> Source of truth for the **mobile identity layer**: the full auth screen surface, session +
> token handling via `flutter_secure_storage`, the auth-state notifier that drives the
> `go_router` redirect, and the profile/settings/admin data + screens. It wires
> `lib/features/identity/*` to the backend identity API; it builds no backend. TO-BUILD on an
> empty project — every screen and layer below is planned, not yet written.

## Goal

Make sign-in / sign-up and the auth surface work end-to-end against the backend `/api/v1`
identity API on device: token persistence in secure storage with rehydration on launch,
the auth-state notifier feeding the route guard, email verification, password reset, and a 2FA
challenge — then build the profile/settings and admin screens on top of the same repository/
provider layers.

## Scope

Feature-first layout: `lib/features/identity/{data,domain,presentation}` per
`context/architecture.md` (data: freezed DTOs + retrofit datasources + repository impls;
domain: entities + repository interfaces; presentation: Riverpod notifiers + screens + widgets).

### Auth flow (screens + notifiers + routes)

- **Sign-in** (`presentation/screens/sign_in_screen` + `sign_in_form`) — identifier (email/phone)
  + password, `remember_me`, `trust_device`; on `requires_two_fa` the notifier stashes the pending
  token in memory (not persisted) and the router navigates to `/two-factor`.
- **Sign-up** (`sign_up_screen` + `sign_up_form`) — first/last name + email + password +
  terms/privacy consent.
- **Verify email** (`verify_email_screen`) — consume an email-link token (`/auth/verify-email-link`).
- **Forgot password** (`forgot_password_screen` + form) — request a reset email.
- **Reset password** (`reset_password_screen` + form) — submit token + new password.
- **Two-factor** (`two_factor_screen` + form) — complete the challenge with the in-memory pending
  token (`/2fa/verify` → tokens).
- **Dashboard** (protected shell home, `dashboard_screen`) — greets the current user
  (`currentUserProvider`) and signs out (clears tokens + state → router redirects to `/sign-in`).

### Session, token, and state handling

- **Auth repository** (`data/repositories/auth_repository_impl`) — sign-in/up/refresh/logout/
  verify/forgot/reset via retrofit datasources; persists the access + refresh pair through the
  `TokenStore` (`flutter_secure_storage`); maps exceptions → `Failure`.
- **Auth-state notifier** (`presentation/providers/auth_notifier`, Riverpod) — immutable `freezed`
  state (`status: unknown | authenticated | unauthenticated`, `user`, `pendingTwoFa`); hydrates
  from `TokenStore` on launch (the router waits on a non-`unknown` status before first paint);
  exposes sign-in / sign-up / verify-2FA / sign-out intents. The `go_router` redirect reads this
  notifier's `status`.
- **Validators** — Zod-equivalent validation in Dart: form/field validators (or a small schema
  helper) for sign-in/up, forgot/reset, two-factor, update-profile, change-password, 2FA-enable
  code. Server field errors (`error.fields`) map onto the matching inputs.

### Profile / settings / admin (screens on the same layers)

- **Settings screen** (`settings_screen` in the protected shell) + widgets: profile edit
  (`PATCH /profile/me`), password change (`PATCH /auth/password`), 2FA setup/enable/disable
  (`GET /2fa/status` → `POST /2fa/setup` (QR) → `POST /2fa/enable` (backup codes) →
  `DELETE /2fa/disable`), active-session list + revoke / revoke-all
  (`GET /sessions`, `DELETE /sessions/:id`, `DELETE /sessions`).
- **Admin users screen** (`admin_users_screen`) + widgets: paginated user list (`GET /admin/users`),
  toggle-active, role assignment (`GET /admin/roles` + `PATCH /admin/users/:id/role`), delete.
  The tab/route is gated on `user.role == 'admin'` (mirror the router guard); the API enforces it.
- **Out of scope:** OAuth providers, devices/security center, push registration (FEAT-06). The
  refresh interceptor itself ships in FEAT-01; this feature provides the refresh repository call
  it invokes.

## Consumed API

Base `/api/v1`, envelope `{ success, data, meta? }`. Branch on `error.code` / `error.fields`
(messages are i18n-localized server-side).

- **Auth:** `POST /auth/sign-in` (2FA → `requires_two_fa` + `pending_token`, else `tokens`)
  · `POST /auth/sign-up` · `POST /auth/refresh` (`{ refresh_token }` → rotated pair)
  · `POST /auth/logout` (`{ refresh_token }`).
- **Verify / password:** `POST /auth/verify-email-link` · `POST /auth/forgot-password`
  · `POST /auth/reset-password` · `PATCH /auth/password`.
- **2FA:** `GET /2fa/status` · `POST /2fa/setup` (secret + QR) · `POST /2fa/enable` (backup
  codes) · `POST /2fa/verify` (challenge → tokens) · `DELETE /2fa/disable`.
- **Profile:** `GET /profile/me` · `PATCH /profile/me`.
- **Sessions:** `GET /sessions?limit=50` · `DELETE /sessions/:id` · `DELETE /sessions`.
- **Admin (role-gated):** `GET /admin/users?page&limit` · `GET /admin/users/:id`
  · `PATCH /admin/users/:id/toggle-active` · `PATCH /admin/users/:id/role` (`{ role_id }`)
  · `DELETE /admin/users/:id` · `GET /admin/roles`.

Tokens persisted only in `flutter_secure_storage`; the bearer + single-flight refresh are owned
by the FEAT-01 interceptor.

## UI / widgets used

- `features/identity/presentation/screens/*` (`sign_in_screen`, `sign_up_screen`,
  `verify_email_screen`, `forgot_password_screen`, `reset_password_screen`, `two_factor_screen`,
  `dashboard_screen`, `settings_screen`, `admin_users_screen`) and feature-local widgets
  (forms, `two_factor_card`, `sessions_list`, `users_list`, `roles_picker`).
- Shared kit (FEAT-01): `AppButton`, `AppTextField`/`AppField`, `AppCard`, the async-state
  scaffold (loading/empty/`Failure`+retry), `SnackBar`/toast feedback; the auth flow + protected
  `ShellRoute` route groups.

## Acceptance criteria

- [ ] Sign-up and sign-in work against the live API; the access + refresh pair is persisted in
      `flutter_secure_storage` and rehydrated on launch (the auth notifier resolves status before
      the router's first redirect/paint).
- [ ] On `requires_two_fa`, the pending token is held in memory (not persisted) and `/two-factor`
      completes auth (`/2fa/verify` → tokens).
- [ ] The `go_router` redirect sends unauthenticated users to `/sign-in` and keeps authed users
      out of the auth flow; the dashboard is the protected shell home.
- [ ] Email-link verification, forgot → reset, and (settings) change-password flows work.
- [ ] Settings screen edits the profile, lists + revokes sessions (single + all), and runs the full
      2FA cycle: status → setup (QR) → enable (backup codes) → disable.
- [ ] Admin users screen lists (paginated from `meta`), toggles active, assigns a role, and deletes
      — visible only when `user.role == 'admin'`; non-admins cannot reach the route.
- [ ] Server field errors (`error.fields`) map onto the right form inputs; assertions branch on
      `error.code`, never localized message text.
- [ ] Sign-out clears `flutter_secure_storage` + the auth notifier state and the router redirects to
      `/sign-in`.
- [ ] All copy + validation errors localized (en/ru/uz); auth state lives in the Riverpod notifier;
      mutations refresh/invalidate the relevant providers; all I/O flows through repositories +
      retrofit datasources (no `dio` in widgets).
- [ ] Tokens-only theming, a11y, dart analyze, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
