# ADR 0005: Email Verification

## Status

Accepted

## Date

2026-11-13

## Context

Email addresses are optional in Xibra Platform. ADR 0002 enables self-service password recovery only for verified email addresses, but there is no way for an account to become verified. Until it exists, the operator CLI is the only recovery channel.

Verification must work for single-page clients, must not depend on a session, and must follow the same hardening rules as password reset tokens.

## Decision

### Verification Tokens

| Property | Rule |
|---|---|
| Storage | `email_verification_tokens`, keyed by `user_id`; only a keyed hash of the token is stored |
| Token | 64 random characters from a CSPRNG, hashed with HMAC-SHA-256 under a domain-separated prefix |
| Lifetime | 24 hours |
| Active tokens | One per user; issuing a new token replaces the previous one atomically |
| Use | Single use; `used_at` is set when the token is consumed |
| Address binding | The verified address is stored with the token; a token is invalid if `users.email` changed after issuance |
| Link | Built from `XIBRA_EMAIL_VERIFICATION_URL`; issuance fails with a clear error when it is not configured |

### Endpoints

| Method | Path | Access | Purpose |
|---|---|---|---|
| `POST` | `/api/v1/auth/email/verification` | Authenticated, active, throttled (3 per hour per user) | Send a verification link to the current email |
| `POST` | `/api/v1/auth/email/verify` | Public, throttled (10 per minute per client) | Consume a token and mark the email verified |

The send endpoint returns `409 EMAIL_ALREADY_VERIFIED`, `422 EMAIL_NOT_SET`, `503 EMAIL_VERIFICATION_UNAVAILABLE`, or `503 EMAIL_DELIVERY_FAILED` where relevant.

The verify endpoint returns one `422 EMAIL_VERIFICATION_TOKEN_INVALID` response for unknown, expired, used, address-mismatched, and inactive-account cases.

### Onboarding

When an email address is provided during onboarding, a verification link is sent after the account is created. Delivery is best-effort: a failure is logged without secrets and never affects account creation.

### Account Lifecycle

- Deactivation ends any active verification token in the same transaction as the status change (extends the effects listed in ADR 0002).
- Reactivation does not restore tokens.
- Any future operation that changes `users.email` must clear `email_verified_at` and end active verification tokens.

### Audit

- `user.email_verification_sent` (channel `email`)
- `user.email_verified` (only when the state actually changes)

No token, hash, or email address is written to audit records or logs.

## Consequences

### Positive

- Self-service password recovery can be enabled for verified addresses.
- The mechanism reuses the proven reset-token design.
- Changing an address cannot leave a stale verification behind.

### Negative

- Verification depends on mail infrastructure that is not production-ready.
- A new table and two endpoints must be maintained.
- Accounts without an email still depend on the CLI recovery channel.

## Security Implications

- Tokens are bearer secrets: hashed at rest, single use, short-lived, throttled, and never logged.
- Responses for the verify endpoint do not reveal whether a token or account exists.
- The link base URL is deployment configuration; tokens are never embedded in API responses.
- Placing the token in the query string exposes it to proxy logs and Referer headers; moving it to a URL fragment requires a follow-up decision and a single change in link construction.

## Alternatives Considered

### Laravel Signed URLs

Stateless signed links with the default `MustVerifyEmail` flow.

Rejected because the Dashboard is a single-page client that would have to forward signature parameters, and the flow gives no server-side invalidation when the address changes.

### Verification Code Entered Manually

A short numeric code typed into the Dashboard.

Rejected because short codes need stricter attempt limits and add a second secret format without a clear benefit over a single-use link.

## Review Conditions

Reconsider if mail delivery moves to a queue, a profile endpoint can change the email address, or the token placement in the link changes.

## References

- `docs/decisions/0002-account-lifecycle-and-password-recovery.md`
- `xibra-api/app/Services/Auth/EmailVerificationService.php`
- `xibra-api/app/Services/Auth/PasswordResetService.php`