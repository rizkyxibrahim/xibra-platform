# Xibra Platform — Development Roadmap

## Status Legend

- [ ] Not started
- [~] In progress
- [x] Implemented and verified
- [!] Blocked or decision required

A task may only be marked as completed after its implementation and verification have been reviewed.

A documented requirement is not considered implemented until the corresponding code, tests, and verification are complete.

---

## Phase 0 — Workspace and Development Environment

### Workspace and Documentation

- [x] Create the Xibra Platform workspace.
- [x] Create the application directory structure.
- [x] Create the root README.md.
- [x] Create the product vision and architecture document.
- [x] Create the master development roadmap.
- [x] Create the AI coding collaboration guide.
- [x] Create the documentation index.
- [x] Create the architecture overview.
- [x] Create the API conventions document.
- [x] Create the database documentation.
- [x] Record the initial architecture decision.
- [ ] Audit and align all workspace documentation with the multi-user architecture.
- [x] Review the invitation-only onboarding specification.
- [x] Finalize the initial multi-user architecture documentation.
- [x] Finalize the Core Multi-User ERD.

### Development Environment

- [x] Verify Git version.
- [x] Verify PHP version.
- [x] Verify Composer version.
- [x] Verify Node.js and npm versions.
- [x] Verify PostgreSQL version.
- [x] Verify VS Code availability.
- [x] Review workspace directory structure.
- [x] Verify local PostgreSQL connectivity.

### Git and Repository Management

- [x] Establish the root workspace Git repository.
- [x] Create the root GitHub remote repository.
- [x] Initialize the xibra-api Git repository.
- [x] Configure the xibra-api remote repository.
- [x] Establish the conventional commit message format.
- [x] Establish the manual Git review and approval workflow.
- [ ] Review the Git status and synchronization state of all repositories.
- [ ] Initialize independent repositories for remaining applications.
- [ ] Configure remote repositories for remaining applications.
- [ ] Verify root repository ignore rules and application repository boundaries.

---

## Phase 1 — Central API Foundation

### Laravel Project Foundation

- [x] Initialize the Laravel 13 application.
- [x] Configure the local environment file.
- [x] Configure PostgreSQL connection.
- [x] Establish the initial API route structure.
- [x] Install Laravel Sanctum.
- [x] Publish and apply the Sanctum migration.
- [x] Apply the initial Laravel migrations.
- [x] Verify migration status.
- [x] Establish the API project README.
- [x] Establish API-specific Claude instructions.
- [x] Review the current API implementation and Git state.
- [x] Establish API versioning conventions.
- [x] Finalize the standard JSON response format.
- [x] Review exception handling.
- [x] Review CORS configuration.
- [x] Verify the health endpoint.
- [x] Verify the API foundation with automated tests.

### Core Multi-User Architecture

- [x] Finalize the Core Multi-User ERD.
- [x] Review existing Laravel migrations and database tables.
- [x] Define the user identity and account model.
- [x] Define username, email, and phone login identifiers.
- [x] Define the initial Owner provisioning process.
- [x] Define the initial Member role.
- [x] Design roles and permissions.
- [x] Design user ownership relationships.
- [x] Define data isolation requirements.
- [x] Design the account lifecycle.
- [x] Define account activation and deactivation behavior.
- [x] Review password recovery requirements.
- [x] Record approved architecture decisions.

### Database Implementation

- [x] Design the initial multi-user database schema.
- [x] Create new migrations without modifying applied migrations unnecessarily.
- [x] Extend the users table with approved identity fields.
- [x] Create roles and permissions tables.
- [x] Create role assignment relationships.
- [x] Create initial reference seeders.
- [x] Implement controlled Owner provisioning.
- [x] Create the initial Member role seeder.
- [x] Review foreign keys and indexes.
- [x] Verify clean migration and seeding.
- [ ] Verify existing database data is protected.

### Authentication and Authorization

