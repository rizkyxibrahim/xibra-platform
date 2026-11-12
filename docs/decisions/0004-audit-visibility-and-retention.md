# ADR 0004: Audit Visibility and Retention

## Status

Accepted

## Date

2026-11-12

## Context

The API already writes two security-relevant record sets:

- `audit_logs` records user, service, and system actions. It stores the acting identity and, where applicable, the affected user, plus the action, optional subject, safe JSON properties, IP address, user agent, and timestamps. Invitation and Telegram actions are already recorded.
- `onboarding_attempts` records failed invitation redemption and session validation attempts, including outcome, response code, IP address, and user agent. It never stores the submitted secret, session token, or their hashes.

Neither table has a read endpoint, and retention is undefined. The `telegram.linked` event currently records `actor_type = service` with `user_id = null`, so a member-scoped audit view would not see that service action even though it affects their own account. Audit data contains IP addresses and user agents, which are personal data and must not be broadly readable.

## Decision

### Read Access

| Method | Path | Access | Scope |
|---|---|---|---|
| `GET` | `/api/v1/audit-logs` | `audit.view` permission (Owner only by default) | All audit records |
| `GET` | `/api/v1/onboarding-attempts` | `audit.view` permission (Owner only by default) | All onboarding attempts |
| `GET` | `/api/v1/me/audit-logs` | Any authenticated, active user | Forced to the caller's `user_id` |

Audit records are read-only through the API. There is no create, update, or delete endpoint; writes happen only inside the domain service that performs the audited action, through `AuditLogger`.

Supported filters:

- `audit-logs`: `action`, `actor_type`, `user_id`, `subject_type`, `subject_id`, `from`, `to`, `per_page`.
- `onboarding-attempts`: `outcome`, `invitation_id`, `from`, `to`, `per_page`.
- `me/audit-logs`: `action`, `subject_type`, `from`, `to`, `per_page`; `user_id` is not accepted.

All three endpoints are paginated using the shared pagination policy, sorted newest first, and rate-limited with a standard authenticated read limiter. Unsupported filter or sort fields are rejected with `422`.

### `user_id` Semantics

`audit_logs.user_id` means the user the event belongs to:

- For a user-initiated action, it is the acting user.
- For a service- or system-initiated action that affects an account, it is the affected user; the acting service is represented by `actor_type = service`.

This requires one behavior change in the existing Telegram flow: `telegram.linked` must store the linked account's `user_id` while keeping `actor_type = service`. `telegram.unlinked` already attributes the action to the user. No other table change is required.

Service actions that do not target a specific user keep `user_id = null` and remain visible only to the Owner.

### Response Shape

`AuditLogResource` exposes:

- `id`
- `action`
- `actor_type`
- `user` (minimal reference: `id`, `username`) or `null`
- `subject_type` and `subject_id`, or `null`
- `properties`
- `ip_address`
- `user_agent`
- `created_at`

`OnboardingAttemptResource` exposes:

- `id`
- `invitation_id`
- `outcome`
- `response_code`
- `ip_address`
- `user_agent`
- `created_at`

Neither resource may expose passwords, access tokens, session tokens, invitation codes or link tokens, Telegram link codes, or any hash of those values. The existing `AuditLogger` contract already forbids such values; the resource layer adds a whitelist rather than returning the raw row.

Telegram usernames are not written to audit records. Numeric Telegram IDs may be included for traceability.

### Privacy

- IP addresses and user agents are personal data. The Owner sees all records; a member sees only records attributed to their own `user_id`.
- Onboarding attempts are Owner-only because they can contain request metadata from unknown or unauthenticated clients.
- Audit and attempt reads are not themselves audited in v1. Access is permission-gated and covered by request logs. This may be revisited if compliance requires an access trail.

### Retention

| Record set | Retention | Rationale |
|---|---|---|
| `onboarding_attempts` | 90 days | Abuse investigation window; no account may exist yet |
| `audit_logs` | 365 days | Security and lifecycle traceability for a full year |

Retention is configured in `config/xibra.php`:

- `xibra.audit.retention_days` (default `365`)
- `xibra.onboarding.attempt_retention_days` (default `90`)

A daily scheduled command, `xibra:prune-audit`, deletes records older than the configured cutoffs in bounded chunks and logs the deleted counts. There is no API deletion path.

Pruning must be paused during an active security or legal hold. The operational procedure is to disable the scheduled command and record the hold; retention must not silently destroy evidence under review.

The same 365-day rule covers account deactivation, reactivation, password reset, and invitation audit events.

## Consequences

### Positive

- The Owner can investigate onboarding abuse and service activity without direct database access.
- Members can review security events that affect their own account.
- Retention is explicit and enforced automatically instead of growing without bound.
- Audit responses have a stable, secret-free contract.
- Service actions are correctly attributed to the affected account.

### Negative

- Three new read endpoints and a scheduled command add surface area.
- The Telegram audit records need a small behavior change and corresponding tests.
- Member self-audit cannot show events that were never attributed to a user.
- Retention deletes security history after one year and must be paused for investigations.

## Security Implications

- Audit reads require authentication and, for global views, the `audit.view` permission.
- Member reads are forcibly scoped to the authenticated `user_id`; a client-supplied user filter is ignored or rejected.
- Audit and attempt resources never return secrets, tokens, codes, or hashes.
- IP addresses and user agents are visible only to the Owner or to the user they belong to.
- Audit rows remain append-only through the application. Database-level immutability is out of scope for v1; database access stays restricted to `xibra-api` per ADR 0001.
- Pruning is a deliberate, logged operation and has a documented pause procedure for holds.

## Alternatives Considered

### Owner-Only Audit Access

Expose only the two Owner endpoints.

Rejected because members should be able to review their own security history, and the dashboard account settings module will need it.

### No Read Endpoints

Keep audit data accessible only through direct database access.

Rejected because it makes routine investigation depend on production database access and provides no auditable application path.

### Indefinite Retention

Keep all audit records forever.

Rejected because it conflicts with data minimization and grows storage without a stated requirement.

### Separate Audit Database

Write audit records to a dedicated database or external service.

Rejected for v1 as unnecessary operational complexity. The current transaction-scoped audit writes are sufficient and keep the audit record atomic with the change it describes.

### Audit the Audit Reads

Record an `audit.viewed` event for every audit read.

Deferred because it adds noise and recursive growth. It can be added if a compliance requirement appears.

## Review Conditions

This decision may be reconsidered if:

- A compliance or legal requirement changes retention or access.
- Audit volume makes the current table or pruning strategy impractical.
- The dashboard requires additional member-facing security history.
- A security incident shows that audit reads need their own access trail.

Any change must be recorded through a new or superseding ADR.

## References

- `docs/decisions/0001-central-api-and-database.md`
- `docs/decisions/0002-account-lifecycle-and-password-recovery.md`
- `docs/database/README.md` section 12
- `docs/api/conventions.md` section 10
- `xibra-api/app/Models/AuditLog.php`
- `xibra-api/app/Models/OnboardingAttempt.php`
- `xibra-api/app/Services/Audit/AuditLogger.php`
- `xibra-api/app/Services/Telegram/TelegramLinkService.php`
