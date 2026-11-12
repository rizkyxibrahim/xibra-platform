# Xibra Platform — System Architecture

## 1. Overview

Xibra Platform uses a centralized, multi-user backend architecture.

The platform consists of independent applications connected through one central backend and one primary data source.

A single platform deployment serves multiple independent users. Each user has an isolated identity, private resources, integrations, permissions, and AI context.

The architecture separates presentation, integration, business logic, authentication, authorization, and data persistence.

The central API is the authoritative source of truth for all connected applications.

## 2. High-Level Architecture

```text
                         XIBRA PLATFORM
                    Multi-User Application Layer

    +-------------------+       +-------------------+
    |  xibra-portfolio  |       |  xibra-dashboard  |
    |   Public Website  |       | Universal Private |
    |                   |       |    Dashboard      |
    +---------+---------+       +---------+---------+
              |                           |
              |       HTTPS / REST API    |
              |                           |
    +---------+---------------------------+---------+
    |                  xibra-api                     |
    |                                                |
    | Identity | Authentication | Authorization     |
    | Validation | Business Logic | API Responses   |
    |                                                |
    | Invitation | Ownership | Integration Services |
    +-------------------------+----------------------+
                              |
                              | Database Connection
                              |
                       +------+------+
                       | PostgreSQL  |
                       +-------------+

    +-------------------+       +-------------------+
    |     xibra-bot     |       |    xibra-core     |
    | Telegram Interface|       |   Hermes Agent    |
    +---------+---------+       +---------+---------+
              |                           |
              +-------------+-------------+
                            |
                     Authenticated API
```

All applications communicate through `xibra-api`.

Only `xibra-api` may access PostgreSQL directly.

`xibra-core` is the single Hermes AI agent. It coordinates approved workflows through authorized API operations.

## 3. Multi-User Architecture

Xibra Platform is designed to serve multiple independent users through one shared application deployment.

Each user has:

- An independent account and identity.
- Private application data.
- Assigned roles and permissions.
- Personal integrations and credentials.
- Isolated AI conversations and memory.
- Their own authorized application access.

### 3.1 Universal Dashboard

`xibra-dashboard` is one universal private application.

All users access the same Dashboard deployment and application.

The API determines which resources and operations are available based on:

- Authenticated user identity.
- Assigned roles.
- Granted permissions.
- Resource ownership.
- Explicit sharing authorization, when supported.

Users do not receive separate Dashboard installations or independent application deployments.

### 3.2 Data Ownership and Isolation

Every private resource must have an explicit ownership model.

The API must:

- Derive the authenticated user from a trusted authentication context.
- Validate resource ownership on every private operation.
- Prevent users from accessing another user's private records.
- Prevent unauthorized ownership reassignment.
- Apply authorization to nested resources and related records.
- Isolate integrations and AI context by user.

Client-provided user identifiers must never be treated as proof of ownership.

Role permissions and resource ownership are separate authorization requirements.

Platform ownership does not automatically grant unrestricted access to every user's private data.

The concrete ownership, isolation, and deletion rules are recorded in ADR 0003. Account lifecycle and password recovery are recorded in ADR 0002, and audit visibility and retention in ADR 0004.

## 4. Component Responsibilities

### 4.1 xibra-api

The central application responsible for:

- User identity and account management.
- Authentication and authorization.
- Role-based access control.
- Invitation management and onboarding.
- Request validation.
- Business logic.
- Database operations.
- Data ownership enforcement.
- API response formatting.
- Integration boundaries.
- Security-sensitive operations and audit records.

The API is the only application permitted to access PostgreSQL.

### 4.2 xibra-portfolio

The public website responsible for presenting professional information and published content.

Responsibilities:

- Public profile presentation.
- Resume presentation.
- Published projects.
- Published blog content.
- Contact submission.

It must use explicitly designated public API endpoints and must not access PostgreSQL directly.

Private user information must never be exposed through public endpoints.

### 4.3 xibra-dashboard

The universal private interface responsible for managing platform data and authorized operations.

Responsibilities:

- User account and profile management.
- Professional portfolio management.
- Platform administration according to permissions.
- Invitation management for the Owner.
- Productivity and scheduling modules.
- Personal finance.
- Knowledge management.
- Integration settings.
- AI assistant interface.

It must use authenticated API endpoints.

The Dashboard must not implement independent authorization rules that conflict with the API.

Hiding a UI element is not a substitute for backend authorization.

### 4.4 xibra-bot

The Telegram interface responsible for receiving user commands and presenting platform information.

Responsibilities:

- User interaction through Telegram.
- Invitation redemption.
- Telegram account linking.
- Productivity workflows.
- Scheduling workflows.
- Reminder notifications.
- Approved finance workflows.
- AI assistant interaction.

It must communicate with `xibra-api` through authenticated service-to-service requests.

Telegram numeric user IDs must originate from trusted Telegram updates.

Telegram usernames must not be treated as proof of identity.

The bot must not access PostgreSQL directly.

### 4.5 xibra-core

The single Hermes AI agent responsible for interpreting requests and coordinating approved assistant workflows.

Responsibilities:

