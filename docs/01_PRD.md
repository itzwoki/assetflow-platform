# AssetFlow — Product Requirements Document (PRD)

**Document Version:** 1.1
**Status:** Approved scope baseline
**Market:** United Kingdom
**Initial Organization Type:** Vehicle Dealership
**Initial Finance Product:** Hire Purchase (HP)
**Related Document:** `00_PROJECT_CHARTER.md`

---

# 1. Product Overview

## 1.1 Purpose

AssetFlow is a dealership-side finance management platform that allows vehicle dealerships to manage the finance preparation lifecycle for their customers.

It centralizes customer and vehicle information, finance product configuration, finance calculations, quotations, proposals, and supporting documents.

The platform prepares a finance application package that the dealership can submit to an external lender through that lender's own portal or submission channel.

AssetFlow does not replace the lender's systems and does not manage lender-side activities.

## 1.2 Problem Statement

Vehicle dealerships need to coordinate customer information, vehicle details, finance calculations, quotations, proposal documents, and supporting documentation when preparing finance applications.

This information may be spread across spreadsheets, documents, internal systems, and lender portals.

AssetFlow aims to provide one structured dealership-side application for managing this preparation process.

## 1.3 Product Goal

Provide a reliable and extensible platform that allows a vehicle dealership to move from customer and vehicle information to a complete, documented finance proposal ready for submission to an external lender.

## 1.4 Product Boundary

The AssetFlow workflow ends at the dealership's submission of the prepared application package to the external lender.

The lender independently performs any checks, underwriting, credit decisions, approval/rejection, agreement issuance, and servicing.

AssetFlow will not recreate or control those lender operations.

---

# 2. Product Scope

## 2.1 Initial Release Scope

The first release will support:

- Vehicle dealership organizations.
- Organization users and access control.
- Organization configuration.
- Customer management.
- Vehicle management.
- External lender reference management.
- Dealership-lender panel relationships.
- Hire Purchase finance product.
- Finance calculation and saved calculation results.
- Quotations.
- Finance proposals/application preparation.
- Organization-owned document library.
- Document templates and placeholders.
- Document generation and conversion.
- Supporting document attachment.
- Finance application package preparation.
- Dealership-side submission record.
- Basic dashboards, reporting, and audit history.

## 2.2 Release Strategy

The product will be developed in increments.

| Release        | Scope                                                       |
| -------------- | ----------------------------------------------------------- |
| Release 1      | Vehicle dealerships and complete Hire Purchase workflow     |
| Release 2      | Finance Lease                                               |
| Release 3      | Operating Lease                                             |
| Later releases | Additional finance products and dealership/asset categories |

Release 1 must be complete and usable before expanding into the next finance product.

The architecture must support future product and asset-category expansion without implementing all future functionality in the first release.

## 2.3 Out of Scope

The following are explicitly excluded:

- Equipment dealerships in Release 1.
- Lender organizations as AssetFlow tenants.
- Lender user accounts and lender portals.
- Credit checks and credit scoring.
- Underwriting.
- Lender approval/rejection processing.
- Lender agreement creation or issuance.
- Lender-side payment processing or servicing.
- Customer repayment collection and allocation.
- Lender accounting and reconciliation.
- Direct lender API integration in the initial release.
- Automatic lender response synchronization.
- Cloud hosting during development.
- Finance Lease and Operating Lease implementation in the initial HP release.

---

# 3. Target Users and Roles

All Release 1 users belong to vehicle dealership organizations.

## 3.1 Organization Owner

- Owns or controls the dealership's AssetFlow organization.
- Manages organization-level access.
- Can access organization settings.
- Can assign administrative responsibilities.

## 3.2 Organization Administrator

- Manages dealership users.
- Assigns roles and permissions.
- Maintains organization configuration.
- Manages branches and available settings where enabled.

## 3.3 Finance Manager

- Oversees dealership finance preparation.
- Accesses finance calculations, quotations, and proposals.
- Reviews finance operations.
- Manages permitted finance configuration.

## 3.4 Finance Officer

- Creates and manages finance calculations.
- Prepares quotations.
- Creates finance proposals.
- Attaches supporting documents.
- Prepares application packages for lender submission.

## 3.5 Sales Officer

- Creates and updates customer records.
- Adds vehicle information.
- Initiates finance quotations where permitted.
- Views assigned customer and finance records.

## 3.6 Document Manager

- Manages the organization's document library.
- Manages document templates and versions.
- Uploads and organizes supporting documents.
- Maintains document metadata and categories.

## 3.7 Accounts/Reporting User

- Views permitted reports.
- Accesses relevant financial and operational summaries.
- Cannot access or modify records outside assigned permissions.

