# LeaseFlow — Project Charter

**Document Version:** 1.1  
**Status:** Approved project direction  
**Market:** United Kingdom  
**Initial Business Type:** Vehicle Dealerships  
**Initial Finance Product:** Hire Purchase (HP)  
**Development Approach:** Local-first, production-quality engineering

---

## 1. Project Overview

LeaseFlow is a dealership-focused finance management platform designed initially for the United Kingdom vehicle finance market.

The platform enables vehicle dealerships to manage customers and vehicles, configure finance products, calculate finance options, generate quotations and proposals, manage supporting documents, and prepare complete finance application packages for submission to external lenders.

LeaseFlow is responsible for the dealership's finance preparation process. The lender submission is the final business boundary of the platform.

Lenders continue to operate through their own systems and portals. LeaseFlow does not attempt to replace those systems.

The project will initially support vehicle dealerships and Hire Purchase (HP). Its architecture must allow additional dealership types, asset categories, and finance products to be introduced through future development without requiring a complete redesign.

## 2. Project Vision

Build a realistic, extensible, secure, and production-quality finance management platform that demonstrates the engineering practices and system design expected of an experienced software engineer.

The project is intended primarily as a professional portfolio project and a practical learning environment.

It is not initially being developed for commercial sale or immediate production deployment.

## 3. Project Objectives

The project aims to:

1. Build a complete dealership-side finance workflow.
2. Implement Hire Purchase as the first fully working finance product.
3. Support customer and vehicle management.
4. Provide configurable finance product definitions and calculation rules.
5. Generate quotations, proposals, and supporting documents.
6. Prepare finance application packages for submission to external lenders.
7. Provide organization-level configuration, branding, users, roles, and permissions.
8. Maintain strict organization-level data and document isolation.
9. Use a service-oriented architecture with independently owned databases.
10. Demonstrate testing, security, observability, documentation, and maintainable engineering practices.
11. Keep development entirely local until the final deployment phase.

## 4. Scope and Boundaries

### 4.1 Included in the initial release

- Vehicle dealership organizations.
- Dealership users, authentication, roles, and permissions.
- Organization configuration and branding.
- Customer management.
- Vehicle and asset management for vehicles.
- External lender reference records and dealership-lender relationships.
- Hire Purchase finance product.
- Finance calculations and configurable calculation rules.
- Finance quotations.
- Finance proposals and application preparation.
- Dealership-owned document management.
- Document templates and placeholder configuration.
- PDF/document generation and supporting document attachment.
- Preparation and submission of finance application packages to external lenders.
- Dealership dashboards, reports, and audit history.

### 4.2 Explicitly outside the initial scope

LeaseFlow will not implement:

- Lender employee accounts or lender portals.
- Lender-side organization management.
- Credit checks or credit scoring.
- Underwriting.
- Lender approval or rejection workflows.
- Lender agreement generation or issuance.
- Lender payment collection or servicing.
- Repayment collection, payment allocation, or lender-side accounting.
- Direct lender integrations unless explicitly approved as a later enhancement.
- Equipment or non-vehicle dealerships in the first release.
- Finance Lease and Operating Lease implementation in the initial HP release.
- Cloud deployment during normal development.

**Boundary:** LeaseFlow prepares the proposal and its supporting documents for the dealership to submit through the lender's external channel. It does not take responsibility for what the lender does after receiving the application.

### 4.3 Future expansion

The architecture should accommodate:

- Finance Lease.
- Operating Lease.
- Additional vehicle finance products.
- Equipment and other asset-based dealerships.
- Additional asset categories and configurable asset attributes.
- Optional lender integrations where supported and justified.
- Additional organization-specific modules and workflows.

Future capabilities must be introduced through deliberate scope and architecture decisions rather than assumed to be part of the first release.

## 5. Target Users

The initial users belong to vehicle dealership organizations.

| User | Main responsibilities |
|---|---|
| Organization Owner | Organization ownership and overall administration |
| Organization Admin | Users, roles, permissions, and configuration |
| Finance Manager | Finance operations, product configuration access, and oversight |
| Finance Officer | Calculations, quotations, proposals, and application preparation |
| Sales Officer | Customer and vehicle information, quotations, and sales-related workflows |
| Document Manager | Document library, templates, and supporting files |
| Accounts/Reporting User | Access to permitted operational and financial reports |
| Read-only User | View permitted records without modification |