- [x] Implement username, email, and phone login.
- [x] Implement logout and token revocation.
- [x] Implement authenticated user retrieval.
- [x] Apply HasApiTokens to the User model.
- [x] Implement authentication middleware.
- [x] Implement role-based authorization.
- [x] Implement permission checks.
- [ ] Implement account deactivation and reactivation (ADR 0002).
- [ ] Implement Owner-only user administration listing.
- [ ] Implement resource ownership policies.
- [ ] Prevent unauthorized ownership reassignment.
- [x] Implement login rate limiting.
- [ ] Implement email verification for optional email addresses.
- [ ] Implement secure password recovery.
- [ ] Implement audit and onboarding-attempt read endpoints (ADR 0004).
- [ ] Design and implement Google OAuth connection foundation.
- [ ] Test authentication and authorization boundaries.

### API Quality

- [x] Implement consistent API response handling.
- [x] Implement request validation.
- [x] Implement API Resources where appropriate.
- [x] Add feature tests.
- [x] Add authentication tests.
- [x] Add authorization tests.
- [ ] Add multi-user ownership isolation tests.
- [x] Configure Laravel Pint.
- [ ] Document API endpoint contracts.
- [ ] Verify the API foundation.

---

## Phase 2 — Invitation and Account Onboarding

### Invitation Management

- [x] Finalize invitation database design.
- [x] Implement invitation migrations.
- [x] Implement invitation code generation.
- [x] Implement invitation link token generation.
- [x] Store invitation secrets as cryptographic hashes.
- [x] Implement configurable invitation expiration.
- [x] Enforce the 1-hour minimum and 72-hour maximum.
- [x] Set the default invitation expiration to 24 hours.
- [x] Implement invitation status management.
- [x] Implement invitation revocation.
- [x] Implement Owner-only invitation authorization.
- [x] Implement invitation audit records.

### Secure Redemption

- [x] Implement invitation validation.
- [x] Implement invitation expiration checks.
- [x] Implement one-time invitation redemption.
- [x] Prevent replay and concurrent redemption.
- [x] Implement short-lived onboarding sessions.
- [x] Track failed validation attempts separately.
- [x] Test expired, revoked, and redeemed invitations.
- [x] Test concurrent redemption behavior.

### Account Onboarding

- [x] Design the onboarding API contract.
- [x] Implement secure account creation.
- [x] Assign the default Member role.
- [x] Implement onboarding session validation.
- [x] Implement account activation.
- [x] Prevent duplicate account creation.
- [x] Verify successful invitation redemption.
- [x] Test the complete onboarding lifecycle.

### Telegram Identity Linking

- [x] Define the bot-to-API authentication mechanism.
- [x] Implement authenticated bot service requests.
- [x] Validate trusted Telegram numeric user IDs.
- [x] Design Telegram account linking.
- [x] Prevent duplicate Telegram identity linking.
- [x] Implement Telegram account linking and unlinking.
- [x] Require existing account verification before linking.
- [x] Test Telegram identity ownership and isolation.
- [x] Document the Telegram service authentication and linking contract in `docs/api/conventions.md`.
- [x] Add the Telegram identity linking requests to the Postman collection.

---

## Phase 3 — Public Portfolio

### Foundation

- [ ] Initialize Nuxt application.
- [ ] Configure Vue 3 and TypeScript.
- [ ] Configure Tailwind CSS.
- [ ] Establish the shared design system.
- [ ] Configure API communication.
- [ ] Establish application environment configuration.

### Public Pages

- [ ] About
- [ ] Resume
- [ ] Works
- [ ] Blog
- [ ] Blog detail
- [ ] Contact

### API Integration

- [ ] Implement public profile endpoints.
- [ ] Implement public resume endpoints.
- [ ] Implement public project endpoints.
- [ ] Implement published blog endpoints.
- [ ] Implement contact submission endpoint.
- [ ] Enforce published-content visibility.
- [ ] Protect private user information.

### Quality

- [ ] Implement responsive layouts.
- [ ] Implement SEO metadata.
- [ ] Add error and loading states.
- [ ] Verify API integration.
- [ ] Verify production build.

---

## Phase 4 — Universal Private Dashboard

### Foundation

- [ ] Initialize Nuxt application.
- [ ] Configure Vue 3 and TypeScript.
- [ ] Configure Tailwind CSS.
- [ ] Establish dashboard design system.
- [ ] Establish universal dashboard layout.
- [ ] Implement authentication flow.
- [ ] Implement protected routes.
- [ ] Implement authenticated user context.
- [ ] Implement permission-aware navigation.
- [ ] Implement secure session handling.

