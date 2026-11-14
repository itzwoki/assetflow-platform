# 03 — Domain Model

**Project:** AssetFlow  
**Document:** Domain Model  
**Version:** 1.0  
**Initial Market:** United Kingdom  
**Release:** Release 1 — Vehicle Dealerships / Hire Purchase  
**Status:** Draft for Review

---

# 1. Purpose

This document defines AssetFlow's business domain: the concepts the platform manages, what each concept represents, how concepts relate to one another, and which business rules must always remain true.

It provides the foundation for:

- Database design
- Service boundaries
- API contracts
- Financial calculation design
- Application workflows
- Authorization and tenant isolation

This document describes the domain independently of database tables or framework-specific implementation.

# 2. Domain Boundaries

AssetFlow is a dealership-facing finance proposal preparation and management platform.

In Release 1, a dealership can:

1. Manage its organization and users.
2. Manage customers and vehicles.
3. Maintain relationships with external finance lenders.
4. Calculate Hire Purchase finance illustrations.
5. Prepare customer quotations.
6. Prepare finance proposals and supporting documents.
7. Generate an application package.
8. Submit the package externally through a lender's portal or another external channel.
9. Record that submission in AssetFlow.
10. Track its own internal records and reporting.

AssetFlow does not perform lender-side underwriting, credit decisions, agreement creation, or payment collection.

## 2.1 Domain actors

| Actor | Responsibility |
|---|---|
| Dealership | Primary organization using AssetFlow |
| Dealership User | Employee or authorized representative operating the platform |
| Customer | Person or business purchasing or financing a vehicle |
| Lender | External finance provider receiving proposals |
| System Administrator | Manages platform-level operations where applicable |
| Background Worker | Performs asynchronous processing such as document generation |

A customer and lender are not automatically platform users.

# 3. Domain Ownership

Each service owns its domain entities and is the only service allowed to modify their authoritative records.

| Service | Domain ownership |
|---|---|
| Identity | Users, organizations, memberships, authentication and sessions |
| Configuration | Organization settings, roles, permissions, product definitions, rules and document-template assignments |
| Financing | Customers, vehicles, lender records and relationships, calculations, quotations, proposals and submission records |
| Documents | Uploaded files, document metadata, versions, templates, generated documents and processing jobs |
| Web | User interface; no authoritative business data ownership |
| Contracts | Shared API and event contracts; no business data ownership |
| Platform | Runtime infrastructure and orchestration; no business data ownership |

Cross-service relationships use identifiers and service APIs or events. A service must not directly access another service's database.

# 4. Identity and Organization Domain

This domain is owned by `AssetFlow-identity`.

## 4.1 Organization

Represents a dealership business using AssetFlow.

Responsibilities:

- Acts as the primary tenant boundary.
- Owns dealership-specific business records.
- Has organization-level settings and branding.
- Has users associated through memberships.
- Has its own lender panel relationships, customers, vehicles, quotations and proposals.

Important rules:

- Every dealership business record must belong to exactly one organization.
- An organization must never access another organization's private records.
- Organization identifiers are used across services to enforce tenant isolation.
- An organization may have multiple users and branches.

## 4.2 User

Represents an individual with an authenticated AssetFlow account.

Responsibilities:

- Authentication and account management.
- Access to one or more organizations through memberships.
- Performing authorized actions.
- Being recorded as the actor for relevant business operations.

A user is not inherently a member of every organization.

## 4.3 Organization Membership

Represents a user's association with an organization.

Responsibilities:

- Connects a user to a dealership.
- Defines the user's organization-level role and access.
- Controls whether the user can access the organization's resources.

Relationship:

- One organization can have many memberships.
- One user can have memberships in multiple organizations.
- Each membership connects one user to one organization.

A user may belong to multiple dealerships without those dealerships sharing business data.

## 4.4 Role and Permission

Defines what a user can do within an organization.

Examples:

- Manage customers.
- Manage vehicles.
- View calculations.
- Create quotations.
- Prepare proposals.
- Record external submissions.
- Manage documents.
- View reports.
- Manage organization settings.

Roles group permissions. Actual authorization is enforced by backend services, not merely by hiding frontend controls.

Release 1 does not require a mandatory internal approval workflow.

# 5. Configuration Domain