Access must be controlled by organization membership, role, and permissions.

## 6. Core Product Modules

1. **Identity and Access Management** — Authentication, organizations, memberships, sessions, roles, permissions, and security.
2. **Organization Configuration** — Organization profile, branding, branches, localization, settings, and configurable business rules.
3. **Customer Management** — Customer records and relevant contact and financial information.
4. **Vehicle Management** — Vehicle records, specifications, values, and supporting information.
5. **Lender Management** — External lender reference records and dealership-lender panel relationships.
6. **Finance Product Management** — Product definitions, versions, configuration, and calculation rules.
7. **Finance Calculator** — Product-specific calculations with reproducible results.
8. **Quotation and Proposal Management** — Quotations, proposal preparation, and application packages.
9. **Document Management** — Organization-owned files, templates, versions, placeholders, and generated documents.
10. **Submission Preparation** — Final application package preparation and dealership-side submission records.
11. **Reporting and Audit** — Operational reporting, activity history, and traceability.

All modules will be accessible through one React web application.

## 7. Finance Product Strategy

The platform will be built around a product-oriented architecture.

Implementation sequence:

1. Hire Purchase (HP).
2. Finance Lease.
3. Operating Lease.
4. Additional finance products after their requirements are defined.

HP is the first complete working product. The other products will be introduced iteratively after HP is implemented, tested, and working end to end.

Product-specific calculations, eligibility/configuration rules, document requirements, and workflow differences must not be scattered as hardcoded assumptions throughout the system.

Finance calculations must be versioned and reproducible. The system should retain the inputs, output, product/rule version, and calculation-engine version used for each saved result.

## 8. Business and Data Ownership

The dealership is the primary organization and tenant in the initial release.

Each dealership owns its:

- Customers.
- Vehicles.
- Quotations and proposals.
- Documents and document templates.
- Organization settings.
- Users and permissions.
- Finance application preparation records.
- Operational activity history.

Lenders are external counterparties. A dealership may have relationships with multiple lenders, and a lender may work with multiple dealerships.

LeaseFlow may maintain lender reference information and dealership-lender panel configuration. It does not give lenders access to the dealership's tenant or data.

Strict tenant isolation must be enforced by backend authorization and data-access rules.

## 9. Technical Direction

### Backend

- Python
- FastAPI
- SQLAlchemy
- Alembic
- Pydantic
- Pytest

### Frontend

- React
- TypeScript
- A single web application
- Reusable component and form architecture

### Data and Infrastructure

- PostgreSQL
- Redis
- Celery or an equivalent background-job mechanism
- MinIO for local object storage
- Docker and Docker Compose

### Engineering and Observability

- GitHub Actions for CI
- Automated tests and code-quality checks
- Structured logging
- Prometheus and Grafana
- OpenTelemetry
- Error tracking integration where appropriate

Versions must be pinned and documented. Final technology choices and service boundaries will be recorded in the architecture specification before implementation.

## 10. Repository Structure

The project will use seven independent repositories:

| Repository | Responsibility |
|---|---|
| `leaseflow-platform` | Docker Compose, local orchestration, gateway, infrastructure, monitoring, and deployment configuration |
| `leaseflow-identity` | Authentication, organizations, memberships, users, roles, permissions, and sessions |
| `leaseflow-configuration` | Organization settings, branding, finance product definitions, versions, and configurable rules |
| `leaseflow-financing` | Customers, vehicles, lenders, calculations, quotations, proposals, and submission preparation |
| `leaseflow-documents` | File storage metadata, document versions, templates, placeholders, generation, and processing |
| `leaseflow-web` | The single React/TypeScript frontend |

Each repository has its own responsibilities and release/versioning discipline.

Shared contracts must not become a shared database-model package.

## 11. Database Ownership

The initial architecture uses four independent PostgreSQL databases:

| Database | Owner |
|---|---|
| `identity_db` | Identity service |
| `configuration_db` | Configuration service |
| `financing_db` | Financing service |
| `document_db` | Document service |