## 3.8 Read-only User

- Views authorized records.
- Cannot create, update, or delete business records.

Roles should be permission-based. The application should not rely solely on hardcoded role names to authorize operations.

---

# 4. Functional Requirements

## 4.1 Identity and Access Management

### Purpose

Provide secure access to AssetFlow and isolate dealership organizations.

### Requirements

**FR-IAM-001:** Users must be able to authenticate securely.

**FR-IAM-002:** The platform must support organization membership.

**FR-IAM-003:** A user must only access organizations to which they belong.

**FR-IAM-004:** The platform must support role-based and permission-based authorization.

**FR-IAM-005:** Administrators must be able to manage users and their assigned access, subject to their permissions.

**FR-IAM-006:** Authentication must support secure token-based access.

**FR-IAM-007:** Refresh tokens and sessions must be managed securely.

**FR-IAM-008:** Password reset and account recovery workflows must be supported.

**FR-IAM-009:** The platform must support account activation/deactivation.

**FR-IAM-010:** Security-sensitive actions must be auditable.

**FR-IAM-011:** Backend services must validate authorization rather than trusting frontend visibility controls.

**FR-IAM-012:** Service-to-service communication must use controlled service authentication.

### Security expectations

- Secure password hashing.
- Token expiry and refresh handling.
- Session invalidation where appropriate.
- Rate limiting for sensitive authentication endpoints.
- Protection against unauthorized organization switching.
- No credentials or secrets committed to repositories.

MFA and API keys may be included as later security enhancements unless required by the final identity specification.

---

## 4.2 Organization Management and Configuration

### Purpose

Allow each dealership to configure and manage its own AssetFlow environment.

### Requirements

**FR-ORG-001:** Each dealership must have an independent organization profile.

**FR-ORG-002:** The organization must be able to maintain its name, contact information, and business details.

**FR-ORG-003:** The organization must be able to configure its branding, including logo and relevant document branding.

**FR-ORG-004:** Organization settings must be isolated from other dealerships.

**FR-ORG-005:** The platform should support dealership branches and locations.

**FR-ORG-006:** Organization-level localization settings must be supported where required.

**FR-ORG-007:** Authorized administrators must be able to manage relevant configuration.

**FR-ORG-008:** Finance product availability must be configurable at organization level where appropriate.

**FR-ORG-009:** Organization settings must not allow one dealership to modify another dealership's configuration.

**FR-ORG-010:** Configuration changes affecting financial behavior must be versioned or auditable.

### Configuration categories

- Organization profile.
- Branding and document identity.
- Branches and locations.
- Users and permissions.
- Finance product availability.
- Asset categories and configurable fields.
- Document templates and assignments.
- Notification preferences.
- Regional settings.

Not every category needs a complex configuration UI in Release 1. The final configuration specification will define which settings are editable by dealership administrators.

---

## 4.3 Customer Management

### Purpose

Maintain dealership customer information required for finance preparation.

### Requirements

**FR-CUS-001:** Authorized users must be able to create customer records.

**FR-CUS-002:** Users must be able to view customer details.

**FR-CUS-003:** Users must be able to update customer information according to permissions.

**FR-CUS-004:** The system must support individual customers.

**FR-CUS-005:** Customer contact information must be maintained.

**FR-CUS-006:** The system must support relevant customer address information.

**FR-CUS-007:** Customer records must be searchable and filterable.

**FR-CUS-008:** The system should detect potential duplicate customer records.

**FR-CUS-009:** Customers must be associated with the correct dealership organization.

**FR-CUS-010:** Customer information used in a finance proposal must be traceable to the relevant customer record.

**FR-CUS-011:** The system must support the customer information fields required by the configured finance application workflow.

**FR-CUS-012:** Sensitive customer data must be protected through access controls and appropriate data-handling practices.

### Customer data considerations

The detailed domain model will determine the exact fields, including:

- Customer name.
- Contact details.
- Address.
- Date of birth where required.
- Employment and income details where required.
- Relevant identity information where required.
- Customer-specific notes and status.

Only information necessary for the dealership's legitimate workflow should be collected.

---

## 4.4 Vehicle Management

### Purpose

Maintain vehicle information used in quotations and finance proposals.

### Requirements

**FR-VEH-001:** Authorized users must be able to create vehicle records.

**FR-VEH-002:** Users must be able to view and update vehicle information.

**FR-VEH-003:** Vehicle records must support manufacturer, model, variant, and relevant specifications.

**FR-VEH-004:** Vehicle records must support registration and identification information where applicable.

**FR-VEH-005:** Vehicle records must support vehicle condition and classification where required.

**FR-VEH-006:** Vehicle records must support purchase price, sale price, and other relevant financial values.

**FR-VEH-007:** Users must be able to search and filter vehicle records.

**FR-VEH-008:** Vehicles must be associated with the correct dealership organization.

**FR-VEH-009:** Vehicles must be linkable to quotations and finance proposals.

**FR-VEH-010:** The platform should support relevant vehicle images and supporting documents.

**FR-VEH-011:** The system must support the distinction between new and used vehicles where relevant.

**FR-VEH-012:** Vehicle data must be validated before it is used in a finance calculation or proposal.

### Future extensibility

Release 1 focuses exclusively on vehicles.

However, asset fields and categories should be designed so future releases can introduce other asset types without embedding vehicle-specific assumptions into every finance module.

This does not require building an unrestricted generic asset-management product in Release 1.

---

## 4.5 External Lender Management

### Purpose

Maintain reference information about lenders with which a dealership works.

Lenders are external counterparties, not AssetFlow tenant organizations.

### Requirements

**FR-LND-001:** A dealership must be able to maintain its list of external lenders.

**FR-LND-002:** A lender reference must support relevant business and contact information.

**FR-LND-003:** A dealership must be able to configure its relationship with each lender.

**FR-LND-004:** A dealership may have relationships with multiple lenders.

**FR-LND-005:** The same lender may be referenced by multiple dealerships without exposing one dealership's private data to another.

**FR-LND-006:** A lender relationship may specify supported finance products.

**FR-LND-007:** Lender reference records may include an external portal URL and dealership-specific reference information.

**FR-LND-008:** Users must be able to select an applicable lender when preparing a proposal.

**FR-LND-009:** Lender reference data must be isolated by dealership where it contains dealership-specific information.

### Lender relationship information

Potential fields include:

- Lender name.
- Contact information.
- Portal URL.
- Dealership account/reference number.
- Supported finance products.
- Relationship status.
- Effective dates.
- Internal notes.

The exact data structure will be defined in the domain model.

### Explicit boundary

AssetFlow does not create lender accounts, manage lender employees, or process lender decisions.

---

## 4.6 Finance Product Configuration

### Purpose

Provide an extensible way to define finance products and their applicable rules.

### Product roadmap

| Product             | Release   |
| ------------------- | --------- |
| Hire Purchase       | Release 1 |
| Finance Lease       | Release 2 |
| Operating Lease     | Release 3 |
| Additional products | Future    |

### Requirements

**FR-PROD-001:** The platform must represent finance products as distinct product definitions.

**FR-PROD-002:** Finance product definitions must support versioning.

**FR-PROD-003:** Each product must define its applicable calculation behavior.

**FR-PROD-004:** Product-specific inputs and validation rules must be configurable or explicitly defined.

**FR-PROD-005:** Product configuration must be separated from customer and vehicle records.

**FR-PROD-006:** The platform must retain the product version used by a saved calculation.

**FR-PROD-007:** Product availability may be controlled at organization level.

**FR-PROD-008:** Lender relationships may specify which products they support.

**FR-PROD-009:** Product-specific document requirements must be supported.

**FR-PROD-010:** The architecture must permit future products without changing unrelated modules unnecessarily.

### Important design rule

Finance products must not be implemented as a single calculator containing scattered conditional logic for every possible product.

Each product must have a clearly defined calculation and workflow responsibility.

---

## 4.7 Hire Purchase Finance Calculator

### Purpose

Calculate and present Hire Purchase finance options using versioned, testable rules.

### Requirements

**FR-HP-001:** Authorized users must be able to initiate an HP calculation.

**FR-HP-002:** The calculation must accept the required vehicle and finance inputs.

**FR-HP-003:** The calculation must validate its inputs.

**FR-HP-004:** The calculation must produce the applicable financial outputs defined in the financial rules specification.

**FR-HP-005:** The user must be able to review the calculation results.

**FR-HP-006:** A calculation may be saved and associated with a customer and vehicle.

**FR-HP-007:** Saved calculations must retain their input values.

**FR-HP-008:** Saved calculations must retain their output values.

**FR-HP-009:** Saved calculations must retain the applicable product and calculation-rule versions.

**FR-HP-010:** Recalculations must not silently change previously saved calculation results.

**FR-HP-011:** The system must support automated tests for calculation accuracy and edge cases.

**FR-HP-012:** The system must distinguish an indicative calculation from a finalized quotation.

### Calculation design considerations

The detailed financial rules specification must define:

- Vehicle cash price.
- Deposit.
- Amount financed.
- Interest/rate assumptions.
- Term.
- Payment frequency.
- Fees and other applicable charges.
- Payment amounts.
- Total payable.
- Interest/finance charge.
- Applicable rounding rules.
- Validation boundaries and edge cases.

The exact formulas and UK-specific treatment must be researched and documented in `08_FINANCIAL_RULES.md`.

The platform must not present an unverified calculation as a legally compliant lender offer.

---

## 4.8 Quotation Management

### Purpose

Create and manage dealership finance quotations for customers.

### Requirements

**FR-QUO-001:** Authorized users must be able to create a quotation for a customer.

**FR-QUO-002:** A quotation must reference a vehicle.

**FR-QUO-003:** A quotation must reference the relevant finance product.

**FR-QUO-004:** An HP quotation must use a saved or reproducible HP calculation.

**FR-QUO-005:** A quotation must retain its financial values and calculation reference.

**FR-QUO-006:** Users must be able to review and update quotations according to their status and permissions.

**FR-QUO-007:** The system must support quotation statuses.

**FR-QUO-008:** Users must be able to generate a quotation document.

**FR-QUO-009:** The quotation must be associated with the dealership's document library and organization.

**FR-QUO-010:** The system must preserve historical quotation values when a new version is created.

**FR-QUO-011:** Quotations must be searchable and filterable.

### Quotation lifecycle

The exact statuses will be defined in the business workflow specification. A starting model may include:

- Draft.
- Ready for review.
- Issued.
- Accepted.
- Declined.
- Expired.
- Cancelled.

These are dealership-side quotation statuses, not lender approval statuses.

---

## 4.9 Finance Proposal and Application Preparation

### Purpose

Prepare a complete finance proposal/application package for external lender submission.

### Requirements

**FR-PROP-001:** Authorized users must be able to create a proposal from an eligible quotation or finance record.

**FR-PROP-002:** A proposal must reference the customer, vehicle, finance product, and relevant calculation.

**FR-PROP-003:** A proposal must contain the information required by the dealership's selected submission workflow.

**FR-PROP-004:** A proposal must allow the selection of an external lender.

**FR-PROP-005:** The system must validate required information before the proposal can be marked ready.

**FR-PROP-006:** The system must identify missing information and required supporting documents.

**FR-PROP-007:** Users must be able to attach relevant supporting documents.

**FR-PROP-008:** Users must be able to review the complete proposal before finalizing it.

**FR-PROP-009:** The system must generate the required proposal documents using the dealership's assigned templates.

**FR-PROP-010:** The system must prepare an organized application package for external submission.

**FR-PROP-011:** The application package must be accessible to authorized dealership users.

**FR-PROP-012:** The system must record the proposal's preparation and finalization history.

**FR-PROP-013:** The system must prevent unauthorized modification of finalized proposal data.

**FR-PROP-014:** If a finalized proposal needs material changes, the system must preserve the original and create an appropriate revised version.

### Proposal lifecycle

A proposed dealership-side lifecycle:

1. Draft.
2. In progress.
3. Pending internal review, if enabled.
4. Ready for submission.
5. Submitted externally.
6. Cancelled or superseded.

The exact status model and transitions will be finalized in `02_BUSINESS_WORKFLOWS.md`.

### Submission boundary

When the dealership submits the package through an external lender portal or channel, AssetFlow may record basic submission information, such as:

- Selected lender.
- Submission date and time.
- Submitting dealership user.
- External reference number, if available.
- Submitted proposal/package version.

This is a dealership audit record only.

The initial release will not synchronize lender responses, process lender decisions, or manage lender-side activities.

---

## 4.10 Document Management

### Purpose

Provide every dealership with its own document library and document-generation capabilities.

### Requirements

**FR-DOC-001:** Each dealership must have its own document library.

**FR-DOC-002:** A dealership must only access documents belonging to its organization.

**FR-DOC-003:** Authorized users must be able to upload documents.

**FR-DOC-004:** Users must be able to categorize and organize documents.

**FR-DOC-005:** Document metadata must be stored separately from the underlying file content.

**FR-DOC-006:** The platform must support document versions where applicable.

**FR-DOC-007:** The platform must support organization-owned document templates.

**FR-DOC-008:** Authorized users must be able to manage document templates.

**FR-DOC-009:** Templates must support defined placeholders.

**FR-DOC-010:** The platform must generate documents using the relevant organization template and business data.

**FR-DOC-011:** Generated documents must be associated with the relevant business record.

**FR-DOC-012:** The platform must support PDF output for applicable documents.

**FR-DOC-013:** The system must validate required placeholder values before document generation.

**FR-DOC-014:** Failed document processing must produce a meaningful error and support safe retry where appropriate.

**FR-DOC-015:** Document download must use authorized access.

**FR-DOC-016:** The platform must record relevant document activities.

### Document categories

Potential categories include:

- Customer documents.
- Vehicle documents.
- Quotation documents.
- Finance proposal documents.
- Supporting application documents.
- Organization documents.
- Generated documents.

### Template ownership

A dealership's templates belong to that dealership.

When generating a quotation or proposal, AssetFlow must use the appropriate template assigned by that dealership.

AssetFlow must not access or retrieve a lender's private document templates.

### Storage

During local development, object storage will be provided through MinIO.

The document service will own document metadata and access rules. Actual files will be stored in object storage.

The final cloud deployment may use a compatible managed object-storage service.

---

## 4.11 Dashboard and Reporting

### Purpose

Provide dealership users with visibility into their own operational activity.

### Requirements

**FR-REP-001:** Users must have a dashboard appropriate to their permissions.

**FR-REP-002:** The dashboard must display relevant customer, vehicle, quotation, and proposal information.

**FR-REP-003:** Users must be able to view proposal preparation and submission counts.

**FR-REP-004:** Users must be able to filter relevant records.

**FR-REP-005:** The platform must provide basic operational reports.

**FR-REP-006:** Reports must respect organization and user permissions.

**FR-REP-007:** Financial figures displayed in reports must be traceable to their source records.

### Initial reporting examples

- Total customers.
- Total vehicles.
- Quotations created.
- Quotations by status.
- Proposals prepared.
- Proposals ready for submission.
- Proposals submitted externally.
- Finance product usage.
- Activity by dealership user.

The first release does not require a full business intelligence platform.

---

## 4.12 Audit History

### Purpose

Maintain traceability for important business and security operations.

### Requirements

**FR-AUD-001:** The platform must record important user actions.

**FR-AUD-002:** Audit records must identify the organization and acting user where applicable.

**FR-AUD-003:** Audit records must identify the affected business entity.

**FR-AUD-004:** Relevant timestamps must be recorded consistently.

**FR-AUD-005:** Important changes to finance calculations and proposals must be traceable.

**FR-AUD-006:** Document upload, update, generation, and access-related events must be recorded where appropriate.

**FR-AUD-007:** Security-sensitive activities must be auditable.

**FR-AUD-008:** Normal users must not be able to modify or delete audit history.

Audit requirements must be balanced with data minimization and retention requirements.

---

# 5. Core Business Workflows

These are high-level workflows. Detailed states, validations, exceptions, and transitions belong in `02_BUSINESS_WORKFLOWS.md`.

## 5.1 Dealership Setup

1. Dealership organization is created or provisioned.
2. Organization profile and settings are configured.
3. Owner/admin account is established.
4. Users are invited or created.
5. Roles and permissions are assigned.
6. Branches and organization settings are configured.
7. External lender references are added.
8. Finance product availability is configured.

## 5.2 Customer and Vehicle Setup

1. User creates or selects a customer.
2. User records required customer information.
3. User creates or selects a vehicle.
4. User records vehicle specifications and relevant values.
5. Customer and vehicle records become available for finance preparation.

## 5.3 Hire Purchase Calculation

1. User selects a customer.
2. User selects a vehicle.
3. User selects Hire Purchase.
4. User enters the required finance inputs.
5. System validates the inputs.
6. System calculates the HP result.
7. User reviews the result.
8. User saves the calculation if required.

## 5.4 Quotation Preparation

1. User creates a quotation.
2. System links the customer and vehicle.
3. User selects the applicable HP calculation.
4. System prepares quotation values.
5. User reviews the quotation.
6. User generates the quotation document.
7. Quotation is issued or retained in draft according to its status.

## 5.5 Proposal Preparation

1. User creates a finance proposal.
2. System obtains the relevant customer, vehicle, quotation, and calculation data.
3. User selects an external lender.
4. User completes required application information.
5. System validates mandatory fields.
6. System identifies missing supporting documents.
7. User uploads or selects the required documents.
8. System generates the relevant proposal documents.
9. User reviews the complete application package.
10. Proposal becomes ready for submission.

## 5.6 External Lender Submission

1. Dealership user downloads or accesses the prepared application package.
2. User submits the application through the lender's external portal or channel.
3. User records the submission in AssetFlow.
4. AssetFlow records the relevant submission details and package version.

**The workflow ends here.** Lender-side processing is outside AssetFlow's initial scope.

---

# 6. Domain and Data Requirements

The final entity definitions and relationships will be documented in `03_DOMAIN_MODEL.md`.

The initial domain is expected to include:

