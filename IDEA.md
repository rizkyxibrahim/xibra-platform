# Xibra Platform — Product Vision and Architecture

## 1. Vision

Xibra Platform is a multi-user personal digital ecosystem designed to connect professional identity, productivity, work management, personal finance, knowledge management, and AI-assisted workflows.

The platform provides shared applications and centralized backend services while maintaining isolated personal data, integrations, and configurations for every user.

The goal is to build one integrated ecosystem where multiple applications share a central backend and a single source of truth without compromising individual user privacy.

Xibra initially serves its platform owner and invited users, with an architecture designed to support additional users as the platform evolves.

## 2. Product Goals

- Maintain an optional public professional identity for each user.
- Provide a universal private dashboard with personalized data and modules.
- Allow users to access selected platform capabilities through Telegram.
- Provide an AI assistant with isolated user context and conversations.
- Support individual integrations, including Google services and AI providers.
- Manage personal activities, productivity, work schedules, and finances.
- Keep data consistent across all applications.
- Protect personal data through server-enforced ownership and authorization.
- Expand the platform gradually through independent modules.

## 3. Core Applications

### 3.1 xibra-api

The central backend and the only application permitted to access the primary PostgreSQL database directly.

Responsibilities:

- User authentication and authorization.
- Invitation-based user onboarding.
- Role-based access control (RBAC).
- API versioning and response conventions.
- Request validation.
- Business logic and data ownership enforcement.
- Database access and relationships.
- Third-party integration management.
- AI provider and conversation management.
- Automated testing.

All applications must use the API as their trusted backend.

The API must derive the authenticated user's identity from a verified authentication context. Client-provided user identifiers must never be trusted as proof of data ownership.

### 3.2 xibra-portfolio

The public-facing professional website.

Initial pages:

- About
- Resume
- Works
- Blog
- Contact

The portfolio consumes public API endpoints. Content management belongs to the private dashboard.

Each user may have one professional profile. Creating a professional profile is optional and is not required to use other Xibra applications.

Public portfolio content must only expose information explicitly published by its owner.

### 3.3 xibra-dashboard

A universal private control center for all Xibra users.

All users access the same dashboard application and deployment. The displayed data, enabled modules, and available actions depend on the authenticated user and their permissions.

Planned modules:

- Overview
- Personal profile and account settings
- Portfolio management
- Productivity
- Projects and tasks
- Schedule and workforce
- Personal finance
- Knowledge management
- AI assistant
- Connected accounts and integrations
- Activity and audit history
- Invitation management (Owner only)

The dashboard communicates with xibra-api and never connects directly to PostgreSQL.

Each user must only access their own private resources unless an explicit sharing mechanism grants additional access.

The dashboard must not rely on client-side filtering as its security boundary. Authorization and ownership validation must always be enforced by the API.

### 3.4 xibra-bot

A Telegram interface for accessing selected platform capabilities.

Initial access is through Telegram, with each Telegram account explicitly linked to an individual Xibra user.

Planned workflows:

- Task management
- Schedule management
- Reminders
- Finance entries
- Activity summaries
- AI-assisted requests
- Selected information retrieval
- Invitation management (Owner only)

The bot communicates exclusively with xibra-api.

Every incoming Telegram request must be mapped to a verified Xibra user account. Telegram identity must not be assumed to represent a user until account linking has been completed.

The bot may provide invitation creation and management interfaces to the Owner. Invitation authorization and lifecycle management remain the responsibility of xibra-api.

### 3.5 xibra-core

The single planned Hermes AI agent.

The platform will use exactly one Hermes agent named xibra-core.

Responsibilities:

- Understand natural language requests.
- Assist with productivity and planning.
- Retrieve and summarize authorized information.
- Support coding and research workflows.
- Interact with approved API capabilities.
- Maintain isolated conversation context for each user.
- Request approval before executing sensitive actions.

The agent must operate within the authenticated user's authorization boundaries.

User-specific conversations, memory, integrations, and data must never be mixed across users.

Hermes must not connect directly to PostgreSQL.

## 4. System Architecture

```text
                         Xibra Platform
                                |
                         +------+------+
                         |   xibra-api |
                         +------+------+
                                |
              +-----------------+-----------------+
              |                 |                 |
         PostgreSQL       Authentication        RBAC
              |
       User-owned data
              |
    +---------+----------+----------------+
    |                    |                |
Portfolio             Dashboard       Telegram Bot
(public)              (universal)      (linked users)
                         |
                  Authenticated API
                         |
                    xibra-core
                   (Hermes agent)
                         |
                   AI Providers
                  Gemini / Others

              External Integrations
                 Google OAuth
               (per-user access)
```

Architectural rules:

1. xibra-api is the central backend.
2. PostgreSQL is the primary data source.
3. Only xibra-api accesses PostgreSQL directly.
4. Every client communicates through the API.
5. Business logic and authorization belong to the API.
6. Every private resource must have a verifiable ownership relationship.
7. Every application must enforce the authenticated user's access boundaries.
8. The dashboard is universal and must provide user-specific data and permissions.
9. Telegram accounts must be explicitly linked to Xibra user accounts.
10. Third-party integrations must maintain individual user authorization.
11. AI conversations and context must remain isolated per user.
12. No application may maintain a competing source of truth.
13. One PostgreSQL database serves the platform; separate databases per user are not required.
14. Hermes must access platform capabilities through approved API interfaces rather than direct database access.
15. Invitation authorization and account creation must be enforced by xibra-api.

## 5. Planned Data Domains

These are planned business domains, not a statement that their database tables already exist.

Database tables and relationships must be designed incrementally before implementation.

### 5.1 Identity and Access

- Users
- Roles
- Permissions
- User-role assignments
- Role-permission assignments
- Authentication tokens
- Password reset records
- User invitations
- Invitation onboarding sessions
- Telegram account mappings

Authentication supports username, email, or phone number with a password.

Username is required. Email and phone number may be optional account attributes, but each populated value must be unique.

At least one verified recovery channel must be available for password recovery.

Public self-registration is not part of the initial onboarding model. New users are onboarded through Owner-controlled invitations.

### 5.2 User Profile and Portfolio

- Professional profiles
- Social links
- Experiences
- Education
- Skills
- Certifications
- Portfolio projects
- Project media
- Blog posts
- Contact messages

Each user may have one optional professional profile.

Portfolio content must be associated with its owner and only published content may be exposed publicly.

### 5.3 Productivity

- Projects
- Tasks
- Labels
- Reminders
- Routines
- Activity records

Private productivity records must be owned by their respective users.

### 5.4 Schedule and Workforce

- Workforce members
- Shift types
- Shift blocks
- Daily standby records

Personal schedules and workforce management must enforce appropriate ownership and role-based permissions.

### 5.5 Personal Finance

- Financial accounts
- Categories
- Income and expenses
- Transfers
- Budgets
- Debts and installments
- Savings goals
- Investments

Each user maintains their own financial accounts and transactions.

Financial data must not be accessible to other users unless a future, explicitly authorized sharing feature is implemented.

### 5.6 Knowledge Management

- Notes
- Tags
- References

Personal knowledge records must remain isolated by ownership.

### 5.7 AI and Integrations

- AI provider settings
- AI conversations
- AI messages
- Agent activity
- Approval requests
- OAuth connections
- OAuth tokens
- Notifications
- Audit logs

Third-party integrations must be associated with individual users.

Google OAuth credentials must be stored securely and must never be shared between users.

AI provider configuration may support platform-managed credentials and future user-provided credentials.

AI context and conversation history must follow the same user ownership boundaries as other private data.

## 6. Multi-User and Data Isolation Principles

Xibra is a multi-user platform, not a shared personal account.

The following principles are mandatory:

- Every user has an independent Xibra identity.
- Every private resource must have a verified ownership relationship.
- The API must derive user identity from a verified authentication context.
- Client-provided user IDs must never establish ownership.
- Ownership must be checked on every read, create, update, and delete operation.
- Users must not access another user's private data by changing resource identifiers.
- Role permissions and resource ownership are separate authorization requirements.
- Platform ownership does not automatically grant unrestricted access to private user data.
- Shared resources require an explicit sharing and authorization model.
- User-specific integrations, credentials, AI context, and conversations must remain isolated.

The initial platform uses one shared PostgreSQL database with logical data separation enforced by the API.

Additional database-level protections may be introduced when the architecture requires them.

## 7. Invitation and Onboarding

Xibra uses an invitation-only onboarding model.

Only the platform Owner may create, view, and revoke invitations. Regular members must not have invitation management permissions.

### 7.1 Invitation Channels

The Owner can create and manage invitations through two interfaces:

- Telegram Bot.
- Private Dashboard.

Both interfaces use the same invitation system and API endpoints. They must not maintain separate invitation records or independent invitation logic.

### 7.2 Invitation Formats

An invitation may be distributed using:

- A human-readable invitation code.
- A shareable invitation link.
- Both formats for the same invitation.

Example invitation code:

`XIBRA-5JG1-DCRJ`

Example invitation link:

`https://app.example.com/invite/{invitation-token}`

The code and link must reference the same underlying invitation record.

### 7.3 Invitation Expiration

- Default expiration: 24 hours after creation.
- Minimum configurable expiration: 1 hour.
- Maximum configurable expiration: 72 hours.
- Expired invitations cannot be redeemed.
- The Owner may revoke an unused invitation before expiration.

### 7.4 Invitation Usage and Status

Each invitation may be successfully used only once.

Supported invitation statuses:

- `pending`: Created and available for redemption.
- `redeemed`: Successfully completed onboarding.
- `expired`: Expiration time has passed without successful onboarding.
- `revoked`: Revoked by the Owner before successful redemption.

Failed validation attempts and incomplete onboarding sessions must be recorded separately from the permanent invitation status.

A short-lived onboarding session must prevent multiple claims of the same invitation while registration is in progress.

### 7.5 Invitation Security

- Invitation creation and management must be authorized by xibra-api.
- Only the Owner may manage invitations.
- Invitation tokens and codes must be generated using cryptographically secure randomness.
- Raw invitation secrets must not be stored in plaintext.
- Invitation redemption must be rate-limited.
- Invitation links must use HTTPS.
- Invitation redemption must be atomic to prevent simultaneous reuse.
- Invitees must not be allowed to select their own role.
- The default role for invited users is `member`.
- Invitation activity must be auditable.

### 7.6 Onboarding Flow

1. The Owner creates an invitation through Telegram Bot or Dashboard.
2. The API generates an invitation code, link, or both.
3. The Owner shares the invitation with the intended recipient.
4. The recipient opens the link or submits the code through Telegram Bot.
5. The API validates the invitation and creates a short-lived onboarding session.
6. The recipient completes account registration through the secure Dashboard onboarding page.
7. The recipient chooses a username, provides optional email or phone number, and creates a password.
8. The API validates the onboarding session, creates the user account, assigns the `member` role, and links the verified Telegram identity when applicable.
9. The invitation is marked as `redeemed` after successful account creation.
10. The new user can access their private Dashboard and linked Telegram Bot capabilities.

Passwords must only be submitted through the secure HTTPS onboarding page. Passwords must never be collected or transmitted through Telegram messages.

Telegram identity must be obtained from a trusted Telegram update and validated through an authenticated bot-to-API service request. A Telegram username alone must never be treated as proof of identity.

## 8. Security Principles

- No unrestricted public registration.
- New users are onboarded through Owner-controlled invitations.
- Initial owner access must be provisioned through a controlled process.
- Protected endpoints require authentication.
- Authorization must be enforced by the API.
- Role-based permissions must be separated from resource ownership.
- User data must remain isolated.
- Google OAuth tokens and other credentials must be encrypted at rest.
- Secrets must never be committed to Git.
- AI integrations must follow the same access boundaries as other clients.
- Sensitive actions require explicit approval.
- Login and password recovery endpoints must be rate-limited.
- Invitation creation and management must be restricted to the Owner.
- Invitation redemption must be rate-limited and protected against replay and concurrent use.
- Telegram account linking must verify account ownership.
- Users must be able to revoke connected accounts and authentication tokens.
- Logs must not expose credentials, access tokens, passwords, or sensitive personal information.
- Important security-sensitive actions must be auditable.

## 9. Technology Stack

| Layer | Technology |
|---|---|
| Backend | PHP and Laravel |
| Database | PostgreSQL |
| API Authentication | Laravel Sanctum |
| Frontend | Nuxt and Vue 3 |
| Programming Language | TypeScript |
| Styling | Tailwind CSS |
| Telegram Bot | Node.js and TypeScript |
| AI Runtime | Hermes |
| AI Provider | Google Gemini and future providers |
| External Integrations | OAuth 2.0 and provider APIs |
| Web Server | Nginx |
| Production Environment | Ubuntu VPS |

Use compatible stable versions when each application is initialized. Verify the actual installed versions before implementation.

## 10. Design Direction

The visual identity should follow these principles:

- Dark-first interface.
- Near-black backgrounds.
- Neutral surface colors.
- Restrained blue and cyan accents.
- Clear typography.
- Consistent spacing.
- Subtle borders.
- Minimal animation.
- Functional and professional layouts.

The universal dashboard must support personalized content and module visibility without requiring separate deployments for individual users.

## 11. Development Strategy

Build the ecosystem in dependency order:

1. Local workspace and development environment.
2. Multi-user identity, authentication, invitations, and RBAC.
3. User context, data ownership, and isolation.
4. Telegram account linking and bot integration.
5. Universal dashboard foundation and personal settings.
6. Personal finance module.
7. Google integrations and AI capabilities.
8. Professional portfolio and public content.
9. Cross-application integration, security testing, and quality assurance.
10. Production deployment, monitoring, and operational maintenance.

Complete and verify each stage before expanding into the next.

## 12. Initial Non-Goals

- Unrestricted public registration.
- Direct database access from frontend applications.
- Direct database access from Telegram bot or Hermes.
- Separate primary databases for individual applications or users.
- Multiple Hermes agents.
- Automatic sharing of private data between users.
- Unrestricted platform-owner access to user credentials or private integrations.
- Premature implementation of every planned business domain.
- Autonomous AI modification of the project owner's local source files.
- A separate dashboard deployment for each user.