Rules:

- Each service owns its database and migrations.
- No service accesses another service's database directly.
- No cross-service database foreign keys.
- Relationships across services use stable identifiers and service APIs or events.
- Data access must respect organization boundaries.
- Shared contracts describe communication; they do not create shared database ownership.

## 12. Architecture Principles

1. Clear service ownership and responsibilities.
2. Independent database ownership.
3. API-first communication between services.
4. Tenant isolation by design.
5. Secure authentication and authorization.
6. Versioned finance products and calculation rules.
7. Reproducible financial calculations.
8. Configurable business behavior where future expansion requires it.
9. Reliable background processing for document operations.
10. Idempotency and retry safety for asynchronous operations.
11. Auditable business operations.
12. Automated testing from the beginning.
13. No unnecessary abstraction or premature microservice expansion.
14. Keep business logic independent from infrastructure where practical.

## 13. Development and Deployment Strategy

### Development

All development and testing will happen locally.

Docker Compose will provide a consistent environment for the services, databases, storage, queue, and supporting infrastructure.

A developer should be able to clone the repositories, follow the documented setup, configure local environment variables, and run the complete application.

Cloud resources must not be required for normal development.

### Final Deployment

Cloud deployment is a final project milestone, after the local application is stable and its operational requirements are documented.

The final deployment architecture may use managed PostgreSQL, object storage, container hosting, secrets management, monitoring, and other cloud services.

## 14. Quality Expectations

The project must demonstrate:

- Clean and maintainable code.
- Consistent naming and formatting.
- Input validation and meaningful error handling.
- Authentication and authorization.
- Tenant isolation.
- Database constraints and migrations.
- Unit, integration, and API tests.
- Frontend component and workflow tests.
- Secure document upload and download.
- Financial calculation tests.
- Structured logs and useful operational visibility.
- API documentation.
- Local setup documentation.
- Reproducible development environment.
- CI checks for relevant repositories.

Production-quality refers to engineering standards and operational discipline; it does not imply that the project is legally certified or approved for regulated commercial use.

## 15. Project Delivery Phases

| Phase | Deliverable |
|---|---|
| 1. Product definition | Charter, PRD, and approved scope |
| 2. Business analysis | Workflows, domain model, and business rules |
| 3. Technical design | Database design, architecture, contracts, and frontend specification |
| 4. Engineering foundation | Repositories, Docker Compose, CI, and service skeletons |
| 5. Core platform | Identity, organization configuration, and permissions |
| 6. HP implementation | Customer, vehicle, HP calculation, quotation, and proposal |
| 7. Document operations | Templates, uploads, generated documents, and application package |
| 8. End-to-end completion | Submission preparation, testing, audit, and reporting |
| 9. Product expansion | Finance Lease, Operating Lease, and further product types |
| 10. Deployment | Cloud deployment, security review, and operational validation |

The detailed sequencing and acceptance criteria will be defined in `10_IMPLEMENTATION_ROADMAP.md`.

## 16. Project Constraints

- Initial market: United Kingdom.
- Initial organization type: Vehicle dealership.
- Initial finance product: Hire Purchase.
- One frontend application.
- Seven repositories.
- Four independently owned databases.
- Local-first development.
- No lender-side workflows.
- No cloud dependency during development.
- Future functionality must be explicitly approved before being added.

## 17. Definition of Project Success

The initial release is successful when:

1. A developer can clone and run the complete application locally.
2. A dealership can register or be provisioned and manage its organization.
3. Authorized users can sign in and perform their permitted tasks.
4. The dealership can create and manage customers and vehicles.
5. A user can configure and use the initial HP product.
6. The system can produce tested and reproducible HP calculations.
7. A user can generate a quotation and prepare a finance proposal.
8. The dealership can manage supporting documents and templates.
9. The system can prepare a complete application package for external lender submission.
10. Organization data and documents remain isolated.
11. Core workflows have automated test coverage.
12. The architecture supports planned product and asset-category expansion without redesigning the whole system.

---

**Project direction:** Build the dealership's finance preparation platform. Hire Purchase comes first. External lender submission is the boundary. Everything else must support that goal.