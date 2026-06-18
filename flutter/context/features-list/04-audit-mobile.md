# Feature 04 — Audit Log & Alerts (Mobile)

- Feature ID: FEAT-04
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — guarded shell screen)

> Source of truth for the authenticated **audit** screen: a filterable audit-log viewer and a
> security-alerts list with resolve actions. Consumes the backend `audit` domain. TO-BUILD on an
> empty project — `lib/features/audit/` does not exist yet; this spec is the build-out plan, built
> like identity (FEAT-02) when scoped.

## Current state (to build)

There is no audit screen and no `lib/features/audit/` tree yet. This feature adds the audit
screen and its `data`/`domain`/`presentation` layers.

## Goal

Give authenticated (admin) users an audit screen: browse the paginated audit log with filters,
review security alerts by severity/resolution, and resolve open alerts — all through Riverpod
notifiers + repositories against the backend audit API.

## Scope

Feature-first layout `lib/features/audit/{data,domain,presentation}`.

- **Audit screen** — new `audit_screen` added to the protected `ShellRoute` (icon + localized
  label); thin route that composes the log list and the alerts list (e.g. a `SegmentedButton` /
  two sections / `TabBar`).
- **Audit-log list** (`presentation/widgets/audit_log_list`) — paginated list of log entries with
  filters (`userId`, `action`, `entity`); each row shows actor, action, entity, timestamp
  (formatted via `intl`).
- **Alert list** (`presentation/widgets/alert_list`) — paginated security alerts with `severity` +
  `resolved` filters and a per-row Resolve action; severity rendered via semantic theme tokens
  (e.g. `colorScheme.error` for high, `AppStatusColors.warning` for medium).
- **Providers** (`presentation/providers/audit_notifier`) — paginated audit-logs notifier, an
  alerts notifier, and a resolve-alert intent (refreshes/invalidates the alerts list).
- **Data layer** — freezed DTOs (`AuditLog`, `SecurityAlert`, `AuditListQuery`, `AlertListQuery`),
  a retrofit datasource, and a repository impl mapping DTO → domain + exceptions → `Failure`.
- Loading / error (`Failure` + retry) / empty states on both lists; resolve gives `SnackBar`
  feedback.
- Gate the tab/route on `user.role == 'admin'` (mirror the identity admin gate / router guard).
- **Out of scope:** writing audit entries (server-side), CSV export, alert-rule configuration.

## Consumed API

Base `/api/v1/audit`, envelope `{ success, data, meta? }`.

- `GET /audit?page&limit&userId&action&entity` → paginated `AuditLog[]` with `meta`.
- `GET /audit/alerts?page&limit&severity&resolved` → paginated `SecurityAlert[]`.
- `PATCH /audit/alerts/:id/resolve` → updated `SecurityAlert`.

## UI / widgets used

- `features/audit/presentation/screens/audit_screen`, `presentation/widgets/{audit_log_list,
  alert_list}`.
- Shared `AppCard`, a badge/status chip, the async-state scaffold (loading/empty/`Failure`+retry),
  `SnackBar` feedback; the protected `ShellRoute` tab group.

## Acceptance criteria

- [ ] The audit tab renders behind the auth guard (unauthenticated → `/sign-in`) and is visible only
      when `user.role == 'admin'`.
- [ ] The audit-log list paginates and filters by `userId` / `action` / `entity`.
- [ ] The alert list paginates and filters by `severity` / `resolved`.
- [ ] Resolving an alert calls `PATCH /audit/alerts/:id/resolve`, refreshes the alerts provider, and
      surfaces success/error via `SnackBar`.
- [ ] Loading, error, and empty states render on both lists.
- [ ] All HTTP flows through the audit repository + retrofit datasource; responses typed via the
      freezed DTOs/entities; no `dio` in widgets.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only theming, a11y, dart analyze, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