This domain is owned by `AssetFlow-configuration`.

It defines how an organization and its finance products behave without owning transactional finance records.

## 5.1 Organization Settings

Represents dealership-specific configuration.

Examples:

- Business details.
- Contact information.
- Branding.
- Currency and locale preferences.
- Default quotation settings.
- Document preferences.
- Notification preferences.

Organization settings must be scoped to the owning organization.

## 5.2 Branch

Represents a physical or operational dealership branch.

Responsibilities:

- Store branch-specific details.
- Associate relevant users and business operations with a location.
- Support future multi-branch reporting and access rules.

A branch belongs to one organization. Branch-level functionality can be introduced incrementally; Release 1 does not need complex branch hierarchies.

## 5.3 Finance Product Definition

Represents a finance product supported by AssetFlow.

Examples:

- Hire Purchase (HP)
- Finance Lease
- Operating Lease

Responsibilities:

- Identify the finance product.
- Define its supported capabilities.
- Connect the product to applicable financial rules.
- Support product-specific fields and document requirements.
- Provide a consistent extension point for future products.

Release 1 activates Hire Purchase only.

Finance products must not be treated as interchangeable calculation labels. Each product has its own business and contractual characteristics.

## 5.4 Finance Product Version

Represents a version of a product's configuration and applicable rules.

Responsibilities:

- Preserve historical product definitions.
- Identify the exact configuration used by a calculation or proposal.
- Support future changes without silently changing historical records.

A historical calculation or proposal must remain reproducible using the product and rule versions originally selected.

## 5.5 Financial Rule Set

Represents a versioned collection of financial rules used by a finance product.

Examples:

- Interest calculation method.
- Deposit handling.
- Payment frequency.
- Term constraints.
- Fees and charges.
- Rounding rules.
- Product-specific calculation parameters.

Financial rules are separate from the calculation transaction itself.

The calculation engine consumes an applicable rule set and produces a result. It must not silently modify an existing historical calculation when rules change.

Exact Hire Purchase formulas and UK-specific financial assumptions are defined in `08_FINANCIAL_RULES.md`.

## 5.6 Asset Category and Field Definition

Represents configurable categories and attributes for assets supported by AssetFlow.

Release 1 supports vehicles.

The model should allow future expansion to other asset categories without redesigning every finance workflow. However, vehicle-specific information must remain explicit rather than being hidden inside an unrestricted collection of generic fields.

Examples of future categories:

- Commercial vehicles.
- Equipment.
- Machinery.

## 5.7 Document Template Assignment

Represents which document template an organization uses for a particular purpose.

Examples:

- Hire Purchase quotation.
- Customer-facing finance illustration.
- Finance proposal supporting document.

Assignments can be organization-specific and product-specific.

The configuration service owns the assignment. The documents service owns the actual template and generated file.

# 6. Financing Domain

This domain is owned by `AssetFlow-financing`.

It contains the dealership's operational finance records.

## 6.1 Customer

Represents a person or business interested in purchasing or financing a vehicle.

Responsibilities:

- Store customer identity and contact information.
- Store relevant customer classification.
- Associate customer records with an organization.
- Support multiple quotations and proposals over time.

Possible customer classifications:

- Individual.
- Business.

Important rules:

- A customer record belongs to one dealership organization.
- Customer data must not be shared automatically between dealerships.
- A customer can have multiple quotations.
- A customer can have multiple proposals.
- Historical quotations and proposals must preserve the relevant customer information used when they were created.

Customer master data may change over time. A change to a customer's current contact details must not silently rewrite historical proposal information.

## 6.2 Vehicle

Represents a vehicle managed by a dealership.

Responsibilities:

- Store vehicle identification and descriptive information.
- Associate the vehicle with the dealership.
- Track its dealership inventory status.
- Make vehicle information available to calculations, quotations and proposals.

Vehicle information may include:

- Registration number.
- VIN.
- Make.
- Model.
- Variant.
- Model year.
- Mileage.
- Fuel type.
- Transmission.
- Vehicle condition.
- Sale price.
- Additional descriptive information.

Important rules:

- A vehicle belongs to one dealership organization.
- A vehicle may be referenced by multiple historical quotations and proposals.
- A vehicle's current inventory status must not determine the validity of historical finance records.
- Vehicle details used in a quotation or proposal must be preserved as a snapshot where necessary.

Release 1 vehicle statuses:

- Available
- Reserved
- Sold
- Archived

These statuses represent dealership inventory state, not lender or finance application state.

## 6.3 Lender

Represents an external finance provider that may receive proposals from a dealership.

Responsibilities:

- Identify the finance provider.
- Store reference information.
- Store relevant contact details.
- Store external portal information where available.

A lender is not a AssetFlow tenant customer in Release 1.

A lender does not receive a AssetFlow login or access to dealership records through the MVP.

The lender record represents an external counterparty, not a lender-side workflow participant.

## 6.4 Dealership-Lender Relationship

Represents a dealership's relationship with an external lender.

This is a many-to-many relationship:

- One dealership can work with multiple lenders.
- One lender can work with multiple dealerships.

Responsibilities:

- Indicate whether the dealership currently works with the lender.
- Record the products supported by that relationship.
- Store dealership-specific lender references.
- Store relevant portal URLs and contact information.
- Provide the context for selecting a lender when preparing a proposal.

The relationship belongs to a dealership organization.

A lender's availability to one dealership must not automatically make it available to another dealership.

Release 1 relationship statuses:

- Active
- Inactive

The relationship does not represent lender approval, credit eligibility, or a guaranteed finance offer.

## 6.5 Finance Calculation

Represents a calculation performed for a specific finance product using a defined set of financial inputs and rules.

For Release 1, this means a Hire Purchase calculation.

Responsibilities:

- Accept calculation inputs.
- Validate those inputs.
- Identify the product and rule versions used.
- Execute the applicable calculation.
- Store the resulting financial breakdown.
- Preserve the calculation inputs and outputs for reproducibility.

Possible inputs include:

- Vehicle price.
- Deposit.
- Amount financed.
- Finance term.
- Interest-related parameters.
- Fees and charges.
- Payment frequency.

Possible outputs include:

- Amount financed.
- Interest amount.
- Payment amount.
- Total payable.
- Financial breakdown.

The exact fields and formulas are defined in the financial rules and database design documents.

Important distinctions:

- A calculation is not a quotation.
- A calculation is not a proposal.
- A calculation does not represent lender approval.
- A calculation does not create a lender agreement.

A calculation may be created independently or associated with a quotation.

A saved calculation must retain the exact inputs, outputs, product version, financial rule version and calculation engine version used.

## 6.6 Quotation

Represents a customer-facing finance offer or illustration prepared by a dealership.

Responsibilities:

- Associate a customer with a vehicle.
- Reference the applicable finance product.
- Reference the calculation used.
- Preserve the financial terms presented to the customer.
- Support document generation.
- Track the dealership's quotation lifecycle.

A quotation is a business record, not simply a calculation result.

It may contain:

- Customer snapshot.
- Vehicle snapshot.
- Finance product.
- Financial terms.
- Validity period.
- Quotation reference.
- Generated document references.

Release 1 quotation statuses:

- Draft
- Issued
- Accepted
- Declined
- Expired
- Cancelled
- Superseded

Acceptance and decline are customer-facing dealership records. They do not represent lender approval or rejection.

Important rules:

- A quotation can be revised.
- A revision must preserve the previous version.
- An issued quotation must not be silently overwritten.
- A quotation may exist without a finance proposal.
- A proposal must reference the relevant quotation where the workflow requires one.

## 6.7 Finance Proposal

Represents a dealership-prepared finance application or proposal package intended for an external lender.

For Release 1, proposals support Hire Purchase only.

Responsibilities:

- Associate the proposal with a customer.
- Associate the proposal with a vehicle.
- Associate the proposal with the selected dealership-lender relationship.
- Reference the applicable finance product.
- Preserve the relevant financial calculation and quotation.
- Capture required application information.
- Reference supporting documents.
- Prepare a lender-facing application package.

A proposal is not a lender decision or a lender-generated agreement.

A proposal may contain:

- Customer information snapshot.
- Vehicle information snapshot.
- Finance terms.
- Selected lender.
- Application-specific information.
- Supporting document references.
- Quotation and calculation references.
- Proposal version.
- Generated package reference.

Release 1 proposal statuses:

