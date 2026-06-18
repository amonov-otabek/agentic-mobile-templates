# Feature 03 — Analytics Dashboard (Mobile)

- Feature ID: FEAT-03
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — this is a guarded tab screen)

> Source of truth for the authenticated **analytics** screen: headline stat cards plus a
> paginated event list (with an optional chart). Consumes the backend `analytic` domain.
> **Scaffold today** — `features/analytic/*` holds only `.gitkeep` placeholders; build it
> like identity when scoped.

## Current state (scaffold)

`features/analytic/{api,components,pages,queries,types}/` contain `.gitkeep` only. There is
no analytics tab screen yet (the tab set is dashboard/messages/users/notifications/settings).
This spec is the build-out plan.

## Goal

Give authenticated users an analytics overview screen: headline stat cards, a time-series
view of events, and a paginated event list — all read through TanStack Query against the
backend analytic API, rendered with the shared component kit and theme tokens.

## Scope

- **Analytics screen** — new `(protected)/analytics.tsx` route file + `Tabs.Screen` entry
  (icon `Icons.analytics`), thin: renders `features/analytic/pages/analytics-page`.
- **Summary stat cards** (`components/stat-cards`) — headline counts from the dashboard
  summary, laid out as `Card`s.
- **Events view** (`components/events-chart`) — time-series visualization using a
  RN-compatible chart lib (e.g. `react-native-gifted-charts` / `victory-native` — confirm
  Expo SDK 56 compatibility and get developer approval before adding a native dep), styled
  from chart tokens (`--chart-1…5`). Falls back to a simple summarized list if a chart dep
  is not approved.
- **Event list** (`components/event-list`) — paginated `FlatList` of analytics events
  (`?page&limit`) with `onEndReached` paging or an explicit pager.
- **Queries** (`queries/analytics.queries`) — `useDashboard` (summary) and `useEvents`
  (paginated, via `api.getPaginated`); typed via `types/analytics.types`
  (`DashboardSummary`, `AnalyticsEvent`, `EventListQuery`).
- **API** (`api/analytics.api`) — thin wrappers over `shared/libs/api`.
- Every panel handles loading (skeleton/spinner) / error / empty / success states.
- Out of scope: emitting analytics events (server-side), CSV export, custom date-range pickers.

## Consumed API

Base `/api/v1/analytics`, envelope `{ success, data, meta? }`.

- `GET /analytics/dashboard` → `DashboardSummary` (headline metrics).
- `GET /analytics/events?page&limit` → paginated `AnalyticsEvent[]` with `meta`.

## UI / components used

- `features/analytic/components/{stat-cards,events-chart,event-list}`,
  `features/analytic/pages/analytics-page`.
- Shared `Card`, `Text`, `Button`; `Icons`; an empty/loading state; a chart dep (pending
  approval); `(protected)` tab group.

## Acceptance criteria

- [ ] The analytics tab renders behind the auth guard (unauthenticated → `/sign-in`).
- [ ] Stat cards render headline metrics from `GET /analytics/dashboard`.
- [ ] The events view renders a time series (or a summarized list fallback) using only chart
      tokens (`--chart-1…5`) — no hardcoded colors.
- [ ] The event list paginates via `?page&limit` and reads counts from `meta`
      (`total/page/limit`).
- [ ] Loading, error, and empty states render on every panel.
- [ ] Data flows through `queries/analytics.queries` + `api/analytics.api` only — no `fetch`
      in components; responses typed via `types/analytics.types`.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
