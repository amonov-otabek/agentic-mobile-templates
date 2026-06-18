# Feature 04 — Audit Log & Alerts (Mobile)

- Feature ID: FEAT-04
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — guarded tab screen)

> Source of truth for the authenticated **audit** screen: a filterable audit-log viewer and
> a security-alerts list with resolve actions. Consumes the backend `audit` domain.
> **Scaffold today** — `features/audit/*` holds only `.gitkeep` placeholders; build it like
> identity when scoped.

## Current state (scaffold)

`features/audit/{api,components,pages,queries,types}/` contain `.gitkeep` only. There is no
audit tab screen yet. This spec is the build-out plan.

## Goal

Give authenticated (admin) users an audit screen: browse the paginated audit log with
filters, review security alerts by severity/resolution, and resolve open alerts — all through
TanStack Query against the backend audit API.

## Scope

- **Audit screen** — new `(protected)/audit.tsx` route file + `Tabs.Screen` entry (icon
  `Icons.audit`), thin: renders `features/audit/pages/audit-page`, which composes the log
  list and the alerts list (e.g. a segmented control / two sections).
- **Audit-log list** (`components/audit-log-list`) — paginated `FlatList` of log entries with
  filters (`userId`, `action`, `entity`); each row shows actor, action, entity, timestamp.
- **Alert list** (`components/alert-list`) — paginated security alerts with `severity` +
  `resolved` filters and a per-row Resolve action (severity rendered via semantic tokens —
  e.g. `text-destructive` for high).
- **Queries** (`queries/audit.queries`) — `useAuditLogs`, `useAlerts`, `useResolveAlert`
  (mutation invalidates the alerts list); typed via `types/audit.types` (`AuditLog`,
  `SecurityAlert`, `AuditListQuery`, `AlertListQuery`).
- **API** (`api/audit.api`) — thin wrappers over `shared/libs/api`.
- Loading / error / empty states on both lists; resolve gives toast feedback.
- Gate the tab/screen on `role === 'admin'` (mirror the identity admin gate).
- Out of scope: writing audit entries (server-side), CSV export, alert-rule configuration.

## Consumed API

Base `/api/v1/audit`, envelope `{ success, data, meta? }`.

- `GET /audit?page&limit&userId&action&entity` → paginated `AuditLog[]` with `meta`.
- `GET /audit/alerts?page&limit&severity&resolved` → paginated `SecurityAlert[]`.
- `PATCH /audit/alerts/:id/resolve` → updated `SecurityAlert`.

## UI / components used

- `features/audit/components/{audit-log-list,alert-list}`,
  `features/audit/pages/audit-page`.
- Shared `Card`, `Text`, `Button`, a badge/status element, an empty/loading state;
  `shared/hooks/use-toast`; `Icons`; `(protected)` tab group.

## Acceptance criteria

- [ ] The audit tab renders behind the auth guard (unauthenticated → `/sign-in`) and is
      visible only to `role === 'admin'`.
- [ ] The audit-log list paginates and filters by `userId` / `action` / `entity`.
- [ ] The alert list paginates and filters by `severity` / `resolved`.
- [ ] Resolving an alert calls `PATCH /audit/alerts/:id/resolve`, invalidates the alerts
      query, and surfaces success/error via toast.
- [ ] Loading, error, and empty states render on both lists.
- [ ] All HTTP flows through `api/audit.api` + `queries/audit.queries`; responses typed via
      `types/audit.types`; no `fetch` in components.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
