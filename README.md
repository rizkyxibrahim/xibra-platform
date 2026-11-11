# Xibra Platform

Xibra Platform is a multi-user personal digital ecosystem that connects professional identity, productivity, work management, personal finance, knowledge management, and AI assistance through one unified platform.

It provides a shared application architecture where each user has an independent identity, private data, integrations, permissions, and AI context.

The platform consists of independent applications connected through one central backend and one primary data source.

## Architecture

```text
xibra-portfolio ──┐
xibra-dashboard ──┤
xibra-bot ────────┼──> xibra-api ───> PostgreSQL
xibra-core ───────┘
```

All connected applications communicate through `xibra-api`.

`xibra-core` is the single Hermes AI agent. It interacts with the platform through authorized API operations and does not access PostgreSQL directly.

Only `xibra-api` may access PostgreSQL.

## Core Components

| Component | Responsibility | Technology |
|---|---|---|
| xibra-api | Central backend, identity, authentication, authorization, business logic, and data access | Laravel, PHP, PostgreSQL |
| xibra-portfolio | Public professional portfolio | Nuxt, Vue 3, TypeScript |
| xibra-dashboard | Universal private platform control center | Nuxt, Vue 3, TypeScript |
| xibra-bot | Telegram interface | Node.js, TypeScript |
| xibra-core | Single Hermes AI agent | Hermes |

## Multi-User Architecture

Xibra Platform is designed to serve multiple independent users through one shared deployment.

- Each user has an independent account and identity.
- Private resources are isolated by ownership.
- Authentication and authorization are enforced by the API.
- User roles and resource ownership are separate security requirements.
- Integrations and AI context are isolated per user.
- No private data is shared automatically between users.
- Platform ownership does not imply unrestricted access to every user's private data.

The Dashboard is a universal application. All users access the same application, while their available features, permissions, and data depend on their authenticated accounts.

## Access and Onboarding

Xibra Platform uses an invitation-only onboarding model.

- Public self-registration is not available.
- Only the platform Owner can create, view, and revoke invitations.
- Invitations can be managed through Telegram Bot or Dashboard.
- Both interfaces use the same central API invitation system.
- New users receive the default `member` role.
- Account creation and identity linking are controlled by the API.

## Workspace Structure

```text
D:\Xibra-Platform\
├── README.md
├── IDEA.md
├── TODO.md
├── CLAUDE.md
├── docs\
│   ├── architecture\
│   ├── api\
│   ├── database\
│   └── decisions\
├── xibra-api\
├── xibra-portfolio\
├── xibra-dashboard\
├── xibra-bot\
└── hermes-config\
```

Each application directory is maintained as an independent Git repository.

## Development Principles

1. Build the central API before dependent applications.
2. Keep application responsibilities separate.
3. Use the API as the only application-facing data boundary.
4. Maintain one primary source of truth.
5. Enforce authentication, authorization, and resource ownership.
6. Isolate user data, integrations, and AI context.
7. Do not provide unrestricted public registration.
8. Keep secrets outside source control.
9. Implement only approved requirements.
10. Verify changes before considering them complete.
11. Use English throughout project documentation, source code comments, and commit messages.
12. The project owner writes and edits all source code manually. AI assistants provide guidance and code examples only.

## Getting Started

1. Read `IDEA.md` to understand the product vision and architecture.
2. Read `TODO.md` to identify the current development phase.
3. Read `CLAUDE.md` before using Claude as a coding assistant.
4. Review the relevant documentation under `docs/`.
5. Review the application-specific README and instructions before working inside an application.
6. Begin with Phase 0 in `TODO.md`.

## Git Workflow

Each application is maintained in its own Git repository.

Use the approved commit types and message format documented in `CLAUDE.md`.

Completed and verified changes may be committed and pushed to their configured remote repositories after explicit owner approval.

Never force-push, commit secrets, or push unverified changes.

The project owner executes and verifies all Git commands.

## Documentation Responsibilities

| File or Directory | Purpose |
|---|---|
| `README.md` | Workspace overview and entry point |
| `IDEA.md` | Product vision and architecture |
| `TODO.md` | Master development roadmap |
| `CLAUDE.md` | AI coding collaboration rules |
| `docs/` | Detailed workspace-level technical documentation |
| Application README files | Application-specific setup and technical guides |