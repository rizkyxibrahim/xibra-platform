# Xibra Platform — API Conventions

## 1. Purpose

This document defines the API standards for all applications communicating with `xibra-api`.

All API endpoints must follow consistent naming, versioning, authentication, authorization, ownership, response, validation, and error-handling conventions.

The API is the authoritative interface for platform business logic and data access.

These conventions apply to the portfolio, dashboard, Telegram bot, Hermes agent, and future integrations.

## 2. API Versioning

All application endpoints must use a versioned prefix:

    /api/v1

Examples:

    GET /api/v1/health
    GET /api/v1/public/profile
    GET /api/v1/auth/me
    GET /api/v1/dashboard
    GET /api/v1/invitations

Breaking API changes must be introduced through a new API version.

Non-breaking changes should preserve compatibility with existing clients.

## 3. Endpoint Organization

Endpoints are organized by access requirements and functional responsibility.

### 3.1 Public

Endpoints that do not require user authentication.

Examples:

    GET /api/v1/public/profile
    GET /api/v1/public/projects
    GET /api/v1/public/blog

Public endpoints may only return explicitly approved public information.

They must not expose private user data, unpublished content, internal records, or sensitive information.

### 3.2 Authentication

Endpoints responsible for account authentication and session management.

Examples:

    POST /api/v1/auth/login
    POST /api/v1/auth/logout
    GET /api/v1/auth/me

Supported login identifiers:

- Username.
- Email address.
- Phone number.

Username is required. Email and phone number are optional account identifiers.

The API must resolve the authenticated identity centrally.

Public self-registration must not be implemented.

### 3.3 User

Endpoints for authenticated users to manage their own resources.

Examples:

    GET /api/v1/me/profile
    PUT /api/v1/me/profile
    GET /api/v1/me/projects
    GET /api/v1/me/integrations

The API must derive the current user from the authenticated request context.

Clients must not supply arbitrary user identifiers to access another user's resources.

### 3.4 Dashboard and Administration

Endpoints for authorized platform and module operations.

Examples:

    GET /api/v1/dashboard
    GET /api/v1/admin/users
    GET /api/v1/admin/invitations
    POST /api/v1/admin/invitations

Administrative endpoints require authentication and the appropriate permissions.

The `admin` route prefix does not replace backend authorization checks.

Only the Owner may perform Owner-restricted invitation management operations.

### 3.5 Service Integrations

Endpoints or operations intended for authenticated application services.

Examples may include:

    POST /api/v1/bot/telegram/link
    POST /api/v1/integrations/assistant/actions

Service endpoints must validate the calling service identity.

Service authentication must not automatically grant access to private user resources.

The implemented bot service mechanism (HMAC-signed requests) is documented in section 12. Service credentials must never authenticate user endpoints, and user tokens must never authenticate service endpoints.

## 4. HTTP Methods

| Method | Purpose |
|---|---|
| GET | Retrieve resources |
| POST | Create resources or execute actions |
| PUT | Replace or fully update a resource |
| PATCH | Partially update a resource |
| DELETE | Delete a resource |

Use resource-oriented and predictable endpoint names.

Use plural resource names for collections.

Use explicit action endpoints when an operation represents a business action rather than a standard resource update.

Examples:

    POST /api/v1/invitations/{id}/revoke
    POST /api/v1/invitations/{id}/redeem
    POST /api/v1/auth/logout

Do not expose database implementation details through endpoint naming.

## 5. Standard JSON Response

All API responses containing a response body must follow the shared JSON contract.

### 5.1 Successful Response

Example:

    {
        "success": true,
        "message": "Projects retrieved successfully.",
        "code": "PROJECTS_RETRIEVED",
        "data": [],
        "meta": null
    }

### 5.2 Error Response

Example:

    {
        "success": false,
        "message": "The request is invalid.",
        "code": "VALIDATION_ERROR",
        "data": null,
        "meta": {
            "errors": {}
        }
    }

### 5.3 Response Fields

| Field | Purpose |
|---|---|
| success | Indicates whether the request succeeded |
| message | Human-readable result |
| code | Stable application-level response code |
| data | Response payload |
| meta | Optional metadata |

The actual HTTP status code must accurately represent the result.

