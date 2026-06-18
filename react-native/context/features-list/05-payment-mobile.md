# Feature 05 — Payments & Invoices (Mobile)

- Feature ID: FEAT-05
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — guarded tab screen)

> Source of truth for the authenticated **payments** screen: a payments list with create +
> refund, and an invoices list with a pay action. Consumes the backend `payment` domain.
> **Scaffold today** — `features/payment/*` holds only `.gitkeep` placeholders; build it like
> identity when scoped.

## Current state (scaffold)

`features/payment/{api,components,pages,queries,types}/` contain `.gitkeep` only. There is no
payments tab screen yet. This spec is the build-out plan.

## Goal

Give authenticated users a payments screen: list payments (filter by status), create a
payment, refund a payment, and list + pay invoices — all through TanStack Query against the
backend payment API.

## Scope

- **Payments screen** — new `(protected)/payments.tsx` route file + `Tabs.Screen` entry (icon
  `Icons.payments`), thin: renders `features/payment/pages/payments-page`, which composes the
  payments list, the create form, and the invoices list (sections or a segmented control).
- **Payment list** (`components/payment-list`) — paginated `FlatList`, filterable by `status`;
  per-row refund action (confirm via a modal/dialog); status badges via semantic tokens.
- **Payment create form** (`components/payment-create-form`) — Zod-validated
  (`validators/payment.validator`) amount/currency/description; create mutation.
- **Invoice list** (`components/invoice-list`) — paginated, filterable by `status`; per-row
  Pay action (confirm).
- **Queries** (`queries/payment.queries`) — `usePayments`, `useCreatePayment`,
  `useRefundPayment`, `useInvoices`, `usePayInvoice`; mutations invalidate the relevant lists.
  Typed via `types/payment.types` (`Payment`, `Invoice`, `CreatePaymentDto`,
  `PaymentListQuery`, `InvoiceListQuery`).
- **API** (`api/payment.api`) — thin wrappers over `shared/libs/api`.
- Loading / error / empty / success states; server field errors map onto inputs.
- Out of scope: a real payment-gateway/native-pay integration, checkout/redirect flows,
  webhooks (server-side).

## Consumed API

Base `/api/v1`, envelope `{ success, data, meta? }`.

- `GET /payments?page&limit&status` → paginated `Payment[]` with `meta`.
- `POST /payments` → `Payment`.
- `POST /payments/:id/refund` (`{ amount }`) → `Payment`.
- `GET /invoices?page&limit&status` → paginated `Invoice[]`.
- `PATCH /invoices/:id/pay` → `Invoice`.

## UI / components used

- `features/payment/components/{payment-list,payment-create-form,invoice-list}`,
  `features/payment/pages/payments-page`.
- Shared `Card`, `Input`, `Field`, `Label`, `Button`, `Text`, a badge/status element, a
  confirm modal, an empty/loading state; `shared/hooks/use-toast`; `Icons`; `(protected)`
  tab group.

## Acceptance criteria

- [ ] The payments tab renders behind the auth guard (unauthenticated → `/sign-in`).
- [ ] The payment list paginates and filters by `status` (counts from `meta`).
- [ ] Creating a payment validates with Zod, posts to `POST /payments`, invalidates the list,
      and shows success/error via toast; server field errors map onto inputs.
- [ ] Refunding a payment confirms, calls `POST /payments/:id/refund`, and refetches.
- [ ] The invoice list paginates/filters by `status`; Pay calls `PATCH /invoices/:id/pay` and
      refetches.
- [ ] Loading, error, and empty states render on every list/form.
- [ ] All HTTP flows through `api/payment.api` + `queries/payment.queries`; responses typed
      via `types/payment.types`; no `fetch` in components.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
