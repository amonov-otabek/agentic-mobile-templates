# Project Overview

A concise, high-level summary of this project. Every agent reads this before any
feature work. Derived from the `react-native` (Expo) mobile template.

## What we are building

A production-ready **React Native + Expo + TypeScript mobile app**. A single
cross-platform client (iOS + Android, with a secondary web target) that talks to the
backend monolith API over HTTP. The UI is feature-based: each domain (identity,
notification, payment, message, audit, analytic) is an isolated folder under
`src/features/`, and navigation is file-based via Expo Router.

## Goals

1. Ship a hardened mobile foundation: auth with secure token storage, route guards,
   typed API client, server-state caching, theming (light/dark), i18n, and a
   reusable component kit — production-grade from the first commit.
2. Keep domain features isolated so they can be developed, tested, and reasoned about
   independently while shipping as one app.

## Platforms

| Surface | Tech | Who uses it |
| --- | --- | --- |
| iOS | Expo / React Native (native build) | end users |
| Android | Expo / React Native (native build) | end users |
| Web (secondary) | `react-native-web` (Expo static output) | preview / fallback |

## Users & roles

| Role | Description |
| --- | --- |
| user | standard authenticated account (own profile, sessions, devices, 2FA) |
| admin | elevated access — admin user list / role management screens |

Roles come from the backend (`PublicUser.role`); the app reflects them in routing
and conditional UI, it does not own RBAC.

## Core flows

1. App launch → `AuthProvider` hydrates the stored token from secure storage and
   seeds the auth store status; `app/index.tsx` redirects to `/dashboard` (authed)
   or `/home` (guest).
2. A guest signs in / signs up (email or phone + password), optionally clears a
   2FA challenge, and receives a JWT access + refresh token pair (stored in
   `expo-secure-store`).
3. The `(protected)` route group is guarded by the auth store — unauthenticated
   users are redirected to `/sign-in`; authed users see the tab navigator.
4. Screens call feature query hooks (TanStack Query) → services → the typed API
   client; results render through the shared component kit.

## Auth & identity (implemented)

The **identity** feature is the only fully-built domain so far: sign-in, sign-up,
forgot/reset password, email verification, two-factor challenge, plus profile,
sessions, devices, and admin query/API layers. It backs the auth store, route
guards, and the `(auth)` / `(protected)` route groups.

## Planned domains (scaffolds only)

`notification`, `payment`, `message`, `audit`, `analytic` exist as empty
feature folders (`api/ components/ pages/ queries/ types/` with `.gitkeep`) and
placeholder `(protected)` screens (`messages`, `notifications`, `users`). They are
**not implemented** — build them like identity when scoped.

## Key constraints

- Tokens live in `expo-secure-store` (Keychain / Keystore) on native; `localStorage`
  on web. The access token is also held in memory for request headers.
- `EXPO_PUBLIC_API_URL` is inlined at build time — a physical device needs a LAN IP,
  not `localhost`.
- All user-facing strings localized through i18n (`en` / `ru` / `uz`).
- Visual values come only from theme tokens (NativeWind / Tailwind) — see
  `context/ui-context.md`.

## In scope (this milestone)

- Feature areas become `context/features-list/` specs; identity is the reference.
  Planned domains follow the same shape.

## Out of scope

- The backend API itself (consumed, not built here — see `context/api-context.md`).
- Native module development beyond the Expo managed workflow / config plugins listed
  in `app.json`.