### Core Modules

- [ ] Overview dashboard.
- [ ] Profile management.
- [ ] Resume management.
- [ ] Experience management.
- [ ] Education management.
- [ ] Skills management.
- [ ] Certifications management.
- [ ] Project management.
- [ ] Blog management.
- [ ] Contact inbox.
- [ ] Account settings.
- [ ] Platform settings.

### Invitation Management

- [ ] Implement Owner-only invitation creation.
- [ ] Implement invitation code display.
- [ ] Implement invitation link display.
- [ ] Implement invitation listing.
- [ ] Implement invitation revocation.
- [ ] Display invitation status and expiration.
- [ ] Integrate with the central API invitation system.

### Future Modules

- [ ] Productivity management.
- [ ] Schedule and workforce management.
- [ ] Personal finance.
- [ ] Knowledge management.
- [ ] Activity and audit history.
- [ ] Integration management.
- [ ] AI assistant interface.

### Quality

- [ ] Verify API integration.
- [ ] Verify multi-user data isolation.
- [ ] Verify authorization boundaries.
- [ ] Verify responsive layouts.
- [ ] Verify production build.

---

## Phase 5 — Telegram Bot

### Foundation

- [ ] Initialize Node.js and TypeScript project.
- [ ] Configure environment variables.
- [ ] Configure Telegram Bot API.
- [ ] Establish a reusable API client.
- [ ] Implement authenticated bot-to-API communication.
- [ ] Implement secure user identification.
- [ ] Implement account onboarding integration.
- [ ] Implement Telegram account linking.
- [ ] Implement structured logging.
- [ ] Implement error handling.

### Initial Features

- [ ] Start and help commands.
- [ ] Account information.
- [ ] Invitation management for Owner.
- [ ] Task management.
- [ ] Schedule information.
- [ ] Reminder management.
- [ ] Daily summaries.
- [ ] Finance entry workflows.

### Quality

- [ ] Test API communication.
- [ ] Test Telegram interactions.
- [ ] Test multi-user identity isolation.
- [ ] Test unauthorized command handling.
- [ ] Configure production process management.
- [ ] Document bot operations.

---

## Phase 6 — Productivity and Workforce Management

### Productivity

- [ ] Finalize task management requirements.
- [ ] Design task database schema.
- [ ] Implement task management API.
- [ ] Implement task ownership and authorization.
- [ ] Implement task status and priority.
- [ ] Implement categories and labels.
- [ ] Implement task deadlines.
- [ ] Implement reminders.
- [ ] Implement recurring tasks where approved.

### Scheduling and Workforce

- [ ] Finalize scheduling requirements.
- [ ] Design schedule database schema.
- [ ] Implement schedule management API.
- [ ] Implement shift management.
- [ ] Implement shift rotation rules.
- [ ] Implement availability and standby management.
- [ ] Implement schedule conflict validation.
- [ ] Implement schedule notifications.
- [ ] Test scheduling rules and ownership boundaries.

### Integration

- [ ] Integrate productivity modules with Dashboard.
- [ ] Integrate productivity workflows with Telegram.
- [ ] Verify cross-application consistency.

---

## Phase 7 — Personal Finance

### Requirements and Data Model

- [ ] Finalize personal finance requirements.
- [ ] Define supported account types.
- [ ] Define income and expense categories.
- [ ] Define transaction lifecycle.
- [ ] Design finance database schema.
- [ ] Define financial data ownership and isolation.
- [ ] Define balance calculation rules.

### API and Dashboard

- [ ] Implement finance accounts.
- [ ] Implement income transactions.
- [ ] Implement expense transactions.
- [ ] Implement transfers.
- [ ] Implement budgets.
- [ ] Implement financial summaries.
- [ ] Implement transaction history.
- [ ] Implement validation and audit records.

### Integrations and Quality

- [ ] Integrate finance workflows with Telegram.
- [ ] Add finance feature tests.
- [ ] Verify multi-user finance isolation.
- [ ] Verify balance calculations.
- [ ] Verify transaction consistency.

---

## Phase 8 — Google Integrations and Hermes AI

### Google Integrations

