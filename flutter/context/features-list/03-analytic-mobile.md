# Feature 03 — Analytics Dashboard (Mobile)

- Feature ID: FEAT-03
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — this is a guarded shell screen)

> Source of truth for the authenticated **analytics** screen: headline stat cards plus a
> paginated event list (with an optional chart). Consumes the backend `analytic` domain.
> TO-BUILD on an empty project — `lib/features/analytic/` does not exist yet; this spec is the
> build-out plan, built like identity (FEAT-02) when scoped.

## Current state (to build)

There is no analytics screen and no `lib/features/analytic/` tree yet. The protected shell
hosts dashboard / messages / notifications / settings tabs (FEAT-01); this feature adds the
analytics screen and its `data`/`domain`/`presentation` layers.

## Goal

Give authenticated (admin) users an analytics overview screen: headline stat cards, a time-series
view of events, and a paginated event list — all read through Riverpod notifiers + repositories
against the backend analytic API, rendered with the shared widget kit and theme tokens. The
backend dashboard/report/event routes are `admin`-only, so the tab/route is gated on
`user.role == 'admin'` (mirror the identity admin gate / router guard); the API enforces it.

## Scope

Feature-first layout `lib/features/analytic/{data,domain,presentation}`.

- **Analytics screen** — new `analytics_screen` added to the protected `ShellRoute` as a tab
  (icon + localized label); thin route widget that renders the analytics page composed of the
  cards, chart, and list. Gate the tab/route on `user.role == 'admin'` (mirror the identity admin
  gate / router guard); the API enforces it.
- **Summary stat cards** (`presentation/widgets/stat_cards`) — headline counts from the dashboard
  summary, laid out as `AppCard`s.
- **Events chart** (`presentation/widgets/events_chart`) — time-series visualization using an
  approved Flutter chart package (e.g. `fl_chart`) — a new pub dep requires developer approval
  (note it in QA notes); colors come only from chart/theme tokens (extend the theme via the
  `AppStatusColors`/a chart `ThemeExtension`). Falls back to a summarized list if a chart dep is
  not approved.
- **Event list** (`presentation/widgets/event_list`) — paginated list (`ListView`/`Sliver`) of
  analytics events (`?page&limit`) with infinite scroll (`onEndReached`-style) or an explicit
  pager driven by `PaginationMeta`.
- **Providers** (`presentation/providers/analytics_notifier`) — a dashboard-summary provider and a
  paginated-events notifier (immutable `freezed` state with loading/empty/error/success), reading
  through the repository.
- **Data layer** — freezed DTOs (`DashboardSummary`, `AnalyticsEvent`, `EventListQuery`), a
  retrofit datasource, and a repository impl mapping DTO → domain + exceptions → `Failure`.
- Every panel handles loading (skeleton/spinner) / error (`Failure` + retry) / empty / success.
- **Out of scope:** emitting analytics events (server-side), CSV export, custom date-range pickers.

## Consumed API

Base `/api/v1/analytics`, envelope `{ success, data, meta? }`.

- `GET /analytics/dashboard` → `DashboardSummary` (headline metrics).
- `GET /analytics/events?page&limit` → paginated `AnalyticsEvent[]` with `meta`.

## UI / widgets used

- `features/analytic/presentation/screens/analytics_screen`, `presentation/widgets/{stat_cards,
  events_chart,event_list}`.
- Shared `AppCard`, the async-state scaffold (loading/empty/`Failure`+retry); an approved chart
  dep (pending); the protected `ShellRoute` tab group.

## Acceptance criteria

- [ ] The analytics tab renders behind the auth guard (unauthenticated → `/sign-in`) and is visible
      only when `user.role == 'admin'` (the backend dashboard/report/event routes are `admin`-only).
- [ ] Stat cards render headline metrics from `GET /analytics/dashboard`.
- [ ] The events chart renders a time series (or a summarized list fallback) using only chart/theme
      tokens — no hardcoded colors.
- [ ] The event list paginates via `?page&limit` and reads counts from `meta`
      (`page/limit/total/totalPages`).
- [ ] Loading, error, and empty states render on every panel.
- [ ] Data flows through `analytics_notifier` + the repository + retrofit datasource only — no `dio`
      in widgets; responses typed via the freezed DTOs/entities.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only theming, a11y, dart analyze, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