- Natural language understanding.
- Context-aware assistance.
- Productivity and scheduling assistance.
- Research and summarization.
- Approved API operations.
- Sensitive action approval workflows.

It must respect API authorization and user context isolation.

It must not access PostgreSQL directly or bypass API security boundaries.

AI requests must execute within the correct user's authorized context.

## 5. Communication Rules

1. Applications communicate through HTTPS in production.
2. REST endpoints use the versioned API prefix `/api/v1`.
3. Protected operations require appropriate authentication.
4. Service-to-service requests require approved service authentication.
5. Authorization is enforced by the backend.
6. Resource ownership is validated by the backend.
7. Request validation occurs at the API boundary.
8. API responses follow a shared JSON contract.
9. Clients must not bypass backend business rules.
10. Sensitive operations require explicit approval where defined.

The API is responsible for validating the identity and authority of every request.

The exact service-to-service authentication mechanism must be documented and approved before implementation.

## 6. Database Ownership

PostgreSQL is owned and accessed by `xibra-api`.

Other applications must never:

- Connect directly to PostgreSQL.
- Execute SQL against the primary database.
- Maintain independent copies of authoritative business records.
- Bypass API authorization.
- Use database credentials to access platform data.

The API is the single application-facing data boundary.

Private data must be associated with the correct user and protected through ownership validation.

Database transactions must be used for operations that require atomic consistency, including invitation redemption and account creation.

## 7. Authentication and Security Boundaries

### 7.1 Identity

Users authenticate through supported account identifiers:

- Username.
- Email address.
- Phone number.

Username is required. Email and phone number are optional but must be unique when populated.

Authentication is handled centrally by `xibra-api`.

### 7.2 Public Access

Public portfolio content may be exposed through explicitly designated public endpoints.

Public endpoints must return only approved public information.

They must not expose private profiles, unpublished content, credentials, or internal platform data.

### 7.3 Private Access

Dashboard and other protected workflows require authenticated access.

The API enforces:

- Account authentication.
- Role-based permissions.
- Resource ownership.
- Account status.
- Operation-specific authorization.

### 7.4 Invitation-Only Onboarding

Public self-registration is not available.

New users join through an invitation created by the platform Owner.

Invitations can be managed through Telegram Bot or Dashboard, using the same central API invitation system.

Approved invitation behavior:

- Default expiration: 24 hours.
- Minimum expiration: 1 hour.
- Maximum expiration: 72 hours.
- One-time redemption.
- Default assigned role: `member`.
- Supported statuses: `pending`, `redeemed`, `expired`, and `revoked`.

Invitation secrets must be generated securely and stored as cryptographic hashes.

The API validates the invitation and controls account creation.

Passwords must only be submitted through a secure HTTPS onboarding page, never through Telegram.

A short-lived onboarding session must prevent concurrent or repeated invitation claims.

### 7.5 Telegram Identity

Telegram identity must be linked through a trusted onboarding or account-linking workflow.

The API must verify the bot service identity before accepting Telegram identity information. The bot authenticates with HMAC-signed service requests; the implemented contract is documented in `docs/api/conventions.md` section 12.

A Telegram identity must not be linked to multiple Xibra accounts.

Existing accounts require account verification before Telegram linking.

### 7.6 AI Access

Hermes must use approved integration capabilities.

AI access must not automatically grant administrator privileges.

AI operations must respect the authenticated user's context and permissions.

Sensitive operations require explicit user approval.

AI conversations, memory, and private context must remain isolated per user.

### 7.7 External Integrations

External integrations belong to individual users.

For Google integrations:

- Each user authorizes their own Google account.
- OAuth credentials are associated with the correct user.
- Tokens are encrypted at rest.
- Users can revoke their connections.
- Gmail and Google Chat use their respective authorization scopes.

One user's integration credentials must never be accessible to another user.

## 8. Deployment Direction

Planned production routing:

| Hostname | Application | Runtime |
|---|---|---|
| `rizkyxibrahim.my.id` | xibra-portfolio | Nuxt / Node.js |
| `www.rizkyxibrahim.my.id` | xibra-portfolio | Nginx redirect or canonical routing |
| `app.rizkyxibrahim.my.id` | xibra-dashboard | Nuxt / Node.js |
| `api.rizkyxibrahim.my.id` | xibra-api | Nginx / PHP-FPM |

Telegram Bot and Hermes are planned as background services without requiring public web endpoints.

PostgreSQL must remain accessible only through the server's local application connection.

All application secrets must be stored outside source control.

Production services must use appropriate operating-system users and file permissions.

## 9. Architecture Evolution

Changes to the architecture must be deliberate.

Before introducing a new application, database, integration, or runtime:

1. Identify the problem it solves.
2. Evaluate whether an existing component can handle it.
3. Define ownership and security boundaries.
4. Evaluate operational and security implications.
5. Document significant decisions.
6. Update related architecture documentation.
7. Obtain owner approval before implementation.

Do not introduce independent databases or duplicate authoritative business logic without an approved architecture decision.

## 10. Current Implementation Status

This document describes the intended architecture.

It does not imply that all applications, endpoints, integrations, security mechanisms, or deployment configurations have already been implemented.

Implementation progress must be tracked in the root `TODO.md` and verified against the actual application code, database migrations, and tests.