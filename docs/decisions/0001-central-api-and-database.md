# ADR 0001: Central API and Database Ownership

## Status

Accepted

## Date

2026-11-11

## Context

Xibra Platform consists of multiple independent applications:

- `xibra-api`
- `xibra-portfolio`
- `xibra-dashboard`
- `xibra-bot`
- `xibra-core`

These applications need to access shared platform data and business capabilities.

Xibra Platform is designed as a multi-user ecosystem. A single platform deployment serves multiple independent users with isolated identities, private resources, permissions, integrations, and AI contexts.

Allowing every application to connect directly to the database would create duplicated business logic, inconsistent authorization, tighter coupling, and additional security risks.

Maintaining separate primary databases for each application or user would also complicate shared business processes and authoritative data management.

A consistent data access strategy is therefore required.

## Decision

Xibra Platform will use a centralized backend architecture with one shared primary PostgreSQL database.

`xibra-api` is the only application permitted to access the primary PostgreSQL database directly.

All other applications must communicate through `xibra-api` using approved API endpoints and authenticated service integrations.

The API is responsible for:

- User identity and authentication.
- Authorization and permission enforcement.
- Resource ownership validation.
- Request validation.
- Business logic.
- Database operations.
- Data integrity.
- API response consistency.
- Integration security boundaries.

PostgreSQL remains the primary source of truth.

### Multi-User Data Model

The platform uses one shared application deployment and database to serve multiple independent users.

Private user-owned resources must follow explicit ownership rules.

The API must enforce user isolation and prevent unauthorized access to another user's private records.

Role permissions and resource ownership are separate authorization requirements.

Platform ownership does not automatically grant unrestricted access to every user's private data.

The platform must not create a separate application or primary database for each user unless a future architecture decision explicitly approves such a change.

## Consequences

### Positive

- Centralized business logic.
- Consistent authentication and authorization.
- Enforced resource ownership and user isolation.
- Reduced database exposure.
- Clear application responsibilities.
- Easier integration between applications.
- One primary source of truth.
- Shared infrastructure for multiple users.
- Independent frontend and client development.
- Consistent security boundaries across integrations.

### Negative

- The API becomes a critical infrastructure dependency.
- Applications depend on API availability.
- API design and versioning require discipline.
- Additional network communication is required.
- The API must enforce ownership checks across all private resources.
- Shared infrastructure requires careful performance and security management.

## Security Implications

- PostgreSQL must not be directly accessible to frontend applications.
- Only `xibra-api` may execute application database queries.
- Bot and AI integrations must use approved and authenticated API capabilities.
- Protected operations require authentication and authorization.
- Private resource access requires ownership validation.
- Client-provided user identifiers must never establish ownership.
- Sensitive actions require explicit approval.
- AI context and external integration credentials must be isolated per user.
- Database credentials must remain private.
- Service authentication must not automatically grant unrestricted user privileges.

## Alternatives Considered

### Direct Database Access

Each application connects directly to PostgreSQL.

Rejected because it distributes data access responsibilities and increases security and maintenance complexity.

### Separate Primary Database Per Application

Each application maintains its own independent primary database.

Rejected because the platform requires shared authoritative data and centralized business rules.

### Separate Primary Database Per User

Every user receives an independent primary database.

Rejected because the initial platform architecture requires shared infrastructure, centralized data management, and a universal dashboard.

This option may only be reconsidered if future isolation, compliance, or scaling requirements justify the additional operational complexity.

## Review Conditions

This decision may be reconsidered if:

- Platform scale introduces a measurable architectural limitation.
- A new integration requires a documented exception.
- Operational or compliance requirements justify a different data architecture.
- Database isolation requirements materially change.
- A future infrastructure change requires a different ownership model.

Any change must be documented through a new or superseding ADR.

Previously accepted decisions must not be silently rewritten to represent a different architectural decision.

## References

- `IDEA.md`
- `README.md`
- `docs/architecture/overview.md`
- `docs/database/README.md`
- `docs/api/conventions.md`