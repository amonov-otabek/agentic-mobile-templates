# Feature 05 — Payments & Invoices (Mobile)

- Feature ID: FEAT-05
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — guarded shell screen)

> Source of truth for the authenticated **payments** screen: a payments list with create +
> refund, and an invoices list with a pay action. Consumes the backend `payment` domain.
> TO-BUILD on an empty project — `lib/features/payment/` does not exist yet; this spec is the
> build-out plan, built like identity (FEAT-02) when scoped.

## Current state (to build)

There is no payments screen and no `lib/features/payment/` tree yet. This feature adds the
payments screen and its `data`/`domain`/`presentation` layers.

## Goal

Give authenticated users a payments screen: list payments (filter by status), create a payment,
refund a payment, and list + pay invoices — all through Riverpod notifiers + repositories against
the backend payment API.

## Scope

Feature-first layout `lib/features/payment/{data,domain,presentation}`.

- **Payments screen** — new `payments_screen` added to the protected `ShellRoute` (icon +
  localized label); thin route that composes the payments list, the create form, and the invoices
  list (sections or a `SegmentedButton`).
- **Payment list** (`presentation/widgets/payment_list`) — paginated list, filterable by `status`;
  per-row refund action (confirm via an `AlertDialog`); status badges via semantic theme tokens.
- **Payment create form** (`presentation/widgets/payment_create_form`) — validated amount/currency/
  description (form validators); create intent on the notifier.
- **Invoice list** (`presentation/widgets/invoice_list`) — paginated, filterable by `status`;
  per-row Pay action (confirm).
- **Providers** (`presentation/providers/payment_notifier`) — paginated payments notifier, create/
  refund intents, paginated invoices notifier, pay-invoice intent; mutations refresh/invalidate the
  relevant lists.
- **Data layer** — freezed DTOs (`Payment`, `Invoice`, `CreatePaymentDto`, `PaymentListQuery`,
  `InvoiceListQuery`), a retrofit datasource, and a repository impl mapping DTO → domain +
  exceptions → `Failure`.
- Loading / error / empty / success states; server field errors (`error.fields`) map onto inputs;
  amounts/currency formatted via `intl`.
- **Out of scope:** a real payment-gateway / native-pay integration, checkout/redirect flows,
  webhooks (server-side).

## Consumed API

Base `/api/v1`, envelope `{ success, data, meta? }`.

- `GET /payments?page&limit&status` → paginated `Payment[]` with `meta`.
- `POST /payments` → `Payment`.
- `POST /payments/:id/refund` (`{ amount }`) → `Payment`.
- `GET /invoices?page&limit&status` → paginated `Invoice[]`.
- `PATCH /invoices/:id/pay` → `Invoice`.

## UI / widgets used

- `features/payment/presentation/screens/payments_screen`, `presentation/widgets/{payment_list,
  payment_create_form,invoice_list}`.
- Shared `AppCard`, `AppTextField`/`AppField`, `AppButton`, a badge/status chip, an `AlertDialog`
  confirm, the async-state scaffold (loading/empty/`Failure`+retry), `SnackBar` feedback; the
  protected `ShellRoute` tab group.

## Acceptance criteria

- [ ] The payments tab renders behind the auth guard (unauthenticated → `/sign-in`).
- [ ] The payment list paginates and filters by `status` (counts from `meta`).
- [ ] Creating a payment validates the form, posts to `POST /payments`, refreshes the list, and
      shows success/error via `SnackBar`; server field errors map onto inputs.
- [ ] Refunding a payment confirms via dialog, calls `POST /payments/:id/refund`, and refetches.
- [ ] The invoice list paginates/filters by `status`; Pay calls `PATCH /invoices/:id/pay` and
      refetches.
- [ ] Loading, error, and empty states render on every list/form.
- [ ] All HTTP flows through the payment repository + retrofit datasource; responses typed via the
      freezed DTOs/entities; no `dio` in widgets.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only theming, a11y, dart analyze, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