| Domain           | Main concepts                                                              |
| ---------------- | -------------------------------------------------------------------------- |
| Identity         | User, organization, membership, role, permission, session                  |
| Configuration    | Organization settings, branch, product definition, product version, rules  |
| Customer         | Customer, contact details, address, relevant application information       |
| Vehicle          | Vehicle, specifications, identifiers, valuation                            |
| Lender reference | Lender, dealership-lender relationship, supported products                 |
| Financing        | Calculation, quotation, proposal, proposal version, submission record      |
| Documents        | File metadata, document version, template, placeholder, generated document |
| Audit            | Audit event, actor, entity reference, timestamp                            |

## 6.1 Data Ownership

The initial architecture has four database-owning services:

- Identity service: identity and organization membership.
- Configuration service: dealership configuration and finance product definitions.
- Financing service: customers, vehicles, lenders, calculations, quotations, proposals, and submission records.
- Document service: document metadata, versions, templates, and processing records.

Cross-service relationships use identifiers and service APIs/events. There must be no direct cross-service database access or foreign keys.

## 6.2 Tenant Isolation

Every dealership-owned business record must be associated with the correct organization context.

Backend services must enforce organization authorization on every relevant operation.

Knowing a record ID must never be sufficient to access another dealership's data.

Documents require the same organization-level isolation as business records.

---

# 7. Non-Functional Requirements

## 7.1 Security

**NFR-SEC-001:** All protected endpoints must require valid authentication.

**NFR-SEC-002:** Authorization must be enforced by backend services.

**NFR-SEC-003:** Tenant isolation must be enforced at the data-access layer.

**NFR-SEC-004:** Sensitive information must not be exposed through logs or error responses.

**NFR-SEC-005:** Secrets must be managed outside source code.

**NFR-SEC-006:** File uploads must validate allowed file types, size, and other relevant safety constraints.

**NFR-SEC-007:** File access must use authorization-controlled mechanisms.

**NFR-SEC-008:** APIs must validate and normalize user input.

**NFR-SEC-009:** Authentication and sensitive endpoints must have appropriate rate limits.

**NFR-SEC-010:** Security-relevant events must be auditable.

## 7.2 Performance

- Common API operations should have reasonable response times under expected local test workloads.
- Database queries must be indexed appropriately.
- Large document operations must not unnecessarily block API request handling.
- Background processing should be used for expensive document tasks.
- Reporting queries must avoid unnecessary full-table scans.
- Performance-sensitive operations should be measured and optimized based on evidence.

Exact performance targets will be set in the engineering standards and implementation roadmap.

## 7.3 Reliability

- Services must handle expected errors consistently.
- Database migrations must be controlled and repeatable.
- Background jobs must support safe retries.
- Duplicate job execution must not create duplicate business results.
- Critical business operations must preserve data consistency.
- Application failures must produce useful logs.
- Local services must recover predictably after restart.

## 7.4 Maintainability

- Clear service boundaries.
- Consistent project structure.
- Type-safe contracts where practical.
- Documented API behavior.
- Automated formatting and linting.
- Reusable frontend components.
- Testable business logic.
- No unnecessary coupling between services.

## 7.5 Scalability and Extensibility

The initial system does not need enterprise-scale traffic, but its design should avoid preventable limitations.

It must support:

- Additional finance products.
- Additional asset categories.
- Additional dealership organizations.
- Organization-specific configurations.
- Growing document libraries.
- Additional API consumers.

## 7.6 Observability

The platform should provide:

- Structured logs.
- Health checks.
- Service-level metrics.
- Error visibility.
- Request correlation.
- Background-job monitoring.
- Basic distributed tracing.

---

# 8. Technical Constraints

## 8.1 Application Architecture

- One React/TypeScript frontend.
- Python/FastAPI backend services.
- Four independently owned PostgreSQL databases.
- Six repositories.
- Docker Compose for local orchestration.
- API-based communication between services.

## 8.2 Storage

- PostgreSQL for structured business data.
- MinIO for local object storage.
- Redis for caching or background-job coordination where appropriate.
- Celery or an equivalent task system for asynchronous work.

## 8.3 Local Development

All services must run locally.

A developer must not need AWS or other paid cloud resources to build, test, or run the application.

The platform repository should provide the main local startup and orchestration workflow.

## 8.4 Cloud Deployment

Cloud deployment is deferred until the local system is complete and validated.

The deployment design must be documented so local infrastructure choices can be replaced by production equivalents without changing core business logic.

---

# 9. Frontend Requirements

The application will use one React/TypeScript frontend with navigation organized around dealership operations.

Potential navigation structure:

1. Dashboard.
2. Customers.
3. Vehicles.
4. Lenders.
5. Finance Calculator.
6. Quotations.
7. Proposals.
8. Documents.
9. Reports.
10. Organization Settings.
11. User and Access Management.

Navigation and individual screens must respect permissions.

The final page-level requirements, component structure, navigation behavior, and user interactions will be defined in `07_FRONTEND_SPEC.md`.