The response code must remain stable enough for clients to handle application outcomes programmatically.

The `message` field is intended for display and must not be used as the primary condition for client-side business logic.

### 5.4 No-Content Responses

HTTP 204 may be used only when an endpoint explicitly defines a response without a body.

Otherwise, use the standard JSON response contract, including for successful update and delete operations.

## 6. HTTP Status Codes

| Status | Usage |
|---|---|
| 200 | Successful request |
| 201 | Resource created |
| 202 | Request accepted for asynchronous processing |
| 204 | Successful request without response content |
| 400 | Invalid request |
| 401 | Authentication required or invalid |
| 403 | Authenticated request lacks required permission |
| 404 | Resource not found or intentionally concealed |
| 409 | Resource conflict or state conflict |
| 422 | Validation failure |
| 429 | Too many requests |
| 500 | Unexpected server error |
| 503 | Service temporarily unavailable |

Use status codes consistently across endpoint groups.

A resource that belongs to another user may return 404 when resource concealment is part of the security policy.

## 7. Authentication

Protected user endpoints use Laravel Sanctum authentication.

The API must:

- Validate authentication credentials and tokens.
- Resolve the authenticated user from trusted authentication context.
- Enforce account status.
- Support logout and token revocation.
- Protect sensitive operations.
- Avoid exposing authentication secrets in responses.
- Apply rate limits to authentication endpoints.
- Never trust client-provided user identifiers as proof of identity.

Login identifiers must be normalized consistently before lookup.

Email addresses and usernames must follow the platform's normalization rules.

Phone numbers must use a consistent international format.

Service-to-service authentication is separate from ordinary user authentication and must be explicitly defined. The bot service contract is defined in section 12.

## 8. Authorization and Data Ownership

Authentication does not automatically grant access to every resource.

Every protected operation must enforce the appropriate authorization requirements.

The API must:

- Check the authenticated user's permissions.
- Validate resource ownership.
- Apply policies to nested and related resources.
- Prevent unauthorized ownership reassignment.
- Prevent access to another user's private records.
- Validate explicit sharing authorization when sharing is supported.

Role permissions and resource ownership are separate authorization requirements.

A user with a valid role must not automatically gain access to another user's private resources.

The platform Owner must not automatically receive unrestricted access to all private user data.

All authorization checks must occur on the backend.

Frontend route guards and hidden UI elements are not security boundaries.

The concrete ownership and isolation rules are recorded in ADR 0003. In particular:

- Private records are resolved through the authenticated user's relationship, never from a client-supplied ownership identifier.
- A request for another user's private record returns `404`, not `403`, so record existence does not leak.
- The Owner role does not bypass private-resource ownership. Owner administration uses separate permission-gated endpoints.

## 9. Validation

- Validate all incoming data.
- Reject invalid data with HTTP 422.
- Return field-level validation errors in `meta.errors`.
- Never trust client-side validation as a security boundary.
- Apply authorization before performing protected operations.
- Validate related resource ownership before creating or updating relationships.
- Normalize identifiers consistently.
- Reject unexpected or unauthorized ownership fields.

Validation must protect both data integrity and security boundaries.

## 10. Pagination

Collection endpoints should use consistent pagination.

Example request:

    GET /api/v1/me/projects?page=1&per_page=15

Example metadata:

    {
        "current_page": 1,
        "per_page": 15,
        "total": 45,
        "last_page": 3
    }

Pagination metadata belongs inside the `meta` field.

The API should enforce a maximum allowed page size.

Collection responses must not expose records outside the authenticated user's authorized scope.

## 11. Invitation and Onboarding

Invitation operations are managed centrally by `xibra-api`.

Only the platform Owner may create, view, and revoke invitations.

Dashboard and Telegram Bot must use the same invitation system and API rules.

Invitation endpoints must:

- Validate invitation status.
- Enforce expiration.
- Enforce one-time redemption.
- Prevent concurrent redemption.
- Prevent reuse of redeemed or revoked invitations.
- Validate the assigned role.
- Use atomic database transactions for redemption and account creation.
- Avoid exposing stored invitation hashes or internal secrets.

Approved invitation defaults:

| Property | Rule |
|---|---|
| Default expiration | 24 hours |
| Minimum expiration | 1 hour |
| Maximum expiration | 72 hours |
| Redemption | One-time |
| Default role | `member` |