- Draft
- In Progress
- Ready for Submission
- Submitted Externally
- Cancelled
- Superseded

Important rules:

- A proposal belongs to one dealership organization.
- A proposal is associated with one selected lender relationship at a time.
- A proposal must reference the applicable finance product.
- A proposal must preserve the information used to prepare it.
- A proposal must not be marked submitted merely because a package was downloaded.
- Submission must be explicitly recorded by an authorized dealership user.
- Revisions must preserve historical proposal and package versions.
- A superseded proposal must remain available for historical reference.

The platform does not track lender-side underwriting, approval, rejection or processing stages.

## 6.8 External Submission Record

Represents the dealership's record that a particular proposal package was submitted to an external lender.

This record is evidence of a dealership-confirmed action, not proof that the lender received or accepted the application.

Responsibilities:

- Identify the proposal and exact version submitted.
- Identify the selected lender relationship.
- Record the submitting dealership user.
- Record the submission timestamp.
- Optionally record an external reference number.
- Optionally record notes.

A submission must preserve the identity of the package version submitted.

Important rules:

- Downloading a package does not create a submission record.
- Generating a package does not create a submission record.
- A user must explicitly confirm that the external submission took place.
- The submission record must be immutable as a historical event.
- Corrections should be recorded as additional audit information rather than silently rewriting history.
- A later proposal revision must not change the package version referenced by an earlier submission.

AssetFlow does not automatically know whether the lender has received, reviewed, approved or rejected the application.

# 7. Documents Domain

This domain is owned by `AssetFlow-document`.

It manages uploaded and generated files, while financing records retain references to relevant documents.

## 7.1 Document

Represents a file stored and managed by AssetFlow.

Examples:

- Customer-provided identity documents.
- Vehicle documents.
- Proof of address.
- Supporting financial documents.
- Generated quotations.
- Generated application packages.

Responsibilities:

- Store file metadata.
- Track processing state.
- Maintain file versions.
- Associate files with the owning organization.
- Provide secure file access.
- Track document ownership and associations.

Important rules:

- Every document belongs to one organization.
- A dealership can only access its own documents.
- Backend authorization must be enforced for every document operation.
- File storage location must not be treated as authorization.
- Documents may be associated with customers, vehicles, quotations or proposals.
- Document metadata and file content must be protected independently.

## 7.2 Document Version

Represents a specific version of a document.

Responsibilities:

- Preserve the exact file content.
- Record upload or generation information.
- Support document history.
- Allow a proposal or quotation to reference the exact version used.

A document version must not be silently replaced when a newer version is uploaded.

## 7.3 Document Template

Represents a reusable document layout used to generate business documents.

Examples:

- Dealership quotation template.
- Finance illustration template.
- Proposal supporting-document template.

Responsibilities:

- Define the document structure.
- Define supported placeholders.
- Support organization-specific branding and content.
- Support product-specific document generation.

The document service owns template content and processing. Configuration owns the organization-level template assignment.

## 7.4 Generated Document

Represents a document generated by AssetFlow using a template and business data.

Examples:

- Customer quotation PDF.
- Finance application package.
- Supporting summary.

Responsibilities:

- Record the template and version used.
- Record the source business record and version.
- Preserve the generated file.
- Track generation status.
- Allow authorized users to download the result.

Generated documents must preserve the data and template versions used during generation.

## 7.5 Document Processing Job

Represents asynchronous work performed on a document.

Examples:

- File validation.
- PDF conversion.
- Placeholder replacement.
- Document generation.
- File verification.

Responsibilities:

- Track processing status.
- Record errors.
- Support retries where safe.
- Avoid duplicate processing where possible.
- Provide traceability for background operations.

A processing job is an operational record, not a replacement for the underlying document.

# 8. Domain Relationships

The following relationships describe the primary business model.

## 8.1 Organization and Identity

- Organization has many memberships.
- User has many memberships.
- Membership connects one user to one organization.
- Organization has its own settings and branches.

## 8.2 Organization and Financing

- Organization has many customers.
- Organization has many vehicles.
- Organization has many dealership-lender relationships.
- Organization has many calculations.
- Organization has many quotations.
- Organization has many proposals.
- Organization has many external submission records.

All these relationships are tenant-scoped.

