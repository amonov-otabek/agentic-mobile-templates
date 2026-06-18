# Feature 07 — Messages (Mobile)

- Feature ID: FEAT-07
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — guarded shell screen)

> Source of truth for the authenticated **messages** screen: compose an outbound message
> (email/sms channel) and browse the message list with delivery status. Consumes the backend
> `message` domain (async delivery). TO-BUILD on an empty project — `lib/features/message/` does
> not exist yet; this spec is the build-out plan, built like identity (FEAT-02) when scoped.

## Current state (to build)

There is no messages screen and no `lib/features/message/` tree yet. The protected shell reserves
a messages tab (FEAT-01); this feature fills it and adds the `data`/`domain`/`presentation` layers.

## Goal

Give authenticated users a messages screen: compose a message to a chosen channel and watch its
delivery status in a paginated list — all through Riverpod notifiers + repositories against the
backend message API.

## Scope

Feature-first layout `lib/features/message/{data,domain,presentation}`.

- **Messages screen** — `messages_screen` rendered by the protected `ShellRoute` messages tab;
  composes the compose form + message list sections.
- **Compose form** (`presentation/widgets/message_compose_form`) — validated channel
  (`email` | `sms`), recipient, subject/body; the create intent enqueues an async delivery job
  server-side.
- **Message list** (`presentation/widgets/message_list`) — paginated list, filterable by `status`;
  each row shows channel, recipient, and a delivery-status badge (queued/sent/failed) via semantic
  theme tokens.
- **Providers** (`presentation/providers/message_notifier`) — paginated messages notifier, a detail
  provider, and a create-message intent (refreshes/invalidates the list).
- **Data layer** — freezed DTOs (`Message`, `CreateMessageDto`, `MessageListQuery`), a retrofit
  datasource, and a repository impl mapping DTO → domain + exceptions → `Failure`.
- Loading / error / empty / success states; server field errors (`error.fields`) map onto inputs;
  timestamps formatted via `intl`.
- **Out of scope:** live status streaming, provider/template configuration (server-side), threaded
  conversations.

## Consumed API

Base `/api/v1/messages`, envelope `{ success, data, meta? }`.

- `GET /messages?page&limit&status` → paginated `Message[]` with `meta`.
- `GET /messages/:id` → `Message`.
- `POST /messages` → `Message` (enqueues `email.jobs` / `sms.jobs` server-side).

## UI / widgets used

- `features/message/presentation/screens/messages_screen`, `presentation/widgets/{message_compose_form,
  message_list}`.
- Shared `AppCard`, `AppTextField`/`AppField` (incl. a multiline body field), `AppButton`, a
  channel `DropdownButton`/picker, a badge/status chip, the async-state scaffold
  (loading/empty/`Failure`+retry), `SnackBar` feedback; the protected `ShellRoute` tab group.

## Acceptance criteria

- [ ] The messages tab renders behind the auth guard (unauthenticated → `/sign-in`).
- [ ] The compose form validates, posts to `POST /messages` with the chosen channel, refreshes the
      list, and shows success/error via `SnackBar`; server field errors map onto inputs.
- [ ] The message list paginates and filters by `status` (counts from `meta`) and renders channel +
      delivery-status badges.
- [ ] Loading, error, and empty states render on the form and list.
- [ ] All HTTP flows through the message repository + retrofit datasource; responses typed via the
      freezed DTOs/entities; no `dio` in widgets.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only theming, a11y, dart analyze, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
