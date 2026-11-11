# Xibra Platform — Product Vision and Architecture

## 1. Vision

Xibra Platform is a personal digital ecosystem designed to connect professional identity, productivity, work management, personal finance, knowledge management, and AI-assisted workflows.

The goal is to build one integrated platform where multiple applications share a central backend and a single source of truth.

## 2. Product Goals

- Maintain a public professional identity.
- Manage personal activities through a private dashboard.
- Access selected platform features through Telegram.
- Use an AI assistant to support daily activities and technical workflows.
- Keep data consistent across all applications.
- Expand the platform gradually through independent modules.

## 3. Core Applications

### 3.1 xibra-api

The central backend and the only application permitted to access the primary PostgreSQL database directly.

Responsibilities:
- Authentication and authorization.
- API versioning and response conventions.
- Request validation.
- Business logic.
- Database access and relationships.
- Integration endpoints.
- Automated testing.

### 3.2 xibra-portfolio

The public-facing professional website.

Initial pages:
- About
- Resume
- Works
- Blog
- Contact

The portfolio consumes public API endpoints. Content management belongs to the private dashboard.

### 3.3 xibra-dashboard

The private control center for managing Xibra Platform.

Planned modules:
- Overview
- Portfolio management
- Productivity
- Projects and tasks
- Schedule and workforce
- Personal finance
- Knowledge management
- Platform settings
- Activity and audit history

The dashboard communicates with xibra-api and never connects directly to PostgreSQL.

### 3.4 xibra-bot

A Telegram interface for accessing selected platform capabilities.

Planned workflows:
- Task management
- Schedule management
- Reminders
- Finance entries
- Activity summaries

The bot communicates exclusively with xibra-api.

### 3.5 xibra-core

The single planned Hermes AI agent.

Responsibilities:
- Understand natural language requests.
- Assist with productivity and planning.
- Retrieve and summarize information.
- Support coding and research workflows.
- Interact with approved API capabilities.
- Request approval before executing sensitive actions.

The platform will use exactly one Hermes agent named xibra-core.

## 4. System Architecture

    xibra-portfolio ─┐
    xibra-dashboard ─┤
    xibra-bot ──────┼──> xibra-api ───> PostgreSQL
    xibra-core ─────┘

Architectural rules:

1. xibra-api is the central backend.
2. PostgreSQL is the primary data source.
3. Only xibra-api accesses PostgreSQL directly.
4. Every client communicates through the API.
5. Business logic and authorization belong to the API.
6. Each application has a clearly defined responsibility.
7. No application may maintain a competing source of truth.

## 5. Planned Data Domains

These are planned business domains, not a statement that their database tables already exist.

### Identity and Access
- Users
- Authentication tokens
- Roles and permissions, if required

### Portfolio
- Profiles
- Experiences
- Education
- Skills
- Certifications
- Projects
- Blog posts
- Contact messages

### Productivity
- Projects
- Tasks
- Labels
- Reminders
- Routines
- Activity records

### Schedule and Workforce
- Workforce members
- Shift types
- Shift blocks
- Daily standby records

### Personal Finance
- Financial accounts
- Categories
- Income and expenses
- Transfers
- Budgets
- Debts and installments
- Savings goals
- Investments

### Knowledge Management
- Notes
- Tags
- References

### AI and Integrations
- Agent activity
- Approval requests
- Integrations
- Notifications
- Audit logs

Database tables and relationships must be designed incrementally before implementation.

## 6. Security Principles

- No public administrator registration.
- Initial administrator access must be provisioned through a controlled process.
- Protected endpoints require authentication.
- Authorization must be enforced by the API.
- Secrets must never be committed to Git.
- AI integrations must follow the same access boundaries as other clients.
- Sensitive actions require explicit approval.
- Logs must not expose credentials or access tokens.

## 7. Technology Stack

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
| Web Server | Nginx |
| Production Environment | Ubuntu VPS |

Use compatible stable versions when each application is initialized. Verify the actual installed versions before implementation.

## 8. Design Direction

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

## 9. Development Strategy

Build the ecosystem in dependency order:

1. Local workspace and development environment.
2. Central API foundation.
3. Public portfolio.
4. Private dashboard.
5. Telegram bot.
6. Hermes integration.
7. Cross-application integration and testing.
8. Production deployment and monitoring.

Complete and verify each stage before expanding into the next.

## 10. Initial Non-Goals

- Public administrator registration.
- Direct database access from frontend applications.
- Direct database access from Telegram bot or Hermes.
- Separate primary databases for individual applications.
- Multiple Hermes agents.
- Premature implementation of every planned business domain.
- Autonomous AI modification of the project owner's local source files.
