# Xibra Platform — Technical Documentation

This directory contains shared technical documentation for the Xibra Platform workspace.

These documents define the platform architecture, API standards, database principles, and significant technical decisions.

They serve as shared references for all applications and integrations.

## Documentation Structure

| Directory | Purpose |
|---|---|
| `architecture/` | System architecture, application boundaries, and multi-user design |
| `api/` | API design, conventions, security, and integration standards |
| `database/` | Database design principles, ownership, and data integrity |
| `decisions/` | Architecture Decision Records (ADRs) |

## Documents

### Architecture

- [System Architecture Overview](architecture/overview.md)

### API

- [API Conventions](api/conventions.md)

### Database

- [Database Documentation](database/README.md)

### Architecture Decisions

- [ADR 0001: Central API and Database Ownership](decisions/0001-central-api-and-database.md)

## Documentation Principles

1. All technical documentation must be written in English.
2. Document significant architectural decisions and their reasoning.
3. Keep architecture, API, and database documentation consistent.
4. Update relevant documentation when implementation changes.
5. Clearly distinguish planned functionality from implemented functionality.
6. Prefer explicit technical requirements and practical examples.
7. Keep application-specific instructions in the relevant application repository.
8. Do not introduce conflicting technical requirements across documents.
9. Review security and data ownership implications before documenting new integrations or capabilities.

## Architecture Documentation Scope

Shared documentation should cover platform-wide concerns, including:

- Multi-user architecture.
- Application responsibilities and communication boundaries.
- Authentication and authorization.
- Resource ownership and data isolation.
- Invitation-only onboarding.
- Database ownership and integrity.
- External integrations.
- AI context isolation and approval requirements.

Application-specific implementation details belong in the corresponding application repository.

## Architecture Decision Records

Significant technical decisions must be documented as Architecture Decision Records.

Each ADR should describe:

- Context and problem.
- Decision.
- Reasons for the decision.
- Consequences and trade-offs.
- Relevant alternatives when applicable.

ADRs should be numbered sequentially and must not silently overwrite previously accepted decisions.

## Maintenance

The project owner is responsible for reviewing and maintaining these documents.

Documentation must remain consistent with:

- Root `README.md`.
- Root `IDEA.md`.
- Root `TODO.md`.
- Root `CLAUDE.md`.
- Application-specific documentation.
- Actual application implementations and database migrations.

Documentation describes intended architecture unless implementation has been verified.