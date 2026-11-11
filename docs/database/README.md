# Xibra Platform — Database Documentation

## 1. Overview

PostgreSQL is the primary relational database for Xibra Platform.

The database is managed exclusively by xibra-api.

All other applications must access platform data through documented API endpoints.

## 2. Database Ownership

The following rules are mandatory:

1. xibra-api owns the database connection.
2. Only xibra-api executes database queries.
3. Frontend applications must never connect directly to PostgreSQL.
4. Telegram bot and Hermes must communicate through xibra-api.
5. Business rules and data integrity must be enforced by the backend.
6. The database must remain the primary source of truth.

## 3. Database Design Principles

- Use relational modeling where appropriate.
- Define primary keys explicitly.
- Define foreign keys for relational integrity.
- Add unique constraints where required.
- Use appropriate indexes for frequently queried fields.
- Use explicit nullability rules.
- Avoid storing duplicated authoritative data.
- Use transactions for operations involving multiple related records.
- Store timestamps consistently and handle application time zones explicitly.
- Avoid premature optimization and unnecessary tables.

## 4. Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Tables | Plural snake_case | `blog_posts` |
| Columns | snake_case | `created_at` |
| Primary keys | `id` | `id` |
| Foreign keys | Singular model name plus `_id` | `user_id` |
| Indexes | Framework-generated names where practical | `users_email_unique` |
| Pivot tables | Consistent relationship naming | `project_tag` |

Laravel migrations and Eloquent conventions should be followed unless a documented reason requires an exception.

## 5. Migrations

- Every schema change must be represented by a migration.
- Migrations must be committed to Git.
- Migrations must be reviewed before production execution.
- Avoid editing migrations that have already been deployed to shared environments.
- Use new migrations for subsequent schema changes.
- Test migrations against a clean development database.
- Consider rollback behavior before applying schema changes.

## 6. Seeders

Seeders provide controlled initial and reference data.

Planned uses include:
- Initial administrator provisioning.
- Reference categories.
- Default configuration.
- Required system records.

Seeders must be repeatable where practical.

Never hardcode production passwords or secrets in committed seeders.

Administrator provisioning must use a controlled and secure process because public registration is not part of the platform.

## 7. Data Integrity

- Enforce required relationships through foreign keys.
- Use database constraints for critical integrity rules.
- Validate incoming data through the API.
- Use database transactions for multi-step operations.
- Define deletion behavior deliberately.
- Prevent unauthorized access to records.
- Maintain consistent ownership rules for personal data.

## 8. Indexing

Indexes should be introduced based on actual query patterns.

Consider indexing:
- Foreign keys.
- Frequently filtered columns.
- Frequently sorted columns.
- Unique identifiers.
- Common lookup combinations.

Avoid adding indexes without considering write overhead and storage cost.

## 9. Database Security

- Keep PostgreSQL inaccessible from public networks unless explicitly required by a reviewed architecture decision.
- Use a dedicated database role for the API.
- Apply least-privilege permissions.
- Store credentials in environment configuration.
- Never commit database passwords.
- Back up data before destructive operations.
- Verify backup restoration periodically.

## 10. Backup and Recovery

Production database operations must include:
- Scheduled backups.
- Secure backup storage.
- Documented retention policy.
- Backup integrity checks.
- Tested restoration procedures.
- Documented recovery steps.

A backup is not considered reliable until restoration has been tested.

## 11. Planned Data Domains

The following domains are planned for gradual implementation:

- Identity and access.
- Portfolio content.
- Productivity.
- Schedule and workforce.
- Personal finance.
- Knowledge management.
- AI activity and approvals.
- Integrations and notifications.

These domains are architectural planning categories, not existing database tables.

Each domain must be designed and reviewed before its migrations are implemented.

## 12. Implementation Status

This document establishes database design principles.

It does not imply that the production schema, migrations, seeders, backups, or all planned domains already exist.