## 8.3 Customer and Vehicle

- Customer can have many quotations.
- Customer can have many proposals.
- Vehicle can appear in many historical quotations.
- Vehicle can appear in many historical proposals.
- A quotation and proposal each reference the relevant customer and vehicle.

Do not enforce a permanent one-to-one relationship between a customer and vehicle.

## 8.4 Lender Relationships

- Lender can have many dealership-lender relationships.
- Organization can have many dealership-lender relationships.
- Each dealership-lender relationship connects one organization to one lender.
- A dealership-lender relationship can support multiple finance products.

## 8.5 Calculation and Quotation

- A calculation can be associated with a quotation.
- A quotation references the calculation used for its financial terms.
- A quotation may be revised into a new version.
- Historical quotation versions retain their original calculation references.

## 8.6 Quotation and Proposal

- A quotation may lead to one or more proposal versions over its lifecycle.
- A proposal references the relevant quotation when applicable.
- A proposal preserves its own customer, vehicle and financial snapshots.
- A quotation can exist without a proposal.

## 8.7 Proposal and Documents

- A proposal may reference multiple supporting documents.
- A proposal may have multiple generated application package versions.
- Each package version identifies the proposal version and source documents used.
- A submission record identifies the exact package version submitted.

## 8.8 Cross-Service Relationships

Relationships across services use stable identifiers.

Examples:

- Identity organization ID referenced by financing records.
- Financing proposal ID referenced by generated documents.
- Configuration product version ID referenced by calculations.
- Configuration template assignment ID used when selecting a document template.

No cross-service database foreign keys or direct database joins are permitted.

# 9. Entity Lifecycle Summary

The following statuses represent the initial business lifecycle.

| Entity | Release 1 lifecycle |
|---|---|
| Organization | Active / Suspended |
| Membership | Invited / Active / Suspended / Revoked |
| Customer | Active / Archived |
| Vehicle | Available / Reserved / Sold / Archived |
| Dealership-Lender Relationship | Active / Inactive |
| Calculation | Draft / Calculated / Saved / Superseded |
| Quotation | Draft / Issued / Accepted / Declined / Expired / Cancelled / Superseded |
| Proposal | Draft / In Progress / Ready for Submission / Submitted Externally / Cancelled / Superseded |
| Document | Uploaded / Processing / Available / Failed / Archived |
| Document Package | Pending / Processing / Ready / Failed / Superseded |
| Document Template | Draft / Active / Inactive |

Not every entity requires a user-visible lifecycle. Some statuses are operational and exist primarily to support processing and error handling.

# 10. Domain Invariants

These rules must always hold regardless of the API, frontend or database implementation.

## 10.1 Tenant Isolation

1. Every dealership business record must belong to an organization.
2. Users may access organization records only through valid membership and permissions.
3. One dealership must never access another dealership's private records.
4. Document access must enforce the same organization boundary.
5. Knowing another organization's record ID must not grant access.

## 10.2 Financial Integrity

1. Every calculation must identify the product and rule versions used.
2. Financial inputs and outputs must be preserved.
3. Historical quotations and proposals must not change silently when current product rules change.
4. Calculation results must be reproducible using their recorded inputs and versions.
5. A calculation must not be interpreted as a lender decision.
6. Financial rounding and precision rules must be consistent with the applicable financial rule set.

## 10.3 Proposal Integrity

1. Every proposal belongs to one dealership organization.
2. Every proposal identifies its selected lender relationship.
3. A proposal must preserve its relevant customer, vehicle and financial information.
4. A proposal version must be distinguishable from previous versions.
5. A generated package must identify its source proposal version.
6. An external submission must identify the exact package version submitted.
7. A proposal must not be marked externally submitted without an explicit dealership-confirmed submission action.

## 10.4 Document Integrity

1. Documents are organization-scoped.
2. Document versions preserve historical file content.
3. Generated documents preserve their source template and business-record versions.
4. A document download must be authorized.
5. Failed document processing must not produce a falsely successful document status.

## 10.5 External Lender Boundary

1. Lenders are external counterparties in Release 1.
2. AssetFlow does not create lender user accounts or lender-side workflows.
3. AssetFlow does not perform lender credit checks or underwriting.
4. AssetFlow does not record lender approval or rejection as a platform-controlled decision.
5. AssetFlow does not create lender agreements or collect lender repayments.
6. External lender activity is recorded only when confirmed and entered by an authorized dealership user.

