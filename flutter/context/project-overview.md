# Project Overview

A concise, high-level summary of this project. Every agent reads this before any
feature work. This is the **planned foundation for a currently empty project** —
no Flutter code exists yet; this document defines the canonical stack and shape we
are building toward. Sibling to the `react-native` mobile template; both consume
the same backend API.

## What we are building

A production-grade **Flutter (stable) + Dart 3 mobile app** for iOS and Android. A
single codebase, feature-first, that talks to the shared backend monolith API over
HTTPS. Material 3 design, Riverpod for state, `go_router` for navigation, a typed
`dio` + `retrofit` client for the API, `freezed` immutable models, and
`flutter_secure_storage` for JWT tokens.

## Goals

1. Ship a hardened mobile foundation: auth (login/register/2FA/refresh), session
   persistence, typed API access, navigation with route guards, theming, i18n, and
   error/empty/loading state handling — production-grade from the first commit.
2. Keep features isolated (`lib/features/<name>/{data,domain,presentation}`) so they
   can be developed, tested, and reasoned about independently inside one app.

## Platforms

| Surface | Tech | Who uses it |
| --- | --- | --- |
| iOS | Flutter (stable), Material 3, Cupertino-aware widgets | end users on iPhone/iPad |
| Android | Flutter (stable), Material 3 | end users on Android phones/tablets |
| Backend it consumes | the shared monolith API (`/api/v1`) over HTTPS | the app, via `dio`/`retrofit` |

The app is a **client only** — it owns no database and no server. All persistence
beyond local cache/tokens lives behind the backend API.

## Core flows

1. A user authenticates (email/phone + password, OAuth, or 2FA) and the app stores
   the returned JWT access + refresh token pair in `flutter_secure_storage`.
2. Authenticated screens call repositories; a `dio` interceptor attaches the bearer
   token, and on `401` transparently refreshes via the refresh token (single-flight)
   and retries — or routes to login if the refresh fails.
3. Screens render off immutable state held in Riverpod notifiers; user actions call
   notifier methods → repositories → the API; results update state; widgets rebuild.
4. Navigation runs through `go_router` with a redirect that gates protected routes on
   auth state and deep-links into the right screen.

## Roles

| Role | Description |
| --- | --- |
| user | standard authenticated account |
| admin | elevated access; admin-only screens shown when the role permits |
| _(guest)_ | unauthenticated; limited to auth + public screens |

Role/permission data comes from the backend on the authenticated profile; the app
gates UI, the API enforces authorization.

## Key constraints

- Tokens live ONLY in `flutter_secure_storage` (never in plain prefs, never in code).
- All user-facing strings localized through `intl`/`flutter_localizations` — no
  hardcoded copy in widgets.
- The app trusts no response shape — every API payload is parsed through `freezed` /
  `json_serializable` models; malformed payloads surface as errors, not crashes.
- Every async surface handles loading, empty, error, and success states explicitly.
- Light and dark themes both ship from day one, driven entirely by design tokens.

## In scope (this milestone)

- Feature areas become `context/features-list/` specs. The first is **01-setup-app**
  (Flutter project + folder skeleton + core network/theme/router/storage + env +
  CI/lint). Auth and domain features follow, mirroring the backend's modules.

## Out of scope

- Backend/server work — that lives in the `backends/*` templates and is consumed,
  not built, here.
- Web/desktop Flutter targets — this milestone is iOS + Android only.