Supported invitation states:

- `pending`
- `redeemed`
- `expired`
- `revoked`

The implemented onboarding flow uses two public endpoints:

- `POST /api/v1/invitations/redeem` accepts `{ "secret": "<code or link token>" }` and opens a 15-minute, single-use onboarding session. Re-redeeming replaces the previous session.
- `POST /api/v1/onboarding/complete` accepts the session token plus `name`, `username`, optional `email` and `phone`, `password`, and `device_name`; it creates the active account, assigns the invitation's role, and returns a Sanctum token.

Invitation validation failures must return safe error messages without exposing sensitive invitation data.

Passwords must never be collected through Telegram.

Account password setup must occur through a secure HTTPS onboarding flow.

Password recovery is defined in ADR 0002: self-service email reset for verified addresses, plus a single-use operator-issued reset link for accounts without a usable email. Passwords are never set, shared, or communicated by the Owner.

## 12. Telegram Identity Linking

Telegram identity is linked to an existing Xibra account; it never creates or authenticates an account on its own.

### 12.1 Bot Service Authentication

The Telegram bot authenticates as a trusted service, separately from user authentication, using HMAC-signed requests.

Every bot request must send:

| Header | Value |
|---|---|
| `X-Xibra-Key-Id` | Configured service key identifier |
| `X-Xibra-Timestamp` | Unix time in seconds |
| `X-Xibra-Nonce` | 16-64 URL-safe characters (`A-Z`, `a-z`, `0-9`, `_`, `-`) |
| `X-Xibra-Signature` | Lowercase hex HMAC-SHA256 of the canonical string |

The canonical string is the following values joined with `\n`:

```
METHOD
REQUEST_URI (path plus query string)
TIMESTAMP
NONCE
SHA256(raw request body) as lowercase hex
```

The API rejects a request when:

- The key identifier does not match.
- The timestamp is more than 300 seconds away from server time.
- The nonce was already used within its 600-second retention window.
- The signature does not match.
- Either credential is not configured (the service fails closed; the secret must be at least 32 characters).

All failures return `401` with code `SERVICE_UNAUTHENTICATED` and an identical message, so the response never reveals which check failed. A nonce is consumed only after a valid signature, and the bot rate limit runs before signature verification so unsigned traffic is rejected cheaply.

The mechanism is configured through `XIBRA_BOT_KEY_ID` and `XIBRA_BOT_SECRET`; the shared secret must be generated with a CSPRNG, stored outside source control, and rotated by changing both values together. `XIBRA_BOT_RATE_LIMIT_PER_MINUTE` controls the per-address bot limit (default 120).

Bot service endpoints are versioned like every other endpoint and are rate-limited. They do not accept Sanctum tokens, and user tokens cannot call them.

### 12.2 Account Linking

Linking requires proof of both sides:

1. An authenticated, active user requests a one-time link code with `POST /api/v1/telegram/link-codes`.
2. The user sends the code to the Telegram bot.
3. The bot submits the code and the numeric Telegram user ID from the verified Telegram update to `POST /api/v1/bot/telegram/link` using a signed service request.

Link codes:

| Property | Rule |
|---|---|
| Format | 10 characters from `ABCDEFGHJKLMNPQRSTUVWXYZ23456789`, displayed as `ABCDE-FGHJK` |
| Matching | Case-insensitive; dashes, spaces, and other punctuation are ignored |
| Lifetime | 10 minutes |
| Storage | HMAC-SHA256 hash only; the plain code is returned once |
| Active codes | One per user; issuing a new code expires the previous one |
| Issuance limit | 5 requests per minute per user |

Only the numeric Telegram user ID is authoritative. Telegram usernames are display metadata, are optional, are normalized without `@`, and must match `[A-Za-z0-9_]{5,32}`; they are never treated as proof of identity.

A Telegram identity belongs to exactly one user and a user has at most one Telegram identity. Both rules are enforced by database unique constraints and translated to `409` responses. A deactivated account cannot issue codes, and a code owned by a deactivated account is reported exactly like an unknown code.

Unlinking removes the database link and records an audit event; a later link starts a new identity record.

### 12.3 Endpoints