---

# 10. API and Integration Requirements

## 10.1 Internal APIs

Each service must expose documented APIs for its owned data and responsibilities.

The contracts repository will contain versioned shared schemas and event definitions.

## 10.2 External Lenders

Release 1 will support manual external submission.

The platform will prepare the required application information and supporting documents for the dealership to submit through the lender's own channel.

No lender API integration or automatic lender response handling is required for the initial release.

The architecture should not prevent future integrations, but they must be treated as separate, explicitly scoped work.

## 10.3 Document Processing

Document generation and conversion should support asynchronous processing where needed.

Jobs must have defined status, failure handling, and retry behavior.

---

# 11. Financial Rules and Calculation Integrity

Finance calculations are central to the product and require particular care.

Requirements:

1. Every calculation must identify its finance product.
2. Every calculation must use a defined version of its applicable rules.
3. Inputs and outputs must be retained for saved calculations.
4. Rounding behavior must be explicit.
5. Invalid input combinations must be rejected.
6. Calculation tests must cover normal, boundary, and exceptional cases.
7. Previously saved results must remain reproducible.
8. A change to calculation rules must not silently rewrite historical quotations or proposals.
9. The application must distinguish illustrative calculations from lender-approved offers.
10. Financial rule documentation must identify assumptions and reliable references.

The detailed HP calculation specification belongs in `08_FINANCIAL_RULES.md`.

The system must not claim regulatory compliance merely because a calculation has been implemented. Regulatory and legal requirements must be separately researched and validated before any real commercial use.

---

# 12. Document and Data Governance

The platform must support appropriate handling of customer and dealership information.

Requirements:

- Organization-owned document libraries.
- Access checks for every file operation.
- Document version history where applicable.
- Secure storage and controlled downloads.
- Appropriate retention and deletion policies.
- Auditability of important document operations.
- Avoidance of unnecessary sensitive-data collection.
- Safe handling of generated files and temporary processing artifacts.

Specific UK data protection, retention, and regulatory requirements must be assessed before commercial deployment. This PRD does not constitute legal advice or a compliance certification.

---

# 13. MVP Acceptance Criteria

Release 1 will be considered complete when the following can be demonstrated locally.

## 13.1 Platform

- [ ] All required repositories can be cloned.
- [ ] The platform can start the application using documented local instructions.
- [ ] Required databases and supporting services start successfully.
- [ ] Database migrations run successfully.
- [ ] Health checks are available.
- [ ] CI checks run for the relevant repositories.

## 13.2 Identity and Organization

- [ ] A dealership organization can be provisioned.
- [ ] Authorized users can authenticate.
- [ ] Users can belong to the correct dealership.
- [ ] Administrators can manage permitted users and roles.
- [ ] Unauthorized users cannot access protected resources.
- [ ] One dealership cannot access another dealership's data.

## 13.3 Customer and Vehicle

- [ ] Users can create, view, update, and search customers.
- [ ] Users can create, view, update, and search vehicles.
- [ ] Customer and vehicle records are organization-scoped.
- [ ] Required validation is applied.

## 13.4 Lenders and Products

- [ ] A dealership can maintain its external lender references.
- [ ] A dealership can configure its lender relationships.
- [ ] Lender reference information is not exposed across dealership tenants.
- [ ] Hire Purchase is available as the initial finance product.
- [ ] Product and calculation rules are versioned.

## 13.5 Hire Purchase

- [ ] A user can enter valid HP inputs.
- [ ] The system validates inputs.
- [ ] The calculator returns expected results.
- [ ] Calculation tests cover important edge cases.
- [ ] Saved calculations retain inputs, outputs, and rule versions.

## 13.6 Quotations and Proposals

- [ ] A user can create a quotation for a customer and vehicle.
- [ ] The quotation uses a reproducible HP calculation.
- [ ] A user can generate a quotation document.
- [ ] A user can prepare a finance proposal.
- [ ] The proposal references the correct customer, vehicle, product, and calculation.
- [ ] Required information is validated.
- [ ] Missing supporting documents are identified.
- [ ] A complete proposal package can be prepared for external submission.
- [ ] Finalized proposal versions are preserved.

## 13.7 Documents

- [ ] Each dealership has its own document library.
- [ ] Users cannot access another dealership's documents.
- [ ] Authorized users can upload and manage files.
- [ ] Authorized users can manage their templates.
- [ ] The system can generate documents from templates.
- [ ] Generated documents are associated with the correct business record.
- [ ] Failed document processing can be identified and handled safely.

## 13.8 External Submission Boundary

