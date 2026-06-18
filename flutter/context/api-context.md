# API Context — Consumed Backend Surface

Machine-readable contract for the backend this app **consumes**: the envelope, auth
model, shared conventions, and the living registry of endpoints the app talks to.
`context/api-rules.md` says how to wire the client; this file IS the contract they
share. Keep it consistent with `context/architecture.md` and with the backend
template's own `api-context.md` (the server is the source of truth).

## Response envelope (every endpoint, no exceptions)

The backend returns one envelope for every response; the app parses it once in the
network layer (`core/network`) into typed `freezed` results before handing data to
repositories.

```dart
// success — parsed into ApiResponse<T>
// { "success": true, "data": <T>, "meta": { ... }? }

// error — parsed into ApiError, then mapped to a Failure
// { "success": false, "error": { "code": "STRING", "message": "STRING", "fields": { "field": ["msg"] }? } }

@freezed
class ApiResponse<T> with _$ApiResponse<T> {
  const factory ApiResponse({ required T data, PaginationMeta? meta }) = _ApiResponse;
}

@freezed
class ApiError with _$ApiError {
  const factory ApiError({
    required String code,
    required String message,
    Map<String, List<String>>? fields,
  }) = _ApiError;
}

@freezed
class PaginationMeta with _$PaginationMeta {
  const factory PaginationMeta({
    required int page, required int limit, required int total, required int totalPages,
  }) = _PaginationMeta;
}
```

The error is an **object** (`{ code, message, fields? }`), not a bare string. The app
keys behavior off `error.code`; `message` is already localized server-side and may be
shown directly (or remapped to a local string by code).

## Auth model

- Mechanism: **JWT bearer**. Login/refresh return an access token (short-lived,
  ~15m) + refresh token (~7d). The access token is sent as
  `Authorization: Bearer <token>` on every authenticated request.
- Storage: both tokens live in `flutter_secure_storage` (`core/storage`) — never in
  prefs, logs, or source.
- Refresh: a `dio` interceptor catches `401`, performs a **single-flight** refresh via
  the refresh token, retries the original request once, and on refresh failure clears
  tokens and routes to login (`go_router` redirect).
- Default posture: treat endpoints as protected unless documented public (auth/health
  /register). The app attaches the token to all `/api/v1` calls except the explicit
  public ones.
- Authorization: roles/permissions come from the authenticated profile; the app gates
  UI, the server enforces access. A `403` means show "not allowed"; a `404` may be the
  server hiding a resource the caller must not know exists.

## Pagination convention

- Request: `?page=1&limit=20` (`page` ≥ 1 default 1; `limit` 1–100 default 20). List
  calls add their own validated filters/sort params.
- Response: items in `data`; `meta: { page, limit, total, totalPages }`. Repositories
  return a typed page object so the UI can drive infinite scroll / pull-to-refresh.

## Validation / error mapping

- The backend validates input and returns `400` with field errors in `error.fields`.
  The app surfaces these per-field on forms (keyed by field name).
- Status → `Failure` mapping (in `core/error`):

| Status | Failure | UI treatment |
| --- | --- | --- |
| 400 | `ValidationFailure(fields)` | inline field errors |
| 401 | `AuthFailure` | trigger refresh; on failure → login |
| 403 | `ForbiddenFailure` | "not allowed" message |
| 404 | `NotFoundFailure` | empty / not-found state |
| 409 | `ConflictFailure` | conflict message (e.g. duplicate) |
| 5xx / network / timeout | `ServerFailure` / `NetworkFailure` | retry affordance |

## Realtime / background (if used)

- The backend exposes realtime over a JWT-authed socket handshake (same token). If a
  feature needs it, the connection lives in `core/network` and pushes into Riverpod
  state — never directly into widgets. Add the channel here when first consumed.

## Consumed-endpoints registry (living document)

Updated by Builders with every endpoint the app starts calling; checked before adding
a new datasource method (reuse > duplicate); audited by `/review`. One row per
`/api/v1` endpoint the app consumes.

| Method | Path | Auth | Feature | Used by (repository / datasource) |
| --- | --- | --- | --- | --- |
| _(empty — first row added by the first feature that calls the API)_ |

## Invariants

1. No API call outside a repository (via a `retrofit` datasource) — and no consumed
   endpoint missing from this registry.
2. No response handled outside the envelope parser — every payload becomes a typed
   model or a typed `Failure`.
3. Shared API types (envelope, error, pagination) are defined once in the network
   layer and imported — never re-declared per feature.
