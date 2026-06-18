# Feature 06 — Notifications & Push (Mobile)

- Feature ID: FEAT-06
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — guarded tab screen)

> Source of truth for the authenticated **notifications** screen: an in-app feed with unread
> tracking + mark-read actions, a delivery-preferences form, and device push registration.
> Consumes the backend `notification` domain. **Scaffold today** — `features/notification/*`
> holds only `.gitkeep` placeholders and `(protected)/notifications.tsx` is a `ComingSoon`
> placeholder; build it like identity when scoped.

## Current state (scaffold)

`features/notification/{api,components,pages,queries,types}/` contain `.gitkeep` only; the
`notifications` tab screen renders `<ComingSoon>`. This spec is the build-out plan.

## Goal

Give authenticated users a notifications screen: read the feed, see the unread count, mark
one or all read, manage delivery preferences, and (mobile-specific) register the device for
push — all through TanStack Query against the backend notification API.

## Scope

- **Notifications screen** — replace the `ComingSoon` placeholder at
  `(protected)/notifications.tsx` with a thin route that renders
  `features/notification/pages/notifications-page` (feed + preferences sections).
- **Notification feed** (`components/notification-list`) — paginated `FlatList` with an
  unread-only filter; per-item Mark-read and a Mark-all-read action; visual read/unread state.
- **Unread count** — `useUnreadCount` drives a tab badge and refreshes after mark-read.
- **Preferences form** (`components/preferences-form`) — channel/category toggles (`Switch`/
  `Checkbox`); loads current preferences and persists via `PUT` (optimistic update where safe).
- **Push registration** (`services/push.service` + `components/push-toggle`) — request
  permission and obtain an Expo push token via `expo-notifications` (confirm Expo SDK 56
  compatibility + add the config plugin in `app.json`; get developer approval for the native
  dep), then register the token with the backend; handle a foreground notification listener.
- **Queries** (`queries/notification.queries`) — `useNotifications`, `useUnreadCount`,
  `usePreferences`, `useUpdatePreferences`, `useMarkRead`, `useMarkAllRead`; mutations
  invalidate the feed + unread count. Typed via `types/notification.types` (`AppNotification`,
  `NotificationPreferences`, `NotificationListQuery`).
- **API** (`api/notification.api`) — thin wrappers over `shared/libs/api`.
- Loading / error / empty / success states.
- Out of scope: rich notification deep links, in-app real-time socket delivery (push covers
  the real-time path on mobile).

## Consumed API

Base `/api/v1/notifications`, envelope `{ success, data, meta? }`.

- `GET /notifications?page&limit&unread_only` → paginated `AppNotification[]` with `meta`.
- `GET /notifications/unread-count` → `{ unread }`.
- `GET /notifications/preferences` → `NotificationPreferences`.
- `PUT /notifications/preferences` → `NotificationPreferences`.
- `PATCH /notifications/:id/read` → `AppNotification`.
- `PATCH /notifications/read-all` → `{ updated }`.
- Push token registration endpoint (confirm exact path with the backend, e.g.
  `POST /notifications/devices` / `POST /devices` with `{ push_token, platform }`).

## UI / components used

- `features/notification/components/{notification-list,preferences-form,push-toggle}`,
  `features/notification/pages/notifications-page`, `features/notification/services/push.service`.
- Shared `Card`, `Text`, `Button`, `Checkbox`, a Switch primitive (add if missing), a
  badge/status element, an empty/loading state; `shared/hooks/use-toast`; `Icons`;
  `(protected)` tab group.

## Acceptance criteria

- [ ] The notifications tab renders behind the auth guard (unauthenticated → `/sign-in`).
- [ ] The feed paginates, supports an `unread_only` filter, and shows read/unread state.
- [ ] The unread-count tab badge reflects `GET /notifications/unread-count` and updates after
      mark-read / mark-all-read.
- [ ] Mark-read and mark-all-read call their endpoints and invalidate the feed + unread count.
- [ ] The preferences form loads current prefs and persists via `PUT`; success/error toast;
      optimistic update reverts on error.
- [ ] Push registration requests permission, obtains an Expo push token, and registers it with
      the backend; permission-denied is handled gracefully (no crash, clear UI state).
- [ ] Loading, error, and empty states render.
- [ ] All HTTP flows through `api/notification.api` + `queries/notification.queries`; responses
      typed via `types/notification.types`; no `fetch` in components.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
