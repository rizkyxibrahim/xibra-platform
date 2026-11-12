# ADR 0002: Account Lifecycle and Password Recovery

## Status

Accepted

## Date

2026-11-12

## Context

Xibra Platform is invitation-only and has no public registration. The initial Owner is provisioned through a controlled seeder, and every other account is created from an Owner-issued invitation.

The `users` table already has an `is_active` flag. It is enforced when an account logs in (`ACCOUNT_INACTIVE`, HTTP 403, after the password check so the endpoint cannot enumerate accounts), by the `EnsureUserIsActive` middleware for authenticated requests, and by Telegram link-code issuance. There is no API to change the flag, no reactivation path, no account deletion policy, and no password recovery flow.

Email and phone numbers are optional. Telegram never collects or transports passwords, and mail infrastructure is not configured yet. Access must be revocable without destroying the audit, onboarding, or future financial history that references the account.

## Decision

### Account States

An account has exactly two lifecycle states:

| State | Storage | Meaning |
|---|---|---|
| Active | `users.is_active = true` | The account may authenticate and use authorized features. |
| Deactivated | `users.is_active = false` | The account is retained but cannot authenticate or act. |

Deactivation is reversible and never deletes rows. There is no soft-delete column in v1. Hard deletion and anonymization require a separate decision because each future data domain has different retention obligations.

### Authorization

Only the platform Owner may manage the lifecycle of another account, through a new `users.manage` permission that is assigned to the Owner role only.

| Method | Path | Access | Purpose |
|---|---|---|---|
| `GET` | `/api/v1/users` | `users.manage` | Paginated account listing for administration |
| `POST` | `/api/v1/users/{user}/deactivate` | `users.manage` | Deactivate an account |
| `POST` | `/api/v1/users/{user}/reactivate` | `users.manage` | Reactivate an account |

Members cannot deactivate their own account in v1. Self-service deactivation is deferred because it introduces lockout and support cases without adding a security capability: a member can already log out and unlink Telegram.

The following guards apply:

- An Owner cannot deactivate their own account (`409`, `CANNOT_DEACTIVATE_SELF`).
- The last active Owner cannot be deactivated (`409`, `LAST_ACTIVE_OWNER`).
- Repeating a deactivation or reactivation is idempotent: it returns `200` and records an audit event only when the state actually changes.
- An unknown target returns `404` through normal route binding.

### Deactivation Effects

Deactivation is one database transaction that:

1. Sets `is_active` to `false`.
2. Deletes every Sanctum token belonging to the account.
3. Expires every active Telegram link code belonging to the account.
4. Expires any active password reset token belonging to the account.
5. Records a `user.deactivated` audit event with the acting Owner, the target account, and the request context.

After deactivation:

- Login returns `403` with `ACCOUNT_INACTIVE` only after a correct password is supplied, preserving the existing anti-enumeration behavior.
- Any token that was not revoked is rejected by `EnsureUserIsActive`.
- Telegram identity lookups and link attempts treat the account as inactive; a link code owned by a deactivated account is reported exactly like an unknown code.
- The account's rows remain in place. No cascade is triggered from the user row.

### Reactivation

Reactivation sets `is_active` to `true` and records a `user.reactivated` audit event.

Tokens, sessions, and Telegram links are not restored. The user signs in again, and a removed Telegram link is re-established through the normal linking flow.

### Password Recovery

Password recovery uses one mechanism with two delivery channels. The Owner never sees, chooses, or communicates a password.

| Method | Path | Access | Purpose |
|---|---|---|---|
| `POST` | `/api/v1/auth/forgot-password` | Public, throttled | Request a reset link by email |
| `POST` | `/api/v1/auth/reset-password` | Public, throttled | Consume a reset token and set a new password |

Self-service channel:

- `forgot-password` accepts an email address and always returns the same generic `200` response (`PASSWORD_RESET_REQUESTED`), whether or not an active account with that email exists.
- When an active account matches **and its email is verified**, a reset token is issued and a link is sent to that address.
- Email verification is not implemented yet, so the CLI channel is the operational recovery path until it is. Implementing email verification is a prerequisite for enabling the self-service channel.

Operator channel:

- `php artisan xibra:issue-password-reset {username}` issues the same kind of token and prints a single-use reset link.
- This channel exists because email is optional and mail delivery may not be configured. It requires server access, is recorded as a `system` actor with channel `cli`, and is intended for the Owner/operator.
- The printed link is a bearer credential. Deliver it over a secure channel and never paste it into tickets, chat history, or logs.

Token design:

| Property | Rule |
|---|---|
| Storage | A new `password_reset_tokens` table, keyed by `user_id`; only a hash of the token is stored |
| Token | At least 32 random bytes, generated with a CSPRNG |
| Lifetime | 60 minutes |
| Active tokens | One per user; issuing a new token invalidates the previous one |
| Use | Single use; marked consumed before the password update commits |
| Base URL | Configurable with `XIBRA_PASSWORD_RESET_URL` for the dashboard link |

`reset-password` accepts `token`, `password`, and `password_confirmation`. It:

1. Resolves the token hash to an unexpired, unused record.
2. Refuses deactivated accounts.
3. Applies the same password policy as onboarding (`Password::min(12)->max(72)`).
4. Updates the password and marks the token used in one transaction.
5. Revokes every existing Sanctum token for the account.
6. Records `user.password_reset_completed` without any token, hash, or password value.

The endpoint does not log the user in. The user signs in with the new password. Unknown, expired, and already-used tokens all return the same `422` with `PASSWORD_RESET_TOKEN_INVALID`, so the response reveals nothing about accounts.

Throttling:

- `forgot-password`: 5 requests per minute per normalized email and 5 per minute per client address.
- `reset-password`: 5 requests per minute per client address.

Mail delivery failures do not change the generic response and are logged without secrets. The token still expires normally.

Audit actions:

- `user.password_reset_issued` with channel `email` or `cli`.
- `user.password_reset_completed`.

No password, reset token, token hash, or email content is written to audit records or logs.

## Consequences

### Positive

- Access can be revoked immediately without losing historical records.
- The Owner can restore access without recreating an account or re-inviting a user.
- Password recovery works for accounts with or without an email address.
- No shared or temporary password ever exists.
- Every lifecycle and recovery action is attributable.

### Negative

- Accounts without an email depend on an operator-run CLI command.
- Self-service email recovery is inactive until email verification is implemented.
- Self-service deactivation is unavailable in v1.
- Deactivation, not deletion, means personal data is retained until a future deletion decision.
- Self-service recovery depends on mail infrastructure that is not production-ready yet.

## Security Implications

- Reset tokens are bearer secrets: hashed at rest, single use, short-lived, throttled, and never logged.
- A successful reset revokes all existing tokens to evict a possible attacker.
- Deactivation revokes all tokens, pending Telegram link codes, and active reset tokens in the same transaction as the status change.
- Login still verifies the password before revealing account status, so deactivation cannot be used for account enumeration.
- Lifecycle changes require an Owner permission and produce audit records.
- The CLI channel requires server access, is audited as a system action, and never prints a password. The operator nevertheless handles a short-lived, single-use bearer link, so issuance and completion are both audited.

## Alternatives Considered

### Self-Service Deactivation

Members deactivate their own accounts and only the Owner can reactivate them.

Deferred because it adds lockout and support scenarios without granting a capability members need. It can be added later through a new decision.

### Owner-Only Hard Delete

Add an Owner-only delete/anonymization endpoint now.

Rejected because deletion behavior must be decided per data domain, and no future domain has a retention policy yet. Deactivation provides immediate revocation without that commitment.

### Email-Only Recovery

Use only the email channel and require users without an email to be re-invited.

Rejected because email is optional by design, mail is not configured, and replacing an account to recover access would orphan history.

### Owner-Set Temporary Password

The Owner assigns a temporary password and requires a change at next login.

Rejected because it creates a shared secret, exposes the password to the Owner, and weakens non-repudiation.

### Laravel Default Email Password Broker

Use the framework's default broker and `password_reset_tokens` table keyed by email.

Rejected as the only mechanism because it cannot serve accounts without an email. The stored-hash, single-use approach is reused, keyed by `user_id` instead.

### Stateless Signed Reset Links

Use a signed, expiring URL without stored state.

Rejected because single-use enforcement and explicit invalidation require server-side state. A signed link cannot be reliably revoked after use.

## Review Conditions

This decision may be reconsidered if:

- Self-service account deletion, export, or deactivation becomes a requirement.
- Account volume makes Owner-only lifecycle administration impractical.
- Mail infrastructure is production-ready, email verification is implemented, and the CLI channel should be restricted or removed.
- A data domain introduces retention obligations that require a deletion or anonymization procedure.

Any change must be recorded through a new or superseding ADR.

## References

- `docs/api/conventions.md` sections 7 and 11
- `docs/database/README.md` sections 7.1 and 13
- `docs/decisions/0001-central-api-and-database.md`
- `xibra-api/app/Http/Controllers/Auth/AuthController.php`
- `xibra-api/app/Http/Middleware/EnsureUserIsActive.php`
- `xibra-api/app/Http/Requests/Onboarding/CompleteOnboardingRequest.php`
