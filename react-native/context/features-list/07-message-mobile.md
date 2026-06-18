# Feature 07 — Messages (Mobile)

- Feature ID: FEAT-07
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — guarded tab screen)

> Source of truth for the authenticated **messages** screen: compose an outbound message
> (email/sms channel) and browse the message list with delivery status. Consumes the backend
> `message` domain (async delivery). **Scaffold today** — `features/message/*` holds only
> `.gitkeep` placeholders and `(protected)/messages.tsx` is a `ComingSoon` placeholder; build
> it like identity when scoped.

## Current state (scaffold)

`features/message/{api,components,pages,queries,types}/` contain `.gitkeep` only; the
`messages` tab screen renders `<ComingSoon>`. This spec is the build-out plan.

## Goal

Give authenticated users a messages screen: compose a message to a chosen channel and watch
its delivery status in a paginated list — all through TanStack Query against the backend
message API.

## Scope

- **Messages screen** — replace the `ComingSoon` placeholder at `(protected)/messages.tsx`
  with a thin route that renders `features/message/pages/messages-page` (compose form +
  message list sections).
- **Compose form** (`components/message-compose-form`) — Zod-validated
  (`validators/message.validator`): channel (`email` | `sms`), recipient, subject/body;
  create mutation enqueues an async delivery job server-side.
- **Message list** (`components/message-list`) — paginated `FlatList`, filterable by `status`;
  each row shows channel, recipient, and a delivery-status badge (queued/sent/failed).
- **Queries** (`queries/message.queries`) — `useMessages` (paginated), `useMessage` (detail),
  `useCreateMessage` (invalidates the list). Typed via `types/message.types` (`Message`,
  `CreateMessageDto`, `MessageListQuery`).
- **API** (`api/message.api`) — thin wrappers over `shared/libs/api`.
- Loading / error / empty / success states; status badges via semantic tokens; server field
  errors map onto inputs.
- Out of scope: live status streaming, provider/template configuration (server-side),
  threaded conversations.

## Consumed API

Base `/api/v1/messages`, envelope `{ success, data, meta? }`.

- `GET /messages?page&limit&status` → paginated `Message[]` with `meta`.
- `GET /messages/:id` → `Message`.
- `POST /messages` → `Message` (enqueues `email.jobs` / `sms.jobs` server-side).

## UI / components used

- `features/message/components/{message-compose-form,message-list}`,
  `features/message/pages/messages-page`.
- Shared `Card`, `Input`, `Field`, `Label`, `Button`, `Text`, a textarea (multiline `Input`),
  a select/picker for channel, a badge/status element, an empty/loading state;
  `shared/hooks/use-toast`; `Icons`; `(protected)` tab group.

## Acceptance criteria

- [ ] The messages tab renders behind the auth guard (unauthenticated → `/sign-in`).
- [ ] The compose form validates with Zod, posts to `POST /messages` with the chosen channel,
      invalidates the list, and shows success/error via toast; server field errors map onto
      inputs.
- [ ] The message list paginates and filters by `status` (counts from `meta`) and renders
      channel + delivery-status badges.
- [ ] Loading, error, and empty states render on the form and list.
- [ ] All HTTP flows through `api/message.api` + `queries/message.queries`; responses typed
      via `types/message.types`; no `fetch` in components.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