- [ ] A dealership can prepare a complete application package.
- [ ] The package can be accessed for manual submission through an external lender channel.
- [ ] A user can record the submission event and relevant reference information.
- [ ] The submission record identifies the proposal version and acting user.
- [ ] No lender-side underwriting, approval, agreement, or servicing workflow is required.

## 13.9 Quality

- [ ] Core backend business logic has automated tests.
- [ ] Critical API workflows have integration tests.
- [ ] Important frontend workflows have tests.
- [ ] Security and tenant-isolation cases have tests.
- [ ] Logs and errors are useful for debugging.
- [ ] Local setup and project documentation are complete.

---

# 14. Risks and Mitigations

| Risk                                     | Mitigation                                                                      |
| ---------------------------------------- | ------------------------------------------------------------------------------- |
| HP calculation assumptions are incorrect | Research and document formulas; test against verified examples                  |
| Product-specific logic becomes hardcoded | Use product-specific rule definitions and versioned calculation logic           |
| Future asset types require major changes | Keep asset categories and fields extensible without over-generalizing Release 1 |
| Cross-tenant data leakage                | Enforce organization scoping in backend services and test isolation             |
| Document access vulnerabilities          | Use controlled storage access, authorization, and file validation               |
| Services become tightly coupled          | Define API contracts and clear data ownership                                   |
| Scope expands into lender operations     | Keep external lender submission as the explicit product boundary                |
| Local setup becomes difficult            | Maintain Docker Compose and a documented single-entry local workflow            |
| Financial results change unexpectedly    | Preserve calculation inputs, outputs, and rule versions                         |

---

# 15. Assumptions and Dependencies

## Assumptions

- The first target users are vehicle dealerships in the UK.
- Dealerships maintain their own customer, vehicle, quotation, proposal, and document information.
- Dealerships work with external lenders.
- Lenders retain control of their own systems and decision processes.
- Initial lender submission is manual through external portals or channels.
- Hire Purchase is the first fully implemented finance product.
- Other finance products will be added iteratively.
- The platform is initially a local-first portfolio project.

## Dependencies

- Verified HP financial rules and assumptions.
- Agreed customer and vehicle data requirements.
- Agreed proposal and supporting-document requirements.
- Document template and PDF processing decisions.
- Approved service and database boundaries.
- A defined local development and testing environment.

---

# 16. Future Considerations

The following are possible future enhancements, not Release 1 commitments:

- Finance Lease.
- Operating Lease.
- Additional finance products.
- Equipment and other asset-based dealerships.
- More configurable asset categories and fields.
- Lender API integrations where supported.
- Automated lender submission where permitted and available.
- External status synchronization where supported.
- Additional reporting and analytics.
- Advanced approval workflows within a dealership.
- Additional document automation.
- Cloud deployment and production operations.

Each enhancement must have its own requirements and acceptance criteria before implementation.

---

# 17. Related Project Documents

| Document                       | Purpose                                                             |
| ------------------------------ | ------------------------------------------------------------------- |
| `00_PROJECT_CHARTER.md`        | Project vision, scope, architecture direction, and success criteria |
| `01_PRD.md`                    | Product requirements and Release 1 scope                            |
| `02_BUSINESS_WORKFLOWS.md`     | Detailed dealership workflows and state transitions                 |
| `03_DOMAIN_MODEL.md`           | Entities, relationships, and domain boundaries                      |
| `04_DATABASE_DESIGN.md`        | Tables, fields, constraints, and indexes                            |
| `05_SYSTEM_ARCHITECTURE.md`    | Services, communication, infrastructure, and security architecture  |
| `06_API_CONTRACTS.md`          | API endpoints, schemas, and event contracts                         |
| `07_FRONTEND_SPEC.md`          | Navigation, screens, components, and UI behavior                    |
| `08_FINANCIAL_RULES.md`        | HP financial formulas, assumptions, examples, and tests             |
| `09_ENGINEERING_STANDARDS.md`  | Coding, testing, security, and operational standards                |
| `10_IMPLEMENTATION_ROADMAP.md` | Development phases and implementation sequence                      |
| `DECISIONS.md`                 | Approved decisions and changes to project direction                 |

---

# 18. Change Control

This PRD is the baseline for Release 1.

Any change affecting product scope, business boundaries, finance calculations, service ownership, database ownership, or core workflows must be recorded in `DECISIONS.md`.

The project must not silently change requirements between implementation sessions.

Future changes must identify:

1. What is changing.
2. Why it is changing.
3. Which requirements and documents are affected.
4. Whether the change affects existing data or architecture.
5. Whether the change is approved for the current release or deferred.

**Release 1 product boundary: A vehicle dealership prepares an HP finance proposal and its supporting documents, then submits the package to an external lender. AssetFlow does not manage what happens inside the lender's business.**
