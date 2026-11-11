# ADR 0001: Central API and Database Ownership

## Status

Accepted

## Date

2026-11-11

## Context

Xibra Platform consists of multiple independent applications:

- xibra-portfolio
- xibra-dashboard
- xibra-bot
- xibra-core

These applications need to access shared platform data and business capabilities.

Allowing every application to connect directly to the database would create duplicated business logic, inconsistent authorization, tighter coupling, and additional security risks.

A consistent data access strategy is therefore required.

## Decision

Xibra Platform will use a centralized backend architecture.

xibra-api is the only application permitted to access the primary PostgreSQL database directly.

All other applications must communicate through xibra-api using approved API endpoints.

The API is responsible for:

- Authentication.
- Authorization.
- Request validation.
- Business logic.
- Database operations.
- Data integrity.
- API response consistency.

PostgreSQL remains the primary source of truth.

## Consequences

### Positive

- Centralized business logic.
- Consistent authorization.
- Reduced database exposure.
- Clear application responsibilities.
- Easier integration between applications.
- One primary source of truth.
- Independent frontend and client development.

### Negative

- The API becomes a critical infrastructure dependency.
- Applications depend on API availability.
- API design and versioning require discipline.
- Additional network communication is required.

## Security Implications

- PostgreSQL must not be directly accessible to frontend applications.
- Bot and AI integrations must use approved API capabilities.
- Protected operations require authentication and authorization.
- Sensitive actions require explicit approval.
- Database credentials must remain private.

## Alternatives Considered

### Direct Database Access

Each application connects directly to PostgreSQL.

Rejected because it distributes data access responsibilities and increases security and maintenance complexity.

### Separate Primary Database Per Application

Each application maintains its own independent primary database.

Rejected because the platform requires shared authoritative data and centralized business rules.

## Review Conditions

This decision may be reconsidered if:

- Platform scale introduces a measurable architectural limitation.
- A new integration requires a documented exception.
- Operational requirements justify a different data architecture.

Any change must be documented through a new or superseding ADR.

## References

- IDEA.md
- docs/architecture/overview.md
- docs/database/README.md
- docs/api/conventions.md
