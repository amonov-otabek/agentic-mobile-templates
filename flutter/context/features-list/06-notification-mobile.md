# Feature 06 — Notifications & Push (Mobile)

- Feature ID: FEAT-06
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — guarded shell screen)

> Source of truth for the authenticated **notifications** screen: an in-app feed with unread
> tracking + mark-read actions, a delivery-preferences form, and device push registration.
> Consumes the backend `notification` domain. TO-BUILD on an empty project —
> `lib/features/notification/` does not exist yet; this spec is the build-out plan, built like
> identity (FEAT-02) when scoped.

## Current state (to build)

There is no notifications screen and no `lib/features/notification/` tree yet. The protected shell
reserves a notifications tab (FEAT-01); this feature fills it and adds the `data`/`domain`/
`presentation` layers.

## Goal

Give authenticated users a notifications screen: read the feed, see the unread count, mark one or
all read, manage delivery preferences, and (mobile-specific) register the device for push — all
through Riverpod notifiers + repositories against the backend notification API.

## Scope

Feature-first layout `lib/features/notification/{data,domain,presentation}`.

- **Notifications screen** — `notifications_screen` rendered by the protected `ShellRoute`
  notifications tab; composes the feed + preferences sections.
- **Notification feed** (`presentation/widgets/notification_list`) — paginated list with an
  unread-only filter; per-item Mark-read and a Mark-all-read action; visual read/unread state via
  theme tokens.
- **Unread count** — an `unreadCountProvider` drives a `NavigationBar` tab badge and refreshes
  after mark-read.
- **Preferences form** (`presentation/widgets/preferences_form`) — channel/category toggles
  (`Switch`/`Checkbox`); loads current preferences and persists via `PUT` (optimistic update where
  safe, reverting on error).
- **Push registration** (`data/datasources/push_service` + `presentation/widgets/push_toggle`) —
  request permission and obtain a device push token via an approved package (e.g.
  `firebase_messaging`, or `flutter_local_notifications` for local display) — a new pub dep +
  native config (iOS APNs / Android FCM) requires developer approval (note it in QA notes); then
  register the token with the backend; handle a foreground message listener that pushes into
  Riverpod state (never directly into widgets).
- **Providers** (`presentation/providers/notification_notifier`) — paginated feed notifier, unread
  count, preferences load/update intents, mark-read / mark-all-read intents; mutations refresh the
  feed + unread count.
- **Data layer** — freezed DTOs (`AppNotification`, `NotificationPreferences`,
  `NotificationListQuery`), a retrofit datasource, and a repository impl mapping DTO → domain +
  exceptions → `Failure`.
- Loading / error / empty / success states.
- **Out of scope:** rich notification deep links, in-app real-time socket delivery (push covers the
  real-time path on mobile).

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

## UI / widgets used

- `features/notification/presentation/screens/notifications_screen`,
  `presentation/widgets/{notification_list,preferences_form,push_toggle}`,
  `data/datasources/push_service`.
- Shared `AppCard`, `AppButton`, `Switch`/`Checkbox`, a badge/status chip, the async-state scaffold
  (loading/empty/`Failure`+retry), `SnackBar` feedback; the protected `ShellRoute` tab group.

## Acceptance criteria

- [ ] The notifications tab renders behind the auth guard (unauthenticated → `/sign-in`).
- [ ] The feed paginates, supports an `unread_only` filter, and shows read/unread state.
- [ ] The unread-count tab badge reflects `GET /notifications/unread-count` and updates after
      mark-read / mark-all-read.
- [ ] Mark-read and mark-all-read call their endpoints and refresh the feed + unread count.
- [ ] The preferences form loads current prefs and persists via `PUT`; success/error `SnackBar`;
      optimistic update reverts on error.
- [ ] Push registration requests permission, obtains a device push token, and registers it with the
      backend; permission-denied is handled gracefully (no crash, clear UI state).
- [ ] Loading, error, and empty states render.
- [ ] All HTTP flows through the notification repository + retrofit datasource; responses typed via
      the freezed DTOs/entities; no `dio` in widgets.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only theming, a11y, dart analyze, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