# 11. Historical Data and Versioning

AssetFlow must distinguish current business information from historical information used in financial transactions.

The following records require historical preservation:

- Finance product versions.
- Financial rule versions.
- Calculation inputs and outputs.
- Quotation versions.
- Proposal versions.
- Generated document versions.
- Application package versions.
- External submission records.

For example, if a dealership changes a customer's address after submitting a proposal, the historical proposal must still show the address used when that proposal was prepared.

Similarly, if a finance rule changes, previously issued quotations and submitted proposals must retain their original financial results.

Versioning must be designed as a domain requirement, not added later as an afterthought.

# 12. Audit and Traceability

Important business operations must be attributable to an authenticated user or system process.

Examples:

- Customer created or updated.
- Vehicle created or updated.
- Lender relationship activated or deactivated.
- Calculation performed.
- Quotation issued or revised.
- Proposal prepared or revised.
- Supporting document uploaded.
- Application package generated.
- External submission recorded.
- Document archived.

Audit records should capture:

- Organization context.
- Actor or system identity.
- Action.
- Entity type and identifier.
- Timestamp.
- Relevant before-and-after information where appropriate.
- Correlation or request identifier.

Audit records must not expose passwords, tokens, or other secrets.

# 13. Explicitly Excluded Domain Concepts in Release 1

The following are not part of the Release 1 domain model:

- Lender user accounts and lender dashboards.
- Lender-side underwriting.
- Credit bureau checks.
- Lender approval and rejection workflows.
- Lender agreement generation and lifecycle management.
- Repayment schedules and payment collection.
- Payment reconciliation.
- Direct lender integrations or automated portal submissions.
- Automated lender decision tracking.
- Mandatory dealership internal approval workflows.
- Finance Lease transactions.
- Operating Lease transactions.
- Non-vehicle asset transactions.

These concepts may be considered in later releases, but they must not complicate the initial Hire Purchase implementation.

# 14. Future Extensibility

The model should support future expansion without implementing future features prematurely.

## Release 2 — Finance Lease

Introduce Finance Lease-specific:

- Product definition and rule sets.
- Financial calculation logic.
- Product-specific proposal fields.
- Document requirements.
- Lifecycle behavior where required.

## Release 3 — Operating Lease

Introduce Operating Lease-specific:

- Product definition and rule sets.
- Rental calculation logic.
- Residual value and related financial inputs where applicable.
- Product-specific proposal fields.
- Document requirements.

## Future Releases

Potential extensions:

- Additional asset categories.
- More finance products.
- Additional lender integrations.
- External service integrations.
- More advanced organization and branch management.
- Additional reporting and operational capabilities.

Future extensions must respect existing domain boundaries and preserve historical records.

# 15. Open Design Items for Later Documents

The following details are intentionally not finalized in this document:

1. Exact entity fields, data types, constraints and indexes — `04_DATABASE_DESIGN.md`.
2. Exact service APIs and cross-service communication — `05_SYSTEM_ARCHITECTURE.md` and `06_API_CONTRACTS.md`.
3. Exact frontend screens and user interactions — `07_FRONTEND_SPEC.md`.
4. UK Hire Purchase formulas, rounding, fees and financial assumptions — `08_FINANCIAL_RULES.md`.
5. Exact document types and required application fields for different lenders.
6. Exact retention, privacy and data-protection requirements.
7. Detailed permission matrix for dealership roles.
8. Exact audit-event schema and retention policy.

These items must be resolved in their appropriate specifications rather than guessed during implementation.

# 16. Domain Model Completion Criteria

This domain model is ready to inform database design when:

- Every Release 1 business concept has a clear definition.
- Each entity has an identified owning service.
- Organization ownership and tenant isolation are explicit.
- Cross-service relationships use identifiers rather than shared databases.
- Calculation, quotation, proposal and external submission are separate concepts.
- Lenders remain external counterparties.
- Historical versions and snapshots are accounted for.
- Document ownership and versioning are clear.
- Release 1 exclusions are explicit.
- Future products can be added without changing the meaning of existing historical records.

**End of Document**