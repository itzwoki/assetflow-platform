# 06 — API Contracts

## 1. Purpose

This document defines the API standards, conventions, endpoints, request/response structures, and communication rules for AssetFlow.

It establishes a consistent API interface across all backend services.

Each FastAPI service owns its Pydantic request and response models and generates its OpenAPI specification automatically.

This document defines the expected API behavior and does not replace service-level implementation models.

Related documents:

- `00_PROJECT_CHARTER.md`
- `01_PRD.md`
- `02_BUSINESS_WORKFLOWS.md`
- `03_DOMAIN_MODEL.md`
- `04_DATABASE_DESIGN.md`
- `05_SYSTEM_ARCHITECTURE.md`
- `07_FRONTEND_SPEC.md`
- `08_FINANCIAL_RULES.md`
- `09_ENGINEERING_STANDARDS.md`

---

## 2. API Architecture

AssetFlow exposes APIs through four backend services:

| Service | Base Path | Responsibility |
|---|---|---|
| Identity | `/api/identity/v1` | Authentication, users, organizations, memberships, roles and permissions |
| Configuration | `/api/configuration/v1` | Organization settings, finance products, product rules and configuration |
| Financing | `/api/financing/v1` | Customers, assets, lenders, calculations, quotations and proposals |
| Documents | `/api/documents/v1` | Documents, templates, uploads, generation and processing |

The frontend communicates with these services through the API Gateway.

Service-to-service communication uses internal APIs with appropriate authentication.

### 2.1 API versioning

All public APIs must be versioned.

Initial version:

`/api/{service}/v1/`

Breaking changes require a new API version.

Non-breaking additions may be introduced within the existing version when backward compatibility is maintained.

### 2.2 HTTP methods

| Method | Purpose |
|---|---|
| GET | Retrieve resources |
| POST | Create resources or execute operations |
| PUT | Replace a resource |
| PATCH | Partially update a resource |
| DELETE | Delete a resource or perform an explicitly defined deletion operation |

Use HTTP methods consistently and avoid using GET for operations that modify data.

---

## 3. General API Conventions

### 3.1 Resource naming

- Use plural nouns for collections.
- Use lowercase paths.
- Separate words using hyphens where necessary.
- Use resource IDs in paths.
- Keep endpoint names descriptive.

Examples:

```text
/customers
/customers/{customer_id}
/finance-products
/quotations
/quotations/{quotation_id}
/proposals/{proposal_id}
/documents/{document_id}
```

### 3.2 Identifiers

Use UUIDs for primary resource identifiers.

Example:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000"
}
```

IDs belonging to another service are treated as logical references and validated through that service's API when required.

### 3.3 Timestamps

Use ISO 8601 timestamps in UTC.

Example:

`2026-11-13T10:30:00Z`

### 3.4 Monetary values

Financial amounts must use decimal-compatible representations.

JSON monetary values should be represented as decimal strings to avoid binary floating-point precision problems.

Example:

```json
{
  "vehicle_price": "25000.00",
  "deposit": "5000.00",
  "interest_rate": "7.50"
}
```

The currency must be explicit where relevant.

Currency is an organization-level display setting, defaulting to `GBP`. Supported values are `GBP` and `USD`. Currency affects formatting only and must not alter calculation or rounding behaviour.

### 3.5 Null and optional fields

- Required fields must always be supplied.
- Optional fields may be omitted.
- Nullable fields may explicitly contain `null`.
- The distinction between omitted and null values must be defined where it affects business behavior.

### 3.6 Pagination

Collection endpoints must support pagination where result sizes may grow.

Initial convention:

```text
?page=1&page_size=20
```

Maximum page size:

`100`

Example response:

```json
{
  "items": [],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total_items": 0,
    "total_pages": 0
  }
}
```

### 3.7 Filtering and sorting

Use query parameters for filtering and sorting.

Examples:

```text
/customers?search=John
/quotations?status=draft
/proposals?created_from=2026-01-01
/customers?sort_by=created_at&sort_order=desc
```

Only explicitly supported filter and sort fields may be used.

---

## 4. Standard Response Structures

### 4.1 Successful responses

Single resource:

```json
{
  "data": {
    "id": "uuid",
    "name": "Example"
  }
}
```

Collection:

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total_items": 0,
    "total_pages": 0
  }
}
```

Operations that do not return a resource may return an appropriate success status without a response body.

### 4.2 Error responses

All services must follow a consistent error structure.

Example:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid fields.",
    "details": [
      {
        "field": "email",
        "message": "A valid email address is required."
      }
    ],
    "request_id": "uuid"
  }
}
```

### 4.3 Standard error codes

| Code | Meaning |
|---|---|
| `VALIDATION_ERROR` | Request validation failed |
| `AUTHENTICATION_REQUIRED` | Authentication is missing or invalid |
| `PERMISSION_DENIED` | User does not have required permission |
| `RESOURCE_NOT_FOUND` | Requested resource does not exist or is inaccessible |
| `RESOURCE_CONFLICT` | Operation conflicts with existing resource state |
| `DUPLICATE_RESOURCE` | Resource already exists |
| `INVALID_STATE_TRANSITION` | Requested state transition is not allowed |
| `EXTERNAL_SERVICE_ERROR` | Required external service operation failed |
| `RATE_LIMIT_EXCEEDED` | Request limit exceeded |
| `INTERNAL_SERVER_ERROR` | Unexpected server error |

Error responses must not expose database errors, stack traces, secrets, or internal implementation details.

---

## 5. HTTP Status Codes

| Status | Usage |
|---|---|
| `200 OK` | Successful retrieval or operation |
| `201 Created` | Resource successfully created |
| `202 Accepted` | Background operation accepted |
| `204 No Content` | Successful operation without response body |
| `400 Bad Request` | Invalid request or operation |
| `401 Unauthorized` | Authentication required or invalid |
| `403 Forbidden` | Authenticated user lacks permission |
| `404 Not Found` | Resource not found or inaccessible |
| `409 Conflict` | Resource or state conflict |
| `422 Unprocessable Entity` | Request validation failure |
| `429 Too Many Requests` | Rate limit exceeded |
| `500 Internal Server Error` | Unexpected internal failure |
| `502 Bad Gateway` | Invalid response from an upstream service |
| `503 Service Unavailable` | Service temporarily unavailable |
| `504 Gateway Timeout` | Upstream service timeout |

Services must use status codes consistently.

---

# 6. Identity Service APIs

Base path:

`/api/identity/v1`

## 6.1 Authentication

| Method | Endpoint | Description |
|---|---|---|
| POST | `/auth/register` | Register a user account. Organization provisioning is a separate, administratively controlled step. |
| POST | `/auth/login` | Authenticate using credentials |
| POST | `/auth/refresh` | Refresh access token |
| POST | `/auth/logout` | Revoke current refresh session |
| POST | `/auth/logout-all` | Revoke all user sessions |
| POST | `/auth/forgot-password` | Request password reset |
| POST | `/auth/reset-password` | Reset password using a valid token |
| POST | `/auth/verify-email` | Verify email address |
| POST | `/auth/resend-verification` | Resend verification email |
| POST | `/auth/otp/verify` | Verify OTP challenge |
| POST | `/auth/mfa/verify` | Complete MFA challenge |

### Login request

```json
{
  "email": "user@example.com",
  "password": "StrongPassword"
}
```

### Login response

```json
{
  "data": {
    "access_token": "token",
    "refresh_token": "token",
    "token_type": "Bearer",
    "expires_in": 900,
    "user": {
      "id": "uuid",
      "email": "user@example.com"
    },
    "organizations": [
      {
        "id": "uuid",
        "name": "Example Dealership",
        "membership_id": "uuid",
        "roles": [
          "owner"
        ]
      }
    ]
  }
}
```

If a user belongs to multiple organizations, the active organization must be selected before accessing organization-scoped business resources.

## 6.2 Organization management

| Method | Endpoint | Description |
|---|---|---|
| POST | `/organizations` | Create organization |
| GET | `/organizations` | List organizations available to current user |
| GET | `/organizations/{organization_id}` | Retrieve organization |
| PATCH | `/organizations/{organization_id}` | Update organization |
| POST | `/organizations/{organization_id}/switch` | Establish active organization context |
| GET | `/organizations/{organization_id}/members` | List organization members |
| POST | `/organizations/{organization_id}/members/invitations` | Invite member |
| GET | `/organizations/{organization_id}/invitations` | List invitations |
| POST | `/organizations/{organization_id}/invitations/{invitation_id}/accept` | Accept invitation |
| DELETE | `/organizations/{organization_id}/members/{member_id}` | Remove organization member |

## 6.3 Roles and permissions

| Method | Endpoint | Description |
|---|---|---|
| GET | `/organizations/{organization_id}/roles` | List roles |
| POST | `/organizations/{organization_id}/roles` | Create role |
| GET | `/organizations/{organization_id}/roles/{role_id}` | Retrieve role |
| PATCH | `/organizations/{organization_id}/roles/{role_id}` | Update role |
| DELETE | `/organizations/{organization_id}/roles/{role_id}` | Soft-delete or deactivate role |
| GET | `/permissions` | List available permissions |
| PUT | `/organizations/{organization_id}/members/{member_id}/roles` | Assign member roles |

Role-permission management:

| Method | Endpoint | Description |
|---|---|---|
| PUT | `/organizations/{organization_id}/roles/{role_id}/permissions` | Replace role permissions |

The backend must prevent removal of permissions or roles in ways that would leave an organization without an authorized administrator.

## 6.4 User account

| Method | Endpoint | Description |
|---|---|---|
| GET | `/users/me` | Retrieve current user |
| PATCH | `/users/me` | Update current user profile |
| PATCH | `/users/me/password` | Change password |
| GET | `/users/me/sessions` | List active sessions |
| DELETE | `/users/me/sessions/{session_id}` | Revoke session |
| GET | `/users/me/security` | Retrieve security settings |

---

# 7. Configuration Service APIs

Base path:

`/api/configuration/v1`

## 7.1 Organization configuration

| Method | Endpoint | Description |
|---|---|---|
| GET | `/organizations/{organization_id}/settings` | Retrieve organization settings |
| PATCH | `/organizations/{organization_id}/settings` | Update organization settings |
| GET | `/organizations/{organization_id}/profile` | Retrieve organization profile |
| PATCH | `/organizations/{organization_id}/profile` | Update organization profile |

## 7.2 Finance products

| Method | Endpoint | Description |
|---|---|---|
| GET | `/finance-products` | List finance products |
| GET | `/finance-products/{product_id}` | Retrieve finance product |
| GET | `/finance-products/{product_id}/versions` | List product versions |
| GET | `/finance-products/{product_id}/versions/{version_id}` | Retrieve product version |
| GET | `/organizations/{organization_id}/finance-products` | List enabled organization products |
| PUT | `/organizations/{organization_id}/finance-products` | Configure enabled finance products |

Finance product definitions and versions are owned by Configuration.

## 7.3 Product configuration

| Method | Endpoint | Description |
|---|---|---|
| GET | `/finance-products/{product_id}/rules` | Retrieve product rules |
| GET | `/finance-products/{product_id}/fields` | Retrieve required input fields |
| GET | `/finance-products/{product_id}/versions/{version_id}/rules` | Retrieve rules for specific version |

Product configuration responses must include version information.

## 7.4 Asset configuration

| Method | Endpoint | Description |
|---|---|---|
| GET | `/asset-categories` | List asset categories |
| GET | `/asset-categories/{category_id}` | Retrieve category |
| GET | `/asset-categories/{category_id}/fields` | Retrieve category fields |

## 7.5 Document template assignments

| Method | Endpoint | Description |
|---|---|---|
| GET | `/organizations/{organization_id}/document-template-assignments` | List template assignments |
| POST | `/organizations/{organization_id}/document-template-assignments` | Assign template |
| PATCH | `/organizations/{organization_id}/document-template-assignments/{assignment_id}` | Update assignment |

Configuration stores assignments and references. Actual templates and document versions are owned by Documents.

---

# 8. Financing Service APIs

Base path:

`/api/financing/v1`

## 8.1 Customers

| Method | Endpoint | Description |
|---|---|---|
| GET | `/customers` | List customers |
| POST | `/customers` | Create customer |
| GET | `/customers/{customer_id}` | Retrieve customer |
| PATCH | `/customers/{customer_id}` | Update customer |
| DELETE | `/customers/{customer_id}` | Soft-delete customer |

### Customer addresses

| Method | Endpoint | Description |
|---|---|---|
| GET | `/customers/{customer_id}/addresses` | List customer addresses |
| POST | `/customers/{customer_id}/addresses` | Add address |
| GET | `/customers/{customer_id}/addresses/{address_id}` | Retrieve address |
| PATCH | `/customers/{customer_id}/addresses/{address_id}` | Update address |
| DELETE | `/customers/{customer_id}/addresses/{address_id}` | Soft-delete address |

### Customer bank accounts

| Method | Endpoint | Description |
|---|---|---|
| GET | `/customers/{customer_id}/bank-accounts` | List bank accounts |
| POST | `/customers/{customer_id}/bank-accounts` | Add bank account |
| PATCH | `/customers/{customer_id}/bank-accounts/{account_id}` | Update bank account |
| DELETE | `/customers/{customer_id}/bank-accounts/{account_id}` | Soft-delete bank account |

Sensitive bank details must be protected in API responses and logs.

## 8.2 Assets and vehicles

| Method | Endpoint | Description |
|---|---|---|
| GET | `/assets` | List assets |
| POST | `/assets` | Create asset |
| GET | `/assets/{asset_id}` | Retrieve asset |
| PATCH | `/assets/{asset_id}` | Update asset |
| DELETE | `/assets/{asset_id}` | Soft-delete asset |

Vehicle-specific details may be managed through the asset resource or dedicated vehicle endpoints depending on the final domain model.

## 8.3 Lenders

| Method | Endpoint | Description |
|---|---|---|
| GET | `/lenders` | List lender records |
| POST | `/lenders` | Create lender record |
| GET | `/lenders/{lender_id}` | Retrieve lender |
| PATCH | `/lenders/{lender_id}` | Update lender |
| DELETE | `/lenders/{lender_id}` | Soft-delete lender |

### Organization lender relationships

| Method | Endpoint | Description |
|---|---|---|
| GET | `/organizations/{organization_id}/lenders` | List dealership lender relationships |
| POST | `/organizations/{organization_id}/lenders` | Add lender relationship |
| PATCH | `/organizations/{organization_id}/lenders/{relationship_id}` | Update relationship |
| DELETE | `/organizations/{organization_id}/lenders/{relationship_id}` | Deactivate relationship |

A lender record and a dealership's relationship with that lender are separate concepts.

## 8.4 Calculations

| Method | Endpoint | Description |
|---|---|---|
| POST | `/calculations` | Execute a finance calculation |
| GET | `/calculations` | List calculations |
| GET | `/calculations/{calculation_id}` | Retrieve calculation |
| GET | `/calculations/{calculation_id}/versions` | List calculation versions |
| GET | `/calculations/{calculation_id}/versions/{version_id}` | Retrieve calculation version |

### Calculation request example

```json
{
  "finance_product_id": "uuid",
  "finance_product_version_id": "uuid",
  "customer_id": "uuid",
  "asset_id": "uuid",
  "inputs": {
    "vehicle_price": "25000.00",
    "deposit": "5000.00",
    "term_months": 48,
    "interest_rate": "7.50"
  }
}
```

The `inputs` object is validated against the selected product's required fields and version-specific rules.

It is an API input structure, not permission to store all financial transaction data in one unstructured JSON database column. Persisted financial information must follow `04_DATABASE_DESIGN.md`.

The calculation response must identify:

- Calculation ID.
- Calculation version.
- Finance product and version.
- Input values.
- Output values.
- Currency.
- Calculation timestamp.

## 8.5 Quotations

| Method | Endpoint | Description |
|---|---|---|
| GET | `/quotations` | List quotations |
| POST | `/quotations` | Create quotation |
| GET | `/quotations/{quotation_id}` | Retrieve quotation |
| PATCH | `/quotations/{quotation_id}` | Update quotation metadata |
| POST | `/quotations/{quotation_id}/versions` | Create new quotation version |
| GET | `/quotations/{quotation_id}/versions` | List quotation versions |
| GET | `/quotations/{quotation_id}/versions/{version_id}` | Retrieve quotation version |
| POST | `/quotations/{quotation_id}/versions/{version_id}/issue` | Issue quotation |
| POST | `/quotations/{quotation_id}/versions/{version_id}/withdraw` | Withdraw quotation |

Quotation version content must be handled according to the quotation lifecycle.

Once a quotation version is included in an externally submitted proposal, its historical content must not be modified.

## 8.6 Proposals

| Method | Endpoint | Description |
|---|---|---|
| GET | `/proposals` | List proposals |
| POST | `/proposals` | Create proposal |
| GET | `/proposals/{proposal_id}` | Retrieve proposal |
| PATCH | `/proposals/{proposal_id}` | Update proposal metadata |
| POST | `/proposals/{proposal_id}/versions` | Create proposal version |
| GET | `/proposals/{proposal_id}/versions` | List proposal versions |
| GET | `/proposals/{proposal_id}/versions/{version_id}` | Retrieve proposal version |
| POST | `/proposals/{proposal_id}/versions/{version_id}/finalize` | Finalize proposal version |
| GET | `/proposals/{proposal_id}/versions/{version_id}/submission-history` | Retrieve external submission history |

### Proposal package generation

| Method | Endpoint | Description |
|---|---|---|
| POST | `/proposals/{proposal_id}/versions/{version_id}/packages` | Request proposal package generation |

This operation coordinates with the Documents Service.

The response may be asynchronous:

```json
{
  "data": {
    "job_id": "uuid",
    "status": "queued"
  }
}
```

The frontend can retrieve processing status from the Documents Service.

### External submissions

| Method | Endpoint | Description |
|---|---|---|
| POST | `/proposals/{proposal_id}/versions/{version_id}/external-submissions` | Record dealership-confirmed external submission |
| GET | `/proposals/{proposal_id}/versions/{version_id}/external-submissions` | List submissions |

### External submission request

```json
{
  "lender_id": "uuid",
  "proposal_package_reference_id": "uuid",
  "submitted_at": "2026-11-13T10:30:00Z",
  "external_reference": "LENDER-REF-123",
  "notes": "Submitted through lender portal"
}
```

The authenticated member is derived from the access token and must not be trusted from the request body.

The backend must validate:

- The lender is associated with the dealership.
- The proposal version belongs to the active organization.
- The package reference exists.
- The package belongs to the same organization.
- The package represents the exact proposal version being submitted.
- The package is in a valid state for submission.

The system records the dealership's confirmation of external submission. It does not independently verify that the lender received or processed the application.

Downloading or generating a package does not create an external submission.

---

# 9. Documents Service APIs

Base path:

`/api/documents/v1`

## 9.1 Document management

| Method | Endpoint | Description |
|---|---|---|
| GET | `/documents` | List organization documents |
| POST | `/documents` | Upload document |
| GET | `/documents/{document_id}` | Retrieve document metadata |
| PATCH | `/documents/{document_id}` | Update document metadata |
| DELETE | `/documents/{document_id}` | Soft-delete document |
| GET | `/documents/{document_id}/versions` | List document versions |
| GET | `/documents/{document_id}/versions/{version_id}` | Retrieve document version |
| GET | `/documents/{document_id}/download` | Obtain authorized download access |

Uploads should use multipart form data or a controlled upload workflow appropriate to the file size.

The service must validate file type, size, organization ownership, and access permissions.

## 9.2 Templates

| Method | Endpoint | Description |
|---|---|---|
| GET | `/templates` | List organization templates |
| POST | `/templates` | Create template |
| GET | `/templates/{template_id}` | Retrieve template |
| PATCH | `/templates/{template_id}` | Update template metadata |
| DELETE | `/templates/{template_id}` | Soft-delete template |
| GET | `/templates/{template_id}/versions` | List template versions |
| POST | `/templates/{template_id}/versions` | Create template version |
| GET | `/templates/{template_id}/versions/{version_id}` | Retrieve template version |

## 9.3 Document generation

| Method | Endpoint | Description |
|---|---|---|
| POST | `/generation-jobs` | Request document generation |
| GET | `/generation-jobs/{job_id}` | Retrieve processing status |
| POST | `/generation-jobs/{job_id}/retry` | Retry eligible failed job |

## 9.4 Proposal packages

| Method | Endpoint | Description |
|---|---|---|
| POST | `/proposal-packages` | Generate proposal package |
| GET | `/proposal-packages/{package_reference_id}` | Retrieve package metadata |
| GET | `/proposal-packages/{package_reference_id}/download` | Download package |

Proposal package generation must receive or resolve the exact proposal version and organization context.

The Documents Service registers the generated package reference.

The Financing Service validates the reference before recording an external submission.

---

# 10. Service-to-Service APIs

Internal APIs are used when one service requires data or validation from another service.

They must be authenticated independently from normal user requests.

Examples:

| Calling Service | Target Service | Purpose |
|---|---|---|
| Financing | Identity | Validate organization membership or service authorization where required |
| Financing | Configuration | Retrieve finance product definitions and rules |
| Financing | Documents | Validate proposal package reference |
| Documents | Financing | Retrieve authorized proposal data for package generation |
| Configuration | Documents | Validate document template references where required |

Internal endpoints must not automatically be exposed as public frontend endpoints.

Use service authentication and pass the relevant organization and resource context.

Avoid unnecessary synchronous dependencies.

---

# 11. Asynchronous Operations

Long-running operations return `202 Accepted` with a job identifier.

Example:

```json
{
  "data": {
    "job_id": "uuid",
    "status": "queued"
  }
}
```

Job status values:

- `queued`
- `processing`
- `completed`
- `failed`
- `cancelled`

The service owning the operation is responsible for its job status and lifecycle.

Background operations must support retry policies and idempotency where applicable.

---

# 12. Authentication Headers

Protected endpoints require:

```http
Authorization: Bearer <access_token>
```

Organization-scoped requests must establish a valid active organization context.

If an organization header is used for routing or context selection, the backend must verify that the authenticated user is a member of that organization.

Additional internal service requests must include the approved service authentication mechanism.

Never trust organization IDs, user IDs, or permission claims supplied only through untrusted request data.

---

# 13. Idempotency

Idempotency must be implemented for operations where retrying a request could create duplicate business records or external side effects.

Examples:

- Creating a quotation.
- Creating a proposal.
- Recording an external submission.
- Requesting document generation.
- Processing background jobs.

For applicable endpoints, support:

```http
Idempotency-Key: <unique-key>
```

The service must define the key's scope, retention period, and behavior when the same key is reused with a different request body.

---

# 14. API Documentation

Each FastAPI service must expose its own generated OpenAPI specification.

Expected local documentation:

```text
Identity:
  /docs
  /openapi.json

Configuration:
  /docs
  /openapi.json

Financing:
  /docs
  /openapi.json

Documents:
  /docs
  /openapi.json
```

The API Gateway may expose a consolidated API documentation entry point.

The OpenAPI specification must be generated from the service-owned Pydantic models and endpoint definitions.

The frontend may generate TypeScript types and clients from these specifications.

API documentation must remain synchronized with actual endpoint behavior.

---

# 15. API Security Requirements

All APIs must follow these rules:

1. Validate every incoming request.
2. Enforce authentication on protected endpoints.
3. Enforce permissions on protected operations.
4. Validate organization ownership for every tenant-scoped resource.
5. Never expose secrets in responses.
6. Never expose stack traces or database internals.
7. Apply rate limits to sensitive endpoints.
8. Validate file uploads.
9. Protect sensitive personal and financial information.
10. Use HTTPS in production.
11. Use timeouts for internal service calls.
12. Log security-relevant actions.
13. Apply idempotency to operations where duplicate execution is harmful.
14. Avoid mass assignment by explicitly defining accepted request fields.

---

# 16. Release 1 API Scope

Release 1 focuses on UK vehicle dealerships using Hire Purchase.

The initial API implementation includes:

- Authentication and organization management.
- Organization members, roles and permissions.
- Organization settings.
- Hire Purchase product configuration.
- Customer management.
- Customer addresses and bank accounts.
- Vehicle management.
- Lender records and dealership-lender relationships.
- HP calculations.
- Quotations and quotation versions.
- Proposals and proposal versions.
- Proposal package generation.
- External submission recording.
- Document management.
- Basic reporting and audit history.

Finance Lease and Operating Lease endpoints or product-specific behavior should be introduced in their respective releases.

The API architecture must permit their addition without redesigning the existing service boundaries.

---

# 17. API Acceptance Criteria

The API architecture is accepted when:

- Each service has a versioned API namespace.
- Each service owns its Pydantic request and response models.
- Each service generates an accurate OpenAPI specification.
- The frontend can generate TypeScript types from the specifications.
- All services use consistent response and error conventions.
- Organization-level authorization is enforced.
- Cross-service references are validated through APIs.
- Financial amounts use decimal-safe representations.
- Long-running operations use background jobs.
- Critical operations support idempotency.
- Proposal submissions reference the exact registered proposal package.
- APIs are documented and testable independently.
- No service accesses another service's database directly.


This is the API blueprint, not yet the final endpoint-by-endpoint implementation contract. Some details remain flexible because they depend on decisions not yet finalized:
1. ~~Whether registration creates an organization automatically or organization creation is a separate step.~~ **Resolved:** self-registration creates only a user account. Organizations are provisioned through a platform-administration area that creates the initial organization owner. Organization owners may then invite users by email with assigned roles, or create users directly.
2. Whether quotation creation and proposal creation use dedicated applicant/asset endpoints or nested request bodies.
3. Whether document generation is initiated by Financing or directly through Documents.
4. The exact permission names and endpoint-level authorization matrix.
Remaining items should be resolved in the relevant specifications before implementation, rather than inventing details now.
**End of Document**