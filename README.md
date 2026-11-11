# Xibra Platform

Xibra Platform is a personal digital ecosystem designed to connect professional identity, productivity, work management, personal finance, knowledge management, and AI assistance.

The platform consists of independent applications connected through one central backend and one primary data source.

## Architecture

    xibra-portfolio ─┐
    xibra-dashboard ─┤
    xibra-bot ───────┼──> xibra-api ───> PostgreSQL
    xibra-core ──────┘

## Core Components

| Component | Responsibility | Technology |
|---|---|---|
| xibra-api | Central backend, authentication, business logic, and data access | Laravel, PHP, PostgreSQL |
| xibra-portfolio | Public professional portfolio | Nuxt, Vue 3, TypeScript |
| xibra-dashboard | Private platform control center | Nuxt, Vue 3, TypeScript |
| xibra-bot | Telegram interface | Node.js, TypeScript |
| xibra-core | Single Hermes AI agent | Hermes |

Only xibra-api may access PostgreSQL directly. All other applications must communicate through the API.

## Workspace Structure

    D:\Xibra-Platform\
    ├── README.md
    ├── IDEA.md
    ├── TODO.md
    ├── CLAUDE.md
    ├── docs\
    ├── xibra-api\
    ├── xibra-portfolio\
    ├── xibra-dashboard\
    ├── xibra-bot\
    └── hermes-config\

## Development Principles

1. Build the central API before dependent applications.
2. Keep application responsibilities separate.
3. Use the API as the only application-facing data boundary.
4. Maintain one primary source of truth.
5. Do not provide public administrator registration.
6. Keep secrets outside source control.
7. Implement only approved requirements.
8. Verify changes before considering them complete.
9. Use English throughout project documentation, source code comments, and commit messages.
10. The project owner writes and edits all source code manually. AI assistants provide guidance and code examples only.

## Getting Started

1. Read IDEA.md to understand the product vision and architecture.
2. Read TODO.md to identify the current development phase.
3. Read CLAUDE.md before using Claude as a coding assistant.
4. Review the relevant documentation under docs/.
5. Begin with Phase 0 in TODO.md.

## Git Workflow

Each application is maintained in its own Git repository.

Use the approved commit types and message format documented in CLAUDE.md.

Completed and verified changes should be committed and pushed to their configured remote repositories.

Never force-push, commit secrets, or push unverified changes.

The project owner executes and verifies all Git commands.

## Documentation Responsibilities

| File or Directory | Purpose |
|---|---|
| README.md | Workspace overview and entry point |
| IDEA.md | Product vision and architecture |
| TODO.md | Master development roadmap |
| CLAUDE.md | AI coding collaboration rules |
| docs/ | Detailed workspace-level technical documentation |
| Application README files | Application-specific setup and technical guides |