- [ ] Review Google OAuth requirements.
- [ ] Define required Google API scopes.
- [ ] Configure the Google Cloud project.
- [ ] Implement per-user OAuth authorization.
- [ ] Implement encrypted token storage.
- [ ] Implement token refresh handling.
- [ ] Implement connection revocation.
- [ ] Integrate Gmail capabilities.
- [ ] Integrate Google Chat capabilities.
- [ ] Test integration ownership and isolation.

### Hermes Runtime

- [ ] Review Hermes runtime requirements.
- [ ] Configure exactly one agent named xibra-core.
- [ ] Configure the agent workspace.
- [ ] Configure approved API access.
- [ ] Configure Telegram integration.
- [ ] Define user context propagation.
- [ ] Ensure AI requests use the correct user context.
- [ ] Prevent direct database access.

### Assistant Capabilities

- [ ] Natural language understanding.
- [ ] Productivity assistance.
- [ ] Schedule assistance.
- [ ] Research and summarization.
- [ ] Coding assistance.
- [ ] Personal finance assistance.
- [ ] Knowledge management assistance.
- [ ] Controlled API actions.
- [ ] Approval workflow for sensitive actions.

### Safety

- [ ] Define permitted actions.
- [ ] Define actions requiring approval.
- [ ] Define prohibited actions.
- [ ] Prevent unauthorized data access.
- [ ] Isolate conversations and memory per user.
- [ ] Test failure and recovery behavior.
- [ ] Test sensitive action approval.

---

## Phase 9 — Integration and Quality Assurance

### Cross-Application Integration

- [ ] Verify portfolio integration.
- [ ] Verify dashboard integration.
- [ ] Verify Telegram bot integration.
- [ ] Verify Hermes integration.
- [ ] Verify authentication across applications.
- [ ] Verify invitation onboarding across interfaces.
- [ ] Verify authorization boundaries.
- [ ] Verify user data isolation.
- [ ] Verify API response consistency.

### Security and Reliability

- [ ] Review authentication and authorization.
- [ ] Review ownership enforcement.
- [ ] Review service-to-service authentication.
- [ ] Review secrets and credential handling.
- [ ] Review input validation.
- [ ] Review rate limiting.
- [ ] Review audit logging.
- [ ] Run automated tests.
- [ ] Review application logs.
- [ ] Test backup and recovery procedures.
- [ ] Document operational procedures.

---

## Phase 10 — Production Deployment

### Infrastructure

- [ ] Prepare Ubuntu VPS.
- [ ] Configure SSH security.
- [ ] Configure firewall.
- [ ] Configure Nginx.
- [ ] Configure PHP-FPM.
- [ ] Configure Node.js runtime.
- [ ] Configure PostgreSQL.
- [ ] Configure application directories.
- [ ] Configure environment secrets.
- [ ] Configure service users and permissions.

### Deployment

- [ ] Deploy xibra-api.
- [ ] Deploy xibra-portfolio.
- [ ] Deploy xibra-dashboard.
- [ ] Deploy xibra-bot.
- [ ] Deploy xibra-core.
- [ ] Configure domain routing.
- [ ] Configure SSL certificates.
- [ ] Configure process supervision.
- [ ] Verify production environment configuration.
- [ ] Verify production health endpoints.

### Operations

- [ ] Configure database backups.
- [ ] Verify backup restoration.
- [ ] Configure log rotation.
- [ ] Configure service monitoring.
- [ ] Configure alerting.
- [ ] Document deployment procedures.
- [ ] Document incident recovery procedures.
- [ ] Verify application recovery after restart.

---

## Current Development Focus

**Phase 1–2 completion — account lifecycle, password recovery, and audit access**

Current priorities:

1. Implement Owner-only account deactivation and reactivation, email verification, and the password reset flow (ADR 0002).
2. Implement the Owner-wide and self-scoped audit endpoints and the retention prune command (ADR 0004).
3. Implement the first owned resource with ownership policies and isolation tests (ADR 0003).
4. Document the remaining implemented API endpoint contracts.

Do not begin dependent application feature development until the central identity, authentication, authorization, ownership, and onboarding foundations are sufficiently defined and verified.

The API project has already been initialized and its initial Laravel migrations have been applied. Do not repeat project initialization or destructive database setup without an explicit technical reason and owner approval.