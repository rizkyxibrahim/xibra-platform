# Xibra Platform — API Conventions

## 1. Purpose

This document defines the API standards for all applications communicating with xibra-api.

All API endpoints must follow consistent naming, response, validation, and error-handling conventions.

## 2. API Versioning

All application endpoints must use a versioned prefix:

    /api/v1

Example:

    GET /api/v1/health
    GET /api/v1/public/profile
    GET /api/v1/admin/projects

Breaking API changes should be introduced through a new API version.

## 3. Endpoint Organization

Endpoints are organized into three primary groups.

### Public

Endpoints that do not require administrator authentication.

Example:

    GET /api/v1/public/profile
    GET /api/v1/public/projects
    GET /api/v1/public/blog

### Authentication

Endpoints responsible for administrator authentication.

Example:

    POST /api/v1/auth/login
    POST /api/v1/auth/logout
    GET /api/v1/auth/user

### Admin

Endpoints requiring authentication and appropriate authorization.

Example:

    GET /api/v1/admin/dashboard
    GET /api/v1/admin/projects
    POST /api/v1/admin/projects
    PUT /api/v1/admin/projects/{id}
    DELETE /api/v1/admin/projects/{id}

Public administrator registration must not be implemented.

## 4. HTTP Methods

| Method | Purpose |
|---|---|
| GET | Retrieve resources |
| POST | Create resources or execute actions |
| PUT | Replace or fully update a resource |
| PATCH | Partially update a resource |
| DELETE | Delete a resource |

Use resource-oriented and predictable endpoint names.

## 5. Standard JSON Response

Successful responses:

    {
        "success": true,
        "message": "Projects retrieved successfully.",
        "code": "PROJECTS_RETRIEVED",
        "data": [],
        "meta": null
    }

Error responses:

    {
        "success": false,
        "message": "The request is invalid.",
        "code": "VALIDATION_ERROR",
        "data": null,
        "meta": {
            "errors": {}
        }
    }

Every API response should use these top-level fields:

| Field | Purpose |
|---|---|
| success | Indicates whether the request succeeded |
| message | Human-readable result |
| code | Stable application-level response code |
| data | Response payload |
| meta | Optional metadata |

The actual HTTP status code must also accurately represent the result.

## 6. HTTP Status Codes

| Status | Usage |
|---|---|
| 200 | Successful request |
| 201 | Resource created |
| 204 | Successful request without response content, only when compatible with the response contract |
| 400 | Invalid request |
| 401 | Authentication required |
| 403 | Access forbidden |
| 404 | Resource not found |
| 409 | Resource conflict |
| 422 | Validation failure |
| 429 | Too many requests |
| 500 | Unexpected server error |

## 7. Validation

- Validate all incoming data.
- Reject invalid data with HTTP 422.
- Return field-level validation errors in meta.errors.
- Never trust client-side validation as a security boundary.
- Apply authorization before performing protected operations.

## 8. Pagination

Collection endpoints should use consistent pagination.

Example request:

    GET /api/v1/admin/projects?page=1&per_page=15

Example metadata:

    {
        "current_page": 1,
        "per_page": 15,
        "total": 45,
        "last_page": 3
    }

Pagination metadata belongs inside the meta field.

## 9. Authentication

Protected endpoints use Laravel Sanctum authentication.

The API must:
- Validate authentication tokens.
- Enforce authorization policies.
- Support logout and token revocation.
- Protect sensitive operations.
- Avoid exposing authentication secrets in responses.

## 10. Error Handling

- Use stable response codes.
- Return safe and useful error messages.
- Do not expose stack traces in production.
- Log internal exceptions securely.
- Never include passwords, tokens, or secrets in logs.
- Keep error responses consistent across endpoint groups.

## 11. API Documentation and Testing

Every implemented endpoint should have:
- A clear purpose.
- HTTP method and path.
- Authentication requirements.
- Request validation rules.
- Success response example.
- Error response examples.
- Automated tests for expected behavior.

## 12. Compatibility

All API clients must follow the documented contract.

Changes to response structures, authentication, or endpoint behavior must be reviewed for their impact on portfolio, dashboard, bot, and Hermes integrations.

## 13. Implementation Status

This document defines the intended API contract.

It does not imply that all endpoints or conventions have already been implemented.
