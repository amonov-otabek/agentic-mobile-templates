# API Context — Consumed Backend Surface

Machine-readable contract for the **backend API this app consumes**. `context/api-rules.md`
says how to wire feature data layers; this file IS the contract they share. The app does
not own this API — it mirrors the backend monolith's envelope, auth model, and
conventions through the typed client in `shared/libs/api.ts`.

## Base URL

- Configured via `EXPO_PUBLIC_API_URL` (inlined at build time), default
  `http://localhost:3000`. A physical device needs a LAN IP; Android emulator uses
  `http://10.0.2.2:3000`. See `.env.example`.
- All product endpoints are prefixed `/api/v1`.

## Response envelope (every endpoint, no exceptions)

Defined in `shared/libs/api.ts`; the client unwraps `data` on success and throws
`ApiError` on failure.

```ts
// success
interface ApiSuccess<T> {
  success: true;
  data: T;
  meta?: ApiMeta;                 // present on paginated lists
}

// error
interface ApiFailure {
  success: false;
  error: { code: string; message: string; fields?: Record<string, string[]> };
}

interface ApiMeta { total: number; page: number; limit: number }
```

The error is an **object** (`{ code, message, fields? }`), not a bare string. A failed
or non-2xx response is thrown as `ApiError` (`status`, `code`, `message`, `fields`);
`204 No Content` resolves to `data: undefined`. Assert on `error.code`, not message text
(messages are i18n-localized by the backend).

## Auth model

- Mechanism: **JWT** — access token (short-lived) + refresh token. Sign-in / sign-up
  return `{ user, tokens?, pending_token?, requires_two_fa? }`; a 2FA challenge omits
  `tokens` and returns a short-lived `pending_token` to submit to `/2fa/verify`.
- **Bearer**: the access token is held in memory (`setAccessToken`) and attached as
  `Authorization: Bearer <token>` by the client on every request.
- **Secure persistence**: access + refresh tokens are stored in `expo-secure-store`
  (Keychain / Keystore) via `shared/libs/storage.ts`; `localStorage` on web.
  `AuthProvider` calls `loadSession()` on startup to rehydrate before first paint.
- **Route guards** are client-side: the auth store status drives `(protected)` /
  `(auth)` redirects. The server remains the source of truth (401 → treat as
  unauthenticated and clear tokens).

## Pagination convention

- Query params: `?page=1&limit=20` (page ≥ 1 default 1; limit 1–100 default 20),
  mirrored by `paginationQuerySchema` in `shared/libs/validators.ts`.
- The client exposes `api.getPaginated<T>()` → `{ data: T[], meta: { total, page, limit } }`.

## Validation convention

- **Zod** on the client mirrors the backend schemas (one schema per input surface in
  `validators/*`). Client validation is first-line UX; the server still validates and
  may return `error.fields`, which the form merges into its field errors.

## Consumed-endpoints registry (living document)

Updated by Builders with every backend endpoint the app calls; checked before adding a
new call (reuse the existing `api/*` wrapper > duplicate). One row per endpoint the app
consumes. The identity feature already wires the auth/profile/2fa surface — list those
rows here as they are confirmed against the backend.

| Method | Path | Auth | Feature | Used by (api fn) |
| --- | --- | --- | --- | --- |
| _(empty — first row added by the first wired endpoint)_ |

## Invariants

1. No HTTP call outside `shared/libs/api.ts` (wrapped per resource in `features/*/api/*`).
2. No assumption that a response is anything but the envelope — success unwraps `data`,
   failure is an `ApiError`.
3. Backend contract types live once in `features/<name>/types/*` and are imported —
   never re-declared inline.
4. Tokens are read/written only through `shared/libs/storage.ts` + the client's
   `setAccessToken`; never persisted elsewhere.