| Method | Path | Authentication | Purpose |
|---|---|---|---|
| `POST` | `/api/v1/telegram/link-codes` | User token | Issue a one-time link code |
| `GET` | `/api/v1/telegram` | User token | Read the caller's own link status |
| `DELETE` | `/api/v1/telegram` | User token | Remove the caller's own link |
| `POST` | `/api/v1/bot/telegram/link` | Bot service signature | Link the code's owner to a Telegram user ID |

User endpoints only ever read or modify the authenticated caller's identity; no identifier is accepted from the request body.

### 12.4 Error Codes

| Code | Status | Meaning |
|---|---|---|
| `SERVICE_UNAUTHENTICATED` | 401 | Bot signature, key, timestamp, nonce, or configuration invalid |
| `TELEGRAM_LINK_CODE_INVALID` | 404 | Unknown code, or code owned by a deactivated account |
| `TELEGRAM_LINK_CODE_EXPIRED` | 409 | Code expired or was replaced by a newer code |
| `TELEGRAM_LINK_CODE_CONSUMED` | 409 | Code already used |
| `TELEGRAM_ALREADY_LINKED` | 409 | The account already has a Telegram identity |
| `TELEGRAM_ACCOUNT_IN_USE` | 409 | The Telegram identity is linked to another account |
| `TELEGRAM_NOT_LINKED` | 404 | Unlink requested without an existing link |
| `TELEGRAM_LINK_THROTTLED` | 429 | Too many failed code attempts for that Telegram account |

## 13. Sensitive Operations and Approval

Sensitive operations must follow explicit approval requirements.

Examples include:

- Financial transactions.
- Destructive operations.
- External account changes.
- Privileged administrative actions.
- Other operations designated as approval-required.

An AI agent or application service must not bypass approval requirements.

The API must validate the approval state before executing the operation.

Approval must be associated with the correct authenticated user and intended operation.

Account lifecycle changes, password reset issuance, and audit access are privileged operations. Their authorization and retention rules are recorded in ADR 0002 and ADR 0004.

## 14. Pagination, Filtering, and Sorting

Collection endpoints should use predictable query parameters.

Recommended conventions:

    ?page=1
    &per_page=15
    &sort_by=created_at
    &sort_direction=desc
    &search=keyword

The API must:

- Validate supported filter fields.
- Validate sorting fields.
- Enforce pagination limits.
- Apply authorization before returning filtered results.
- Prevent arbitrary database column selection through query parameters.

Each resource may define additional filters when required.

## 15. Error Handling

- Use stable response codes.
- Return safe and useful error messages.
- Do not expose stack traces in production.
- Log internal exceptions securely.
- Never include passwords, tokens, or secrets in logs.
- Keep error responses consistent across endpoint groups.
- Do not expose internal database errors to clients.
- Distinguish authentication, authorization, validation, and business-rule failures.

Unexpected exceptions must be handled centrally.

## 16. API Documentation and Testing

Every implemented endpoint should have:

- A clear purpose.
- HTTP method and path.
- Authentication requirements.
- Required permissions.
- Ownership requirements.
- Request validation rules.
- Success response example.
- Error response examples.
- Automated tests for expected behavior.

Tests for private endpoints must include:

- Unauthenticated requests.
- Authorized requests.
- Unauthorized roles or permissions.
- Access attempts to another user's resources.
- Invalid input.
- Relevant state conflicts.

Invitation and onboarding tests must cover expiration, revocation, repeated redemption, and concurrent redemption behavior where applicable.

Telegram linking tests must cover signed and unsigned service requests, replay, code expiration and reuse, duplicate identities, and cross-user isolation.

## 17. Compatibility

All API clients must follow the documented contract.

Changes to response structures, authentication, authorization, ownership, or endpoint behavior must be reviewed for their impact on:

- xibra-portfolio
- xibra-dashboard
- xibra-bot
- xibra-core

Breaking changes must be introduced through a new API version.

Client applications must not independently reproduce authoritative backend business rules.

## 18. Implementation Status

This document defines the intended API contract.

It does not imply that all endpoints, authentication mechanisms, authorization policies, or conventions have already been implemented.

Implementation progress must be tracked in the root `TODO.md` and verified against actual application code, database migrations, and automated tests.