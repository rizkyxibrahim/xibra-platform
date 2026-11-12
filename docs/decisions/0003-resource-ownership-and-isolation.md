# ADR 0003: Resource Ownership and Data Isolation

## Status

Accepted

## Date

2026-11-12

## Context

Xibra Platform uses one shared PostgreSQL database for multiple independent users. ADR 0001 establishes that only `xibra-api` accesses the database and that role permissions and resource ownership are separate authorization requirements.

The identity, invitation, onboarding, Telegram, and audit tables already exist. The next feature batch introduces the first private user-owned resource (portfolio content or another approved domain), so the ownership convention must be concrete before any table, policy, route, or test is written.

Existing rules in `docs/database/README.md` and `docs/api/conventions.md` describe the principles but do not define table classification, nullability, deletion behavior, nested-resource handling, or the response used when a user requests another user's record.

## Decision

### Table Classification

Every table must be classified as exactly one of:

| Classification | Meaning | Current examples |
|---|---|---|
| User-owned | Rows belong to exactly one user and are reachable only through that user's authorized context. | `telegram_identities`, `telegram_link_codes`, `personal_access_tokens`, future portfolio, productivity, finance, and integration records |
| Shared reference | Read-mostly platform reference data used by many users; not private to one account. | `roles`, `permissions`, `role_user`, `permission_role` |
| System-managed | Platform records written by the API or a trusted service, with their own authorization model. | `users`, `invitations`, `onboarding_sessions`, `onboarding_attempts`, `audit_logs` |

A new table must be classified in the same change that introduces it. The maintained classification list lives in `docs/database/README.md` section 4.2. Ambiguous ownership is not permitted.

`users` is system-managed identity data. A user can read their own profile through `/api/v1/auth/me`; profile update, when added, is scoped to the same authenticated user. The account row itself is not an ordinary owned aggregate and is not transferred or reassigned.

### User-Owned Table Rules

- The aggregate root has `user_id BIGINT NOT NULL` with a foreign key to `users.id`, indexed for ownership queries.
- `user_id` is never fillable and is never accepted from request input. It is assigned only from the authenticated user's relationship at creation time.
- Composite indexes are added when real query patterns require them, for example `(user_id, created_at)` or `(user_id, status)`.
- Unique constraints that are per-user include `user_id`, for example `unique(['user_id', 'slug'])`.
- Child records inherit ownership through their parent foreign key. A child table may omit `user_id` when it cannot be reached independently and every query resolves the parent first.
- Deletion behavior is explicit per relationship. Aggregate roots use `restrictOnDelete` unless the child is meaningless without the parent and the aggregate is only ever removed as a unit.
- Existing Telegram tables use `cascadeOnDelete` because a Telegram identity or link code has no meaning without its user. They remain as-is, and v1 exposes no user deletion endpoint. Future aggregates that may need retention use `restrictOnDelete`.
- There is no ownership reassignment operation. Changing a user's role never transfers records.
- Cross-user sharing is not supported. A future sharing feature requires an explicit grant table and a new decision; it must not be simulated by accepting a `user_id`, weakening a policy, or reusing the Owner role.

### Authorization Rules

Authorization requires two independent checks:

1. The authenticated user has the required role or permission.
2. The authenticated user owns the target record, or the record is explicitly shared with them.

The Owner role grants administrative capability, not implicit access to member private records. Owner-only administration endpoints (`/users`, `/invitations`, `/audit-logs`, `/onboarding-attempts`) are separate, permission-gated routes and never reuse private-resource routes.

Private resource access follows these rules:

- Lists are built from the authenticated user's relationship, for example `$request->user()->projects()->paginate(...)`.
- Single-record lookups resolve through the same relationship, for example `$request->user()->projects()->findOrFail($id)`, or through a route binding callback with the same query.
- Nested routes use `scopeBindings()` and resolve each segment through its parent. The parent's ownership is checked before the child is returned.
- Policies implement `viewAny`, `view`, `create`, `update`, and `delete` for the model. Policies are defense-in-depth; the ownership-scoped query is the primary boundary.
- A policy must never compare a client-supplied identifier to `user_id`.
- Requests may not include `user_id`, `owner_id`, or an equivalent field. Form requests whitelist accepted fields; unexpected ownership fields are rejected or ignored by validation, never applied.
- A user requesting another user's private record receives `404` with `NOT_FOUND`. `403` is reserved for a known shared or system resource where the caller lacks a role or permission. The `404` choice prevents private record existence from leaking.
- Global Eloquent scopes are not the security boundary. They are optional for convenience only; `withoutGlobalScopes()` must not be able to expose another user's rows because the base query is already constrained by the authenticated relationship.
- API Resources expose only safe fields. Public portfolio responses use a separate read model, return only published rows and public columns, and never expose contact details, account identifiers, or private metadata.

### Deletion

v1 uses account deactivation instead of deletion (ADR 0002). No endpoint deletes a user or an owned aggregate.

All ownership foreign keys are deliberate: `restrictOnDelete` for records with retention value, `cascadeOnDelete` only for records that cannot exist independently. A future deletion or anonymization procedure must define behavior for every domain before any endpoint is added.

### Isolation Test Matrix

Every owned resource must ship with tests covering at least:

| Case | Expected result |
|---|---|
| List as user A | Only user A's records are returned |
| Show user B's record as user A | `404` |
| Update user B's record as user A | `404`, record unchanged |
| Delete user B's record as user A | `404`, record unchanged |
| Create with `user_id` of user B in the body | Field is not applied; record belongs to user A |
| Nested child under user B's parent as user A | `404` |
| Owner token on a member's private record | `404` |
| Unauthenticated or inactive account | `401` or `403` per the authentication contract, never data |

Tests must assert both the response and the database state.

### Implementation Pattern

- Policies live in `app/Policies` and are discovered or registered explicitly.
- Controllers authorize the action before executing it and always obtain records through the authenticated user's relationships.
- Form requests define the accepted field set and never include ownership fields.
- Resource classes expose stable public fields; they do not expose `user_id` unless the client genuinely needs it, and never expose another user's identifiers.
- Isolation tests live in `tests/Feature/Isolation`.
- New migrations add the ownership foreign key, index, and any per-user unique constraint in the same migration that creates the table.

## Consequences

### Positive

- The first owned resource has an unambiguous schema, route, policy, and test pattern.
- Cross-user access fails closed and does not reveal whether a record exists.
- The Owner cannot accidentally browse member private data through ordinary resource endpoints.
- Deactivation remains the only lifecycle operation, so no cascade can destroy retained history.
- The model scales to future domains without a per-user database or schema.

### Negative

- Every owned resource needs an explicit policy and an isolation test suite.
- Nested routes require deliberate parent resolution rather than implicit global binding.
- Returning `404` instead of `403` can make some client error messages less specific.
- The Owner needs separate administration endpoints for tasks that might otherwise be simple cross-user queries.

## Security Implications

- Ownership comes only from the trusted authentication context.
- Query scoping is the primary boundary; policies are a second check.
- Mass assignment cannot set ownership.
- Role changes and administrative actions cannot move private records between accounts.
- Public content is published from a separate read model, not by relaxing ownership on the private model.
- Every ownership foreign key has an explicit deletion behavior.

## Alternatives Considered

### Global `BelongsToUser` Scope as the Primary Boundary

Apply a global scope to every owned model and rely on it for isolation.

Rejected as the primary boundary because global scopes can be bypassed with `withoutGlobalScopes()` and are easy to forget in raw or aggregate queries. A query already constrained by the authenticated relationship is not bypassable the same way. A convenience scope may still be added later.

### Owner Can Read All Private Data

Allow the Owner role to bypass ownership for administration.

Rejected because it conflicts with ADR 0001 and turns the Owner into a superuser over private member data. Administration needs are served by explicit, audited endpoints with their own permission.

### Return `403` for Cross-User Access

Rejected because `403` confirms that the record exists. `404` keeps private record existence confidential.

### Separate Schema or Database Per User

Rejected by ADR 0001. It adds operational complexity without improving isolation beyond correct backend enforcement.

### Opaque Identifiers Instead of Ownership Checks

Use UUIDs or ULIDs to make records hard to guess.

Rejected as a substitute for authorization. Identifiers may still use UUIDs or ULIDs for other reasons, but ownership is enforced by the API regardless of identifier format.

## Review Conditions

This decision may be reconsidered if:

- A product requirement introduces explicit sharing between users.
- A future domain requires deletion or anonymization rather than deactivation.
- Auditing or compliance requires owner-level read access to member private records.
- Query performance shows that relationship-scoped queries need a different indexing or caching strategy.

Any change must be recorded through a new or superseding ADR.

## References

- `docs/decisions/0001-central-api-and-database.md`
- `docs/decisions/0002-account-lifecycle-and-password-recovery.md`
- `docs/database/README.md` section 4
- `docs/api/conventions.md` sections 8, 9, and 10
- `xibra-api/app/Models/User.php`
- `xibra-api/app/Models/TelegramIdentity.php`
- `xibra-api/app/Models/TelegramLinkCode.php`
