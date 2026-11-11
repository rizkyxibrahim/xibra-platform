# Xibra Platform — System Architecture

## 1. Overview

Xibra Platform uses a centralized backend architecture.

Independent applications communicate through xibra-api, while PostgreSQL remains the primary data source.

The architecture separates presentation, integration, business logic, and data persistence.

## 2. High-Level Architecture

    +-------------------+       +-------------------+
    |  xibra-portfolio  |       |  xibra-dashboard  |
    |   Public Website  |       |  Private Control  |
    +---------+---------+       +---------+---------+
              |                           |
              | HTTPS / REST API          |
              |                           |
    +---------+---------------------------+---------+
    |                 xibra-api                     |
    |                                               |
    | Authentication | Authorization | Business    |
    | Validation     | API Responses  | Logic       |
    +-------------------------+---------------------+
                              |
                              | Database Connection
                              |
                       +------+------+
                       | PostgreSQL |
                       +-------------+

    +-------------------+       +-------------------+
    |     xibra-bot     |       |    xibra-core     |
    | Telegram Interface|       |   Hermes Agent    |
    +---------+---------+       +---------+---------+
              |                           |
              +-------------+-------------+
                            |
                       xibra-api

## 3. Component Responsibilities

### xibra-api

The central application responsible for:
- Authentication.
- Authorization.
- Request validation.
- Business logic.
- Database operations.
- API response formatting.
- Integration boundaries.

### xibra-portfolio

The public website responsible for presenting professional information and published content.

It must use public API endpoints and must not access PostgreSQL directly.

### xibra-dashboard

The private interface responsible for managing platform data and administrative operations.

It must use authenticated API endpoints.

### xibra-bot

The Telegram interface responsible for receiving user commands and presenting platform information.

It must communicate with xibra-api rather than accessing application data directly.

### xibra-core

The single Hermes AI agent responsible for interpreting requests and coordinating approved assistant workflows.

It must respect API authorization and approval requirements.

## 4. Communication Rules

1. Applications communicate through HTTPS in production.
2. REST endpoints use the versioned API prefix.
3. Protected operations require authentication.
4. Authorization is enforced by the backend.
5. Request validation occurs at the API boundary.
6. API responses follow a shared JSON contract.
7. Clients must not bypass backend business rules.

## 5. Database Ownership

PostgreSQL is owned and accessed by xibra-api.

Other applications must never:
- Connect directly to PostgreSQL.
- Execute SQL against the primary database.
- Maintain independent copies of authoritative business records.
- Bypass API authorization.

The API is the single application-facing data boundary.

## 6. Authentication and Security Boundaries

### Public Access

Public portfolio content may be exposed through explicitly designated public endpoints.

### Private Access

Dashboard and other protected workflows require authenticated access.

### AI Access

Hermes must use approved integration capabilities.

AI access must not automatically grant administrator privileges.

Sensitive operations require explicit user approval.

## 7. Deployment Direction

Planned production routing:

| Hostname | Application | Runtime |
|---|---|---|
| rizkyxibrahim.my.id | xibra-portfolio | Nuxt / Node.js |
| www.rizkyxibrahim.my.id | xibra-portfolio | Nginx redirect or canonical routing |
| app.rizkyxibrahim.my.id | xibra-dashboard | Nuxt / Node.js |
| api.rizkyxibrahim.my.id | xibra-api | Nginx / PHP-FPM |

Telegram bot and Hermes are planned as background services without requiring public web endpoints.

PostgreSQL must remain accessible only through the server's local application connection.

## 8. Architecture Evolution

Changes to the architecture must be deliberate.

Before introducing a new application, database, integration, or runtime:
1. Identify the problem it solves.
2. Evaluate whether an existing component can handle it.
3. Define ownership and security boundaries.
4. Document significant decisions.
5. Update related architecture documentation.

## 9. Current Implementation Status

This document describes the intended architecture.

It does not imply that all applications, endpoints, integrations, or deployment configurations have already been implemented.
