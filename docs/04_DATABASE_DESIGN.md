# 04 — Database Design

**Project:** AssetFlow  
**Document:** Database Design  
**Status:** Proposed — subject to final schema review  
**Market:** United Kingdom  
**Initial release:** Vehicle Dealerships — Hire Purchase (HP)

---

# 1. Purpose

This document defines the database architecture, ownership boundaries, entities, relationships, constraints, and data integrity rules for AssetFlow.

The database design must support:

- Multiple dealership organizations.
- Organization-specific users, roles, and permissions.
- Organization-specific settings and finance product availability.
- Customer and vehicle management.
- Multiple lender relationships per dealership.
- Hire Purchase calculations.
- Quotations and proposals.
- External lender submission tracking.
- Document generation and package management.
- Historical data retention and soft deletion.
- Future Finance Lease and Operating Lease products.
- Future support for additional asset categories.

The design must be production-quality while remaining practical to develop and run locally.

# 2. Database Architecture

AssetFlow will use four independently owned PostgreSQL databases.

| Database | Owning service | Responsibility |
|---|---|---|
| `identity_db` | `AssetFlow-identity` | Users, organizations, memberships, roles, permissions and authentication |
| `configuration_db` | `AssetFlow-configuration` | Organization settings, finance products, product versions and configurable rules |
| `financing_db` | `AssetFlow-financing` | Customers, addresses, bank accounts, vehicles, lenders, calculations, quotations, proposals and submissions |
| `document_db` | `AssetFlow-document` | Documents, uploads, versions, templates, generated files and package references |

## 2.1 Local PostgreSQL setup

All four databases will run on one local PostgreSQL server during development.

```text
PostgreSQL
│
├── identity_db
├── configuration_db
├── financing_db
└── document_db
```

Separate PostgreSQL servers are not required for local development.

The logical database separation must remain intact.

## 2.2 Database ownership

Each service owns its database.

- A service may read and write its own database.
- A service must not directly query another service's database.
- Cross-service relationships use logical identifiers.
- Cross-service information is obtained through APIs or events.
- No cross-database foreign keys are permitted.
- Each service manages its own migrations.

For example, `financing_db` may store an `organization_id` referencing an organization in `identity_db`, but it must not create a foreign key to that database.

## 2.3 Transaction boundaries

Transactions must remain within their owning service.

Operations involving multiple services must use application-level workflows.

Use:

- Idempotent operations.
- Retry handling.
- Transactional outbox patterns where appropriate.
- Event-driven synchronization where useful.
- Compensating actions for multi-service operations.

Distributed database transactions are not part of the architecture.

---

# 3. Global Database Conventions

## 3.1 Naming conventions

- Table names: plural, `snake_case`.
- Column names: `snake_case`.
- Primary keys: `id`.
- Foreign keys: `<referenced_entity>_id`.
- Timestamps: UTC.
- Database: PostgreSQL.
- Primary keys: UUID.
- Monetary values: `NUMERIC`, never floating-point.
- Boolean fields: `BOOLEAN`.
- Dates: `DATE`.
- Timestamps: `TIMESTAMPTZ`.

Use SQLAlchemy models and Alembic migrations.

## 3.2 Standard audit fields

Applicable primary business entities should include:

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `created_at` | TIMESTAMPTZ | Creation timestamp |
| `created_by` | UUID, nullable | User responsible for creation |
| `updated_at` | TIMESTAMPTZ | Last update timestamp |
| `updated_by` | UUID, nullable | User responsible for last update |
| `is_active` | BOOLEAN | Whether the record is active |
| `is_deleted` | BOOLEAN | Whether the record has been soft-deleted |

Defaults:

```text
is_active = TRUE
is_deleted = FALSE
created_at = CURRENT_TIMESTAMP
updated_at = CURRENT_TIMESTAMP
```

These fields apply to primary business entities where appropriate, not automatically every supporting or transactional table.

## 3.3 `is_active`

Indicates whether a record is currently active or selectable in normal application workflows.

Examples:

- An inactive lender cannot be selected for a new proposal.
- An inactive finance product cannot be selected for new quotations.
- An inactive customer can be excluded from ordinary customer selection.
- An inactive user cannot authenticate.
- An inactive organization cannot operate normally.

Historical records must still display references to inactive entities.

Deactivation must not rewrite or delete historical records.

## 3.4 `is_deleted`

Represents soft deletion.

When `is_deleted = TRUE`:

- The record is excluded from ordinary UI listings and selection.
- The record remains in the database.
- Historical references remain intact.
- Audit history is retained.
- Physical deletion is not performed through ordinary application workflows.

Soft deletion must not bypass financial or historical retention requirements.

## 3.5 General data rules

- Store monetary values using appropriate decimal precision.
- Store currency explicitly using ISO 4217 codes, such as `GBP`.
- Use appropriate precision for interest rates and percentages.
- Normalize email addresses before uniqueness validation.
- Do not store passwords or raw API secrets.
- Use database constraints for important integrity rules.
- Add indexes for common filtering and lookup operations.
- Do not store complete financial business records in unstructured JSONB fields.

---

# 4. Identity Database

**Database:** `identity_db`  
**Service:** `AssetFlow-identity`

Owns user accounts, organizations, memberships, authentication, and authorization.

Roles and permissions belong here, not in the configuration database.

## 4.1 `users`

Stores individual user accounts.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `first_name` | VARCHAR(100) | Required |
| `last_name` | VARCHAR(100) | Required |
| `email` | VARCHAR(320) | Required |
| `normalized_email` | VARCHAR(320) | Required, globally unique |
| `password_hash` | TEXT | Nullable for supported passwordless accounts |
| `phone_number` | VARCHAR(30) | Nullable |
| `email_verified_at` | TIMESTAMPTZ | Nullable |
| `last_login_at` | TIMESTAMPTZ | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Email addresses are globally unique across user accounts.

Before storage:

- Trim whitespace.
- Normalize the email to lowercase.
- Store the normalized value.
- Enforce a unique constraint on `normalized_email`.

`XY@Company.com` and `xy@company.com` represent the same account.

A user can belong to multiple organizations through separate membership records.

## 4.2 `organizations`

Stores dealership organizations registered on AssetFlow.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `name` | VARCHAR(255) | Required |
| `legal_name` | VARCHAR(255) | Nullable |
| `registration_number` | VARCHAR(100) | Nullable |
| `vat_number` | VARCHAR(100) | Nullable |
| `contact_email` | VARCHAR(320) | Nullable |
| `contact_phone` | VARCHAR(30) | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

An organization represents a dealership tenant.

Branch management is excluded from Release 1.

## 4.3 `organization_members`

Connects users to organizations.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required, logical reference |
| `user_id` | UUID | Required, logical reference |
| `joined_at` | TIMESTAMPTZ | Required |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

A user may belong to multiple organizations.

A membership must be unique for an organization/user combination while it is active and not deleted.

Use a partial unique index where appropriate to support soft deletion.

## 4.4 `roles`

Stores organization-specific roles.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required, logical reference |
| `name` | VARCHAR(100) | Required |
| `code` | VARCHAR(100) | Required |
| `description` | TEXT | Nullable |
| `is_system_role` | BOOLEAN | Default FALSE |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Roles are scoped to an organization.

Examples:

- Organization Administrator
- Manager
- Finance Executive
- Sales Executive
- Read Only

A role code must be unique within its organization.

## 4.5 `permissions`

Stores available system permissions.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `code` | VARCHAR(150) | Required, unique |
| `name` | VARCHAR(150) | Required |
| `description` | TEXT | Nullable |
| `module` | VARCHAR(100) | Required |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |

Example permission codes:

```text
customers.view
customers.create
customers.update
customers.delete

vehicles.view
vehicles.create
vehicles.update
vehicles.delete

quotations.view
quotations.create
quotations.update
quotations.issue

proposals.view
proposals.create
proposals.update
proposals.submit

documents.view
documents.upload
documents.delete

organization.settings.view
organization.settings.update

users.view
users.manage

roles.view
roles.manage
```

Permissions are defined centrally and assigned to organization-specific roles.

## 4.6 `role_permissions`

Maps permissions to roles.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `role_id` | UUID | FK → `roles.id` |
| `permission_id` | UUID | FK → `permissions.id` |
| `created_at` | TIMESTAMPTZ | Required |

Unique constraint:

`(role_id, permission_id)`

## 4.7 `member_roles`

Maps organization members to their assigned roles.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_member_id` | UUID | FK → `organization_members.id` |
| `role_id` | UUID | FK → `roles.id` |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |

Constraints:

- Unique `(organization_member_id, role_id)`.
- The assigned role must belong to the same organization as the membership.
- A member can have multiple roles if authorized.

## 4.8 `refresh_tokens`

Stores refresh-token metadata.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `user_id` | UUID | FK → `users.id` |
| `token_hash` | TEXT | Required |
| `expires_at` | TIMESTAMPTZ | Required |
| `revoked_at` | TIMESTAMPTZ | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `ip_address` | INET | Nullable |
| `user_agent` | TEXT | Nullable |

Only a hash of the refresh token is stored.

## 4.9 `api_keys`

Stores API key metadata.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Logical reference |
| `name` | VARCHAR(150) | Required |
| `key_prefix` | VARCHAR(30) | Required |
| `key_hash` | TEXT | Required |
| `scopes` | JSONB | Required |
| `expires_at` | TIMESTAMPTZ | Nullable |
| `last_used_at` | TIMESTAMPTZ | Nullable |
| `revoked_at` | TIMESTAMPTZ | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

The raw API key is displayed once at creation.

Only its cryptographic hash is stored.

## 4.10 Additional identity tables

| Table | Purpose |
|---|---|
| `password_reset_tokens` | Password reset token metadata and expiry |
| `email_verification_tokens` | Email verification |
| `otp_challenges` | One-time password challenges |
| `user_sessions` | Session/device tracking and revocation |
| `mfa_methods` | Multi-factor authentication methods |
| `login_attempts` | Authentication attempt tracking |
| `security_audit_logs` | Security-sensitive actions |

Authentication secrets must not be stored in plaintext.

---

# 5. Configuration Database

**Database:** `configuration_db`  
**Service:** `AssetFlow-configuration`

Owns finance product definitions, product versions, organization settings, and configurable business rules.

It does not own users, roles, or permissions.

## 5.1 `finance_products`

Stores the stable identity of each finance product.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `code` | VARCHAR(50) | Required, unique |
| `name` | VARCHAR(150) | Required |
| `description` | TEXT | Nullable |
| `display_order` | INTEGER | Default 0 |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Initial products:

| Code | Product name | Release |
|---|---|---|
| `HP` | Hire Purchase | Release 1 |
| `FL` | Finance Lease | Release 2 |
| `OL` | Operating Lease | Release 3 |

AssetFlow is one platform. These are finance products offered through it, not separate software products.

## 5.2 `finance_product_versions`

Stores versioned finance product definitions.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `finance_product_id` | UUID | FK → `finance_products.id` |
| `version_number` | INTEGER | Required |
| `name` | VARCHAR(150) | Required |
| `description` | TEXT | Nullable |
| `effective_from` | DATE | Required |
| `effective_to` | DATE | Nullable |
| `definition` | JSONB | Configurable product metadata |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Unique `(finance_product_id, version_number)`.

Published product versions must not be silently overwritten.

## 5.3 `organization_finance_products`

Controls which finance products an organization can use.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Logical reference to identity database |
| `finance_product_id` | UUID | FK → `finance_products.id` |
| `enabled_from` | DATE | Nullable |
| `enabled_to` | DATE | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Unique active organization/product combination.

Release 1 enables HP only.

## 5.4 `organization_settings`

Stores organization-specific platform settings.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Logical reference |
| `setting_key` | VARCHAR(150) | Required |
| `setting_value` | JSONB | Required |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Unique active `(organization_id, setting_key)`.

Examples:

```text
timezone
date_format
currency
default_locale
default_document_template
```

This table must not replace proper relational business entities.

## 5.5 `organization_profiles`

Stores organization-specific branding and presentation information.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Logical reference |
| `display_name` | VARCHAR(255) | Required |
| `legal_name` | VARCHAR(255) | Nullable |
| `logo_document_id` | UUID | Logical reference to documents |
| `primary_color` | VARCHAR(20) | Nullable |
| `secondary_color` | VARCHAR(20) | Nullable |
| `website` | VARCHAR(255) | Nullable |
| `contact_email` | VARCHAR(320) | Nullable |
| `contact_phone` | VARCHAR(30) | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

## 5.6 `product_rules`

Stores configurable business and financial rules associated with a finance product version.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `finance_product_version_id` | UUID | FK → `finance_product_versions.id` |
| `rule_code` | VARCHAR(100) | Required |
| `name` | VARCHAR(150) | Required |
| `description` | TEXT | Nullable |
| `rule_type` | VARCHAR(50) | Required |
| `rule_definition` | JSONB | Required |
| `effective_from` | DATE | Required |
| `effective_to` | DATE | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Rule definitions may use JSONB for genuinely configurable logic and parameters.

Transaction-specific financial values must remain relational.

## 5.7 `asset_categories`

Defines supported asset categories.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `code` | VARCHAR(50) | Required, unique |
| `name` | VARCHAR(150) | Required |
| `description` | TEXT | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Initial category:

```text
VEHICLE
```

## 5.8 `asset_field_definitions`

Defines configurable fields for asset categories.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `asset_category_id` | UUID | FK → `asset_categories.id` |
| `field_key` | VARCHAR(100) | Required |
| `field_label` | VARCHAR(150) | Required |
| `data_type` | VARCHAR(50) | Required |
| `is_required` | BOOLEAN | Default FALSE |
| `validation_rules` | JSONB | Nullable |
| `display_order` | INTEGER | Default 0 |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Release 1 vehicle information must still use dedicated relational columns in Financing.

## 5.9 Additional configuration tables

| Table | Purpose |
|---|---|
| `document_template_assignments` | Maps organizations and document types to template references |
| `notification_settings` | Organization notification configuration |
| `integration_settings` | Organization integration configuration |
| `configuration_audit_logs` | Configuration change history |

Secrets must be encrypted or managed through secure secret storage.

---

# 6. Financing Database

**Database:** `financing_db`  
**Service:** `AssetFlow-financing`

This database owns the primary business domain:

- Customers and their reusable information.
- Customer addresses.
- Customer bank accounts.
- Vehicles and assets.
- Lenders and dealership lender relationships.
- Calculations.
- Quotations and quotation versions.
- Proposals and proposal versions.
- External submissions.
- Historical transaction snapshots.
- Business audit history.

## 6.1 Customer data model — important design principle

A customer is a single master entity.

A customer's address and bank accounts are maintained in their respective customer tables, regardless of whether the customer is being used in a quotation or proposal.

We do not create separate reusable customer or address records for quotations and proposals.

Quotations and proposals reference the same customer records.

Historical transaction versions preserve the values actually used when required.

This separates:

1. **Master data:** Current reusable customer information.
2. **Transaction references:** Which customer and related records were used.
3. **Historical snapshots:** The exact information included in a quotation or submitted proposal.

## 6.2 `customers`

Stores dealership customers.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required, logical reference |
| `customer_type` | VARCHAR(30) | Required: Individual / Business |
| `first_name` | VARCHAR(100) | Nullable |
| `last_name` | VARCHAR(100) | Nullable |
| `business_name` | VARCHAR(255) | Nullable |
| `date_of_birth` | DATE | Nullable |
| `email` | VARCHAR(320) | Nullable |
| `phone_number` | VARCHAR(30) | Nullable |
| `nationality` | VARCHAR(100) | Nullable |
| `company_registration_number` | VARCHAR(100) | Nullable |
| `vat_number` | VARCHAR(100) | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `updated_at` | TIMESTAMPTZ | Required |
| `updated_by` | UUID | Nullable |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Rules:

- Every customer belongs to one dealership organization.
- A customer may have multiple addresses.
- A customer may have multiple bank accounts.
- A customer may participate in multiple quotations and proposals.
- Customer records are not shared across organizations by default.
- Customer type determines applicable validation requirements.

## 6.3 `customer_addresses`

Stores reusable customer addresses.

There is one address table for all customer addresses. It is not separated by quotation or proposal.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `customer_id` | UUID | FK → `customers.id` |
| `address_type` | VARCHAR(30) | Required |
| `address_line_1` | VARCHAR(255) | Required |
| `address_line_2` | VARCHAR(255) | Nullable |
| `city` | VARCHAR(100) | Required |
| `county` | VARCHAR(100) | Nullable |
| `postcode` | VARCHAR(20) | Required |
| `country_code` | CHAR(2) | Required |
| `is_primary` | BOOLEAN | Default FALSE |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

A customer can have:

- Current residential address.
- Previous residential address.
- Business address.
- Correspondence address.

The same address record can be referenced by different transactions.

An address must belong to the same customer referenced by the quotation or proposal using it.

## 6.4 `customer_bank_accounts`

Stores reusable customer bank account information.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `customer_id` | UUID | FK → `customers.id` |
| `account_holder_name` | VARCHAR(255) | Required |
| `bank_name` | VARCHAR(150) | Nullable |
| `sort_code` | VARCHAR(20) | Nullable |
| `account_number` | VARCHAR(50) | Nullable |
| `iban` | VARCHAR(50) | Nullable |
| `bic` | VARCHAR(20) | Nullable |
| `is_primary` | BOOLEAN | Default FALSE |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Bank details are sensitive information.

They must be encrypted at rest where appropriate and excluded from ordinary logs and API responses.

## 6.5 `assets`

Stores dealership assets.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required, logical reference |
| `asset_category_id` | UUID | Logical reference to configuration |
| `asset_type` | VARCHAR(50) | Required |
| `asset_reference` | VARCHAR(100) | Nullable |
| `description` | TEXT | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `updated_at` | TIMESTAMPTZ | Required |
| `updated_by` | UUID | Nullable |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Release 1 supports vehicles.

## 6.6 `asset_vehicles`

Stores vehicle-specific information.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `asset_id` | UUID | FK → `assets.id`, unique |
| `registration_number` | VARCHAR(30) | Nullable |
| `vin` | VARCHAR(50) | Nullable |
| `make` | VARCHAR(100) | Required |
| `model` | VARCHAR(100) | Required |
| `derivative` | VARCHAR(150) | Nullable |
| `year_of_manufacture` | SMALLINT | Nullable |
| `first_registration_date` | DATE | Nullable |
| `fuel_type` | VARCHAR(50) | Nullable |
| `transmission` | VARCHAR(50) | Nullable |
| `body_type` | VARCHAR(50) | Nullable |
| `engine_capacity_cc` | INTEGER | Nullable |
| `colour` | VARCHAR(50) | Nullable |
| `mileage` | INTEGER | Nullable |
| `vehicle_condition` | VARCHAR(30) | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

A vehicle may be referenced by multiple historical quotations and proposals.

Do not enforce a one-to-one relationship between a vehicle and a quotation or proposal.

## 6.7 `lenders`

Stores external lender reference information.

Lenders are counterparties, not AssetFlow tenant organizations.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `name` | VARCHAR(255) | Required |
| `legal_name` | VARCHAR(255) | Nullable |
| `registration_number` | VARCHAR(100) | Nullable |
| `website` | VARCHAR(255) | Nullable |
| `contact_email` | VARCHAR(320) | Nullable |
| `contact_phone` | VARCHAR(30) | Nullable |
| `head_office_address` | TEXT | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

## 6.8 `organization_lenders`

Maps dealerships to their lender relationships.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Logical reference |
| `lender_id` | UUID | FK → `lenders.id` |
| `relationship_name` | VARCHAR(150) | Nullable |
| `portal_url` | VARCHAR(500) | Nullable |
| `external_dealer_reference` | VARCHAR(150) | Nullable |
| `contact_name` | VARCHAR(150) | Nullable |
| `contact_email` | VARCHAR(320) | Nullable |
| `contact_phone` | VARCHAR(30) | Nullable |
| `notes` | TEXT | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

One dealership can work with multiple lenders.

One lender can work with multiple dealerships.

## 6.9 `organization_lender_products`

Defines the finance products supported by a dealership's lender relationship.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_lender_id` | UUID | FK → `organization_lenders.id` |
| `finance_product_id` | UUID | Logical reference to configuration |
| `external_product_reference` | VARCHAR(150) | Nullable |
| `notes` | TEXT | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

A lender relationship may support multiple finance products.

A finance product may be supported by multiple lenders.

## 6.10 `calculations`

Stores financial calculation records.

A calculation is distinct from a quotation and proposal.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required |
| `customer_id` | UUID | Nullable, FK → `customers.id` |
| `asset_id` | UUID | Nullable, FK → `assets.id` |
| `finance_product_id` | UUID | Logical reference |
| `finance_product_version_id` | UUID | Logical reference |
| `calculation_reference` | VARCHAR(100) | Required |
| `status` | VARCHAR(30) | Required |
| `currency_code` | CHAR(3) | Required |
| `calculation_engine_version` | VARCHAR(50) | Required |
| `calculated_at` | TIMESTAMPTZ | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Suggested statuses:

```text
DRAFT
CALCULATED
SAVED
SUPERSEDED
```

## 6.11 `calculation_versions`

Stores calculation inputs and results for reproducibility.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `calculation_id` | UUID | FK → `calculations.id` |
| `version_number` | INTEGER | Required |
| `finance_product_version_id` | UUID | Logical reference |
| `calculation_engine_version` | VARCHAR(50) | Required |
| `currency_code` | CHAR(3) | Required |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `calculated_at` | TIMESTAMPTZ | Nullable |

Unique `(calculation_id, version_number)`.

The exact HP input and output fields will be defined in `08_FINANCIAL_RULES.md`.

Financial inputs and outputs must use relational columns and appropriate related tables.

Recalculation creates a new version rather than overwriting historical results.

## 6.12 `calculation_applicants`

Stores the applicant information used by a calculation version.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `calculation_version_id` | UUID | FK → `calculation_versions.id` |
| `customer_id` | UUID | Nullable, FK → `customers.id` |
| `applicant_type` | VARCHAR(30) | Required |
| `created_at` | TIMESTAMPTZ | Required |

The `customer_id` references the existing customer master record.

A co-applicant may reference another customer record or be represented by applicant information collected for the transaction where no customer master record exists.

Do not create duplicate reusable customer records simply because someone participates in a calculation.

The exact calculation inputs required from applicants will be defined in the financial specification.

## 6.13 `calculation_assets`

Stores the asset references used by a calculation version.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `calculation_version_id` | UUID | FK → `calculation_versions.id` |
| `asset_id` | UUID | Nullable, FK → `assets.id` |
| `asset_type` | VARCHAR(50) | Required |
| `created_at` | TIMESTAMPTZ | Required |

The asset reference identifies the master asset.

Any calculation-specific vehicle inputs must be preserved in the calculation version.

## 6.14 `quotations`

Stores quotation identities.

A quotation is a business record that may have multiple versions.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required |
| `quotation_number` | VARCHAR(100) | Required |
| `customer_id` | UUID | FK → `customers.id` |
| `asset_id` | UUID | FK → `assets.id` |
| `finance_product_id` | UUID | Logical reference |
| `current_version_id` | UUID | Nullable logical reference |
| `status` | VARCHAR(30) | Required |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `updated_at` | TIMESTAMPTZ | Required |
| `updated_by` | UUID | Nullable |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Suggested statuses:

```text
DRAFT
ISSUED
ACCEPTED
DECLINED
EXPIRED
CANCELLED
SUPERSEDED
```

Quotation acceptance or decline represents a dealership-recorded customer response, not a lender decision.

## 6.15 `quotation_versions`

Stores quotation version history and transaction-specific information.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `quotation_id` | UUID | FK → `quotations.id` |
| `version_number` | INTEGER | Required |
| `calculation_version_id` | UUID | Nullable logical reference |
| `finance_product_version_id` | UUID | Logical reference |
| `currency_code` | CHAR(3) | Required |
| `valid_until` | DATE | Nullable |
| `issued_at` | TIMESTAMPTZ | Nullable |
| `accepted_at` | TIMESTAMPTZ | Nullable |
| `declined_at` | TIMESTAMPTZ | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Unique `(quotation_id, version_number)`.

A quotation version references the customer and asset through the parent quotation.

It does not create another customer or reusable address.

### Historical information

When a quotation version is issued, the transaction must preserve the relevant values used to produce that quotation.

Where a customer or vehicle value can change later, the quotation version must retain its original value.

The exact snapshot structure must be finalized against the quotation's required fields.

An issued quotation may be revised according to its lifecycle.

Once a quotation version is used in a submitted external proposal, its referenced information must remain preserved.

## 6.16 `quotation_version_applicants`

Associates applicants with a specific quotation version.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `quotation_version_id` | UUID | FK → `quotation_versions.id` |
| `customer_id` | UUID | Nullable, FK → `customers.id` |
| `applicant_type` | VARCHAR(30) | Required |
| `created_at` | TIMESTAMPTZ | Required |

This table represents the role a customer plays in a particular quotation version.

Examples:

- Primary applicant.
- Co-applicant.

It does not represent a separate customer record.

For a co-applicant who is not already a dealership customer, the quotation version must retain the applicant information actually used. This can be handled through a transaction-specific applicant snapshot associated with this row.

### Address selection

An applicant's reusable addresses remain in `customer_addresses`.

The selected address can be referenced by the transaction applicant through an `address_id` logical relationship within the same database.

The selected address must belong to the same customer referenced by the applicant.

For historical accuracy, the relevant address values must be preserved with the quotation version when it is issued. This is transaction snapshot data, not a second reusable address record.

## 6.17 `quotation_version_assets`

Associates the asset used by a quotation version and preserves relevant transaction-specific vehicle information.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `quotation_version_id` | UUID | FK → `quotation_versions.id` |
| `asset_id` | UUID | FK → `assets.id` |
| `created_at` | TIMESTAMPTZ | Required |

The asset's reusable details remain in `assets` and `asset_vehicles`.

The quotation version must preserve relevant vehicle values where later master-data changes could alter the historical quotation.

## 6.18 `proposals`

Stores proposal identities.

A proposal represents the dealership's application package prepared for submission to an external lender.

It is not a lender-generated agreement.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required |
| `proposal_number` | VARCHAR(100) | Required |
| `quotation_id` | UUID | Nullable, FK → `quotations.id` |
| `customer_id` | UUID | FK → `customers.id` |
| `asset_id` | UUID | FK → `assets.id` |
| `finance_product_id` | UUID | Logical reference |
| `current_version_id` | UUID | Nullable logical reference |
| `status` | VARCHAR(40) | Required |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `updated_at` | TIMESTAMPTZ | Required |
| `updated_by` | UUID | Nullable |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Suggested statuses:

```text
DRAFT
IN_PROGRESS
READY_FOR_SUBMISSION
SUBMITTED_EXTERNALLY
CANCELLED
SUPERSEDED
```

There are no lender underwriting, approval, rejection, or agreement-creation statuses.

## 6.19 `proposal_versions`

Stores proposal version history and the exact information prepared for submission.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `proposal_id` | UUID | FK → `proposals.id` |
| `version_number` | INTEGER | Required |
| `quotation_version_id` | UUID | Nullable, FK → `quotation_versions.id` |
| `calculation_version_id` | UUID | Nullable, logical reference |
| `finance_product_version_id` | UUID | Logical reference |
| `organization_lender_id` | UUID | Nullable, FK → `organization_lenders.id` |
| `currency_code` | CHAR(3) | Required |
| `prepared_at` | TIMESTAMPTZ | Nullable |
| `submitted_at` | TIMESTAMPTZ | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Unique `(proposal_id, version_number)`.

A submitted proposal version cannot be edited or deleted through ordinary application workflows.

Changes require a new version.

## 6.20 `proposal_version_applicants`

Associates applicants with a proposal version.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `proposal_version_id` | UUID | FK → `proposal_versions.id` |
| `customer_id` | UUID | Nullable, FK → `customers.id` |
| `applicant_type` | VARCHAR(30) | Required |
| `created_at` | TIMESTAMPTZ | Required |

The customer is referenced from the existing `customers` table.

The applicant type identifies the person's role in this proposal.

A person who is not an existing customer may be captured as a proposal-specific applicant without creating an unnecessary dealership customer record.

### Historical applicant snapshot

The proposal version must preserve the exact applicant information submitted to the lender.

The snapshot should contain explicit relational fields for the applicable information, such as:

- First name.
- Last name.
- Date of birth.
- Email.
- Phone number.
- Business name.
- Company registration number.
- Other required lender application fields.

The snapshot is historical transaction information. It is not a second customer master record.

## 6.21 Applicant address selection and snapshots

The customer's reusable addresses remain exclusively in `customer_addresses`.

A proposal applicant may reference the selected address record.

The selected address must belong to the referenced customer.

For a submitted proposal, the exact address values sent to the lender must be preserved as part of the proposal version's historical snapshot.

Do not create another reusable address record just because the address is used in a proposal.

If multiple addresses or address history are required for an application, those requirements must be represented explicitly in the transaction's applicant/address snapshot model.

## 6.22 Applicant bank account selection and snapshots

Reusable bank accounts remain in `customer_bank_accounts`.

A proposal applicant can reference the selected customer bank account.

The selected bank account must belong to the referenced customer.

When the proposal is submitted, preserve the exact account details included in the application.

Sensitive information must be encrypted and protected from ordinary logging.

## 6.23 `proposal_version_assets`

Associates the vehicle or asset with a proposal version.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `proposal_version_id` | UUID | FK → `proposal_versions.id` |
| `asset_id` | UUID | FK → `assets.id` |
| `created_at` | TIMESTAMPTZ | Required |

The master vehicle record remains in `assets` and `asset_vehicles`.

The proposal version must preserve the exact vehicle details included in the application, including any relevant values that may have changed since the master record was created.

These values belong to the historical transaction snapshot, not another reusable vehicle record.

## 6.24 Proposal financial data

Proposal financial information must use explicit relational columns and, where appropriate, dedicated related tables.

Do not store the entire proposal's financial data in one JSONB field.

The detailed HP schema must include applicable fields such as:

- Vehicle cash price.
- Deposit.
- Part exchange allowance, where applicable.
- Amount financed.
- Interest rate.
- Term.
- Payment frequency.
- Instalment amount.
- Fees and charges.
- Optional final payment or balloon payment, where applicable.
- Total payable.
- Total interest.
- Tax/VAT treatment, where applicable.
- Other required lender-specific HP fields.

The exact field definitions, calculation methodology, rounding and precision belong in `08_FINANCIAL_RULES.md`.

All values used for submission must be preserved in the proposal version.

## 6.25 `proposal_contacts`

Stores proposal-specific points of contact when required by the application.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `proposal_version_id` | UUID | FK → `proposal_versions.id` |
| `contact_type` | VARCHAR(50) | Required |
| `first_name` | VARCHAR(100) | Nullable |
| `last_name` | VARCHAR(100) | Nullable |
| `email` | VARCHAR(320) | Nullable |
| `phone_number` | VARCHAR(30) | Nullable |
| `job_title` | VARCHAR(100) | Nullable |
| `created_at` | TIMESTAMPTZ | Required |

A point of contact is not automatically a finance applicant or customer.

If the point of contact is also a customer, the appropriate customer relationship can be recorded.

## 6.26 `external_submissions`

Records dealership-confirmed submissions to external lenders.

AssetFlow does not directly submit applications to lender systems in Release 1.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required |
| `proposal_id` | UUID | FK → `proposals.id` |
| `proposal_version_id` | UUID | FK → `proposal_versions.id` |
| `organization_lender_id` | UUID | FK → `organization_lenders.id` |
| `proposal_package_reference_id` | UUID | Logical reference to document database |
| `external_reference` | VARCHAR(150) | Nullable |
| `submission_channel` | VARCHAR(50) | Required |
| `submitted_at` | TIMESTAMPTZ | Required |
| `submitted_by` | UUID | Required |
| `notes` | TEXT | Nullable |
| `created_at` | TIMESTAMPTZ | Required |

Suggested submission channels:

```text
LENDER_PORTAL
EMAIL
OTHER
```

Rules:

- One submission record references one exact proposal version.
- One submission record references one registered proposal package.
- The package must belong to the same proposal version.
- The selected lender relationship must match the submitted proposal version.
- The actor must belong to the organization and have the required permission.
- Downloading a package does not constitute submission.
- Submission is recorded only after dealership confirmation.

Multiple submissions may be recorded when the business workflow requires them.

### Package reference integrity

`proposal_package_reference_id` is a logical cross-service reference because `proposal_package_references` belongs to `document_db`.

It cannot have a conventional PostgreSQL foreign key across databases.

The Documents service must validate that the package exists and belongs to the requested proposal version before submission is recorded.

The Financing service must verify the validated response.

## 6.27 `business_audit_logs`

Stores important business actions.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required |
| `entity_type` | VARCHAR(100) | Required |
| `entity_id` | UUID | Required |
| `action` | VARCHAR(100) | Required |
| `performed_by` | UUID | Nullable |
| `performed_at` | TIMESTAMPTZ | Required |
| `description` | TEXT | Nullable |
| `metadata` | JSONB | Nullable |

Examples:

- Customer created.
- Customer address updated.
- Vehicle updated.
- Quotation issued.
- Proposal version created.
- Proposal package generated.
- External submission recorded.
- Proposal cancelled.

Audit records are append-only.

Sensitive personal and financial information must not be copied into audit metadata unnecessarily.

---

# 7. Documents Database

**Database:** `document_db`  
**Service:** `AssetFlow-document`

Owns document files, metadata, templates, versions, generated documents and proposal package references.

Documents uploaded by one dealership must not be accessible to another dealership unless an explicitly authorized sharing mechanism is introduced in a future release.

## 7.1 `documents`

Stores document metadata.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Logical reference |
| `document_type` | VARCHAR(100) | Required |
| `original_file_name` | VARCHAR(255) | Required |
| `storage_key` | TEXT | Required |
| `mime_type` | VARCHAR(150) | Required |
| `file_extension` | VARCHAR(20) | Nullable |
| `file_size_bytes` | BIGINT | Required |
| `checksum` | VARCHAR(128) | Required |
| `status` | VARCHAR(30) | Required |
| `uploaded_by` | UUID | Nullable |
| `uploaded_at` | TIMESTAMPTZ | Required |
| `created_at` | TIMESTAMPTZ | Required |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Suggested statuses:

```text
UPLOADED
PROCESSING
AVAILABLE
FAILED
ARCHIVED
```

Storage:

- Local development: MinIO.
- Production: Amazon S3.

The database stores metadata and storage keys, not file binaries.

## 7.2 `document_versions`

Stores document version history.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `document_id` | UUID | FK → `documents.id` |
| `version_number` | INTEGER | Required |
| `storage_key` | TEXT | Required |
| `checksum` | VARCHAR(128) | Required |
| `file_size_bytes` | BIGINT | Required |
| `mime_type` | VARCHAR(150) | Required |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |

Unique `(document_id, version_number)`.

Existing versions must not be silently overwritten.

## 7.3 `document_templates`

Stores document templates owned by a dealership or provided as platform defaults.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Nullable for platform templates |
| `template_name` | VARCHAR(255) | Required |
| `template_code` | VARCHAR(100) | Required |
| `document_type` | VARCHAR(100) | Required |
| `finance_product_id` | UUID | Nullable logical reference |
| `description` | TEXT | Nullable |
| `status` | VARCHAR(30) | Required |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `updated_at` | TIMESTAMPTZ | Required |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Suggested statuses:

```text
DRAFT
ACTIVE
INACTIVE
```

A dealership cannot modify another dealership's templates.

## 7.4 `document_template_versions`

Stores versioned template files.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `document_template_id` | UUID | FK → `document_templates.id` |
| `version_number` | INTEGER | Required |
| `document_id` | UUID | FK → `documents.id` |
| `placeholder_schema` | JSONB | Required |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `is_active` | BOOLEAN | Default TRUE |

Template versions must be preserved so historical generated documents can be traced to the exact template used.

## 7.5 `generated_documents`

Tracks documents generated from business data.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required |
| `document_template_version_id` | UUID | FK → `document_template_versions.id` |
| `document_id` | UUID | FK → `documents.id` |
| `source_entity_type` | VARCHAR(100) | Required |
| `source_entity_id` | UUID | Required |
| `source_version_id` | UUID | Required |
| `generation_status` | VARCHAR(30) | Required |
| `generated_at` | TIMESTAMPTZ | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |

Suggested statuses:

```text
PENDING
PROCESSING
READY
FAILED
```

For proposal documents, the source entity and source version identify the exact proposal version.

Financing owns proposal data.

Documents receives a validated generation payload and generates the document.

## 7.6 `proposal_package_references`

Stores references to generated proposal application packages.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required |
| `proposal_id` | UUID | Logical reference |
| `proposal_version_id` | UUID | Logical reference |
| `package_reference` | VARCHAR(150) | Required, unique |
| `package_document_id` | UUID | FK → `documents.id` |
| `package_version` | INTEGER | Required |
| `status` | VARCHAR(30) | Required |
| `generated_at` | TIMESTAMPTZ | Nullable |
| `created_at` | TIMESTAMPTZ | Required |
| `created_by` | UUID | Nullable |
| `is_active` | BOOLEAN | Default TRUE |
| `is_deleted` | BOOLEAN | Default FALSE |

Suggested statuses:

```text
PENDING
PROCESSING
READY
FAILED
SUPERSEDED
```

A package reference identifies one exact proposal version and its generated package.

A package cannot be treated as ready until generation succeeds.

Historical package references must remain available.

## 7.7 `document_processing_jobs`

Tracks asynchronous document operations.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required |
| `document_id` | UUID | Nullable, FK → `documents.id` |
| `job_type` | VARCHAR(100) | Required |
| `status` | VARCHAR(30) | Required |
| `attempt_count` | INTEGER | Default 0 |
| `started_at` | TIMESTAMPTZ | Nullable |
| `completed_at` | TIMESTAMPTZ | Nullable |
| `error_message` | TEXT | Nullable |
| `created_at` | TIMESTAMPTZ | Required |

Use idempotency keys and retry policies for asynchronous processing.

## 7.8 `document_access_logs`

Tracks important document access events.

| Column | Type | Constraints / Description |
|---|---|---|
| `id` | UUID | PK |
| `organization_id` | UUID | Required |
| `document_id` | UUID | FK → `documents.id` |
| `user_id` | UUID | Nullable |
| `action` | VARCHAR(50) | Required |
| `performed_at` | TIMESTAMPTZ | Required |
| `ip_address` | INET | Nullable |

Examples:

- Upload.
- Download.
- Preview.
- Archive.
- Delete.

---

# 8. Entity Relationships

## 8.1 Identity relationships

```text
organizations
    |
    └── organization_members
            |
            ├── users
            |
            └── member_roles
                    |
                    └── roles
                            |
                            └── role_permissions
                                    |
                                    └── permissions
```

## 8.2 Customer relationships

```text
customers
    |
    ├── customer_addresses
    |
    ├── customer_bank_accounts
    |
    ├── quotations
    |
    ├── proposals
    |
    └── calculation_applicants
```

A customer has one master record.

The customer's addresses and bank accounts are maintained once.

Transactions reference those records rather than creating separate reusable copies.

## 8.3 Asset relationships

```text
assets
    |
    ├── asset_vehicles
    |
    ├── calculations
    |
    ├── quotations
    |
    └── proposals
```

A vehicle may be referenced by multiple transactions.

## 8.4 Quotation and proposal relationships

```text
customers
    |
    └── quotations
            |
            └── quotation_versions
                    |
                    └── quotation_version_applicants
                            |
                            └── customer reference

quotations
    |
    └── proposals
            |
            └── proposal_versions
                    |
                    ├── proposal_version_applicants
                    │       |
                    │       └── customer reference
                    |
                    ├── proposal_version_assets
                    |
                    └── external_submissions
                            |
                            └── proposal_package_reference
```

The customer master record is never duplicated simply because it participates in multiple transactions.

Version-specific snapshots preserve historical values when needed.

---

# 9. Cross-Service Relationships

Because databases are independent, cross-service relationships use logical identifiers.

| Entity | Owning database | Referenced by |
|---|---|---|
| Organization | Identity | Configuration, Financing, Documents |
| User | Identity | Services requiring actor attribution |
| Membership | Identity | Authorization checks |
| Role / Permission | Identity | Authorization checks |
| Finance product | Configuration | Financing |
| Finance product version | Configuration | Financing |
| Asset category | Configuration | Financing |
| Organization finance product | Configuration | Financing workflows |
| Proposal | Financing | Documents |
| Proposal version | Financing | Documents |
| Generated document | Documents | Financing references |
| Proposal package reference | Documents | Financing submissions |

## 9.1 Cross-service validation

A logical reference must be validated by its owning service.

Examples:

- Financing validates finance product references with Configuration.
- Documents validates organization access through trusted authorization context.
- Financing validates proposal package references with Documents.
- Identity validates membership and permissions before an operation is authorized.

Do not create local shadow copies of another service's database merely to simulate foreign keys.

## 9.2 Avoid unnecessary synchronous calls

Not every request needs all four services.

| Operation | Required service data |
|---|---|
| Login | Identity |
| Customer listing | Financing |
| Organization settings | Configuration |
| Finance product selection | Configuration |
| HP calculation | Financing + relevant Configuration rules |
| Proposal creation | Financing + relevant Configuration data |
| Document upload | Documents |
| Document download | Documents + authorization context |
| Proposal details | Financing; Documents only when document information is needed |
| Submission recording | Financing + validated package reference from Documents |

The API gateway or application composition layer can combine responses when a screen needs information from multiple services.

---

# 10. Tenant Isolation and Data Security

Every dealership-owned business record must be scoped by `organization_id`.

This includes:

- Customers.
- Addresses through their customer ownership.
- Bank accounts through their customer ownership.
- Assets.
- Lender relationships.
- Calculations.
- Quotations.
- Proposals.
- Submissions.
- Documents.
- Templates.
- Package references.

## 10.1 Tenant isolation rules

- Never trust an organization ID supplied by the frontend without authorization checks.
- Resolve the user's active organization membership through Identity.
- Verify the user's permissions for the requested operation.
- Scope database queries by the authorized organization.
- Verify ownership of related entities before creating relationships.
- Verify that customer addresses and bank accounts belong to the referenced customer.
- Never allow one dealership to access another dealership's records by changing a UUID.

## 10.2 Sensitive data

The following require additional protection:

- Password hashes.
- Refresh tokens.
- API keys.
- Customer bank account details.
- Date of birth.
- Addresses.
- Financial application information.
- Identity-related information.

Use encryption at rest and in transit, access controls, secure secrets management, and log redaction.

## 10.3 Database access

- Each service must use its own database credentials.
- Database credentials must follow least privilege.
- Application database users must not have unrestricted administrative permissions.
- Migrations should use appropriately privileged credentials.
- Production secrets must not be committed to source control.

---

# 11. Indexing and Constraints

Indexes should be added according to actual query patterns and uniqueness requirements.

## 11.1 Identity

```text
users(normalized_email) UNIQUE
organization_members(organization_id, user_id)
roles(organization_id, code)
role_permissions(role_id, permission_id) UNIQUE
member_roles(organization_member_id, role_id) UNIQUE
refresh_tokens(user_id, expires_at)
```

## 11.2 Configuration

```text
finance_products(code) UNIQUE
finance_product_versions(finance_product_id, version_number) UNIQUE
organization_finance_products(organization_id, finance_product_id)
organization_settings(organization_id, setting_key)
asset_categories(code) UNIQUE
```

## 11.3 Financing

```text
customers(organization_id)
customer_addresses(customer_id)
customer_bank_accounts(customer_id)
assets(organization_id)
asset_vehicles(registration_number)
asset_vehicles(vin)
organization_lenders(organization_id, lender_id)
calculations(organization_id, calculation_reference)
quotations(organization_id, quotation_number)
quotation_versions(quotation_id, version_number) UNIQUE
quotation_version_applicants(quotation_version_id)
proposals(organization_id, proposal_number)
proposal_versions(proposal_id, version_number) UNIQUE
proposal_version_applicants(proposal_version_id)
external_submissions(proposal_id)
external_submissions(proposal_version_id)
external_submissions(proposal_package_reference_id)
```

## 11.4 Documents

```text
documents(organization_id, document_type)
documents(checksum)
document_versions(document_id, version_number) UNIQUE
document_templates(organization_id, template_code)
generated_documents(source_entity_type, source_entity_id)
proposal_package_references(package_reference) UNIQUE
proposal_package_references(proposal_version_id)
document_processing_jobs(status, created_at)
```

Where soft deletion applies, uniqueness constraints must be designed deliberately.

For example, an organization may reuse a quotation number after an old record has been soft-deleted only if the business rules explicitly permit it.

Do not add indexes indiscriminately. Review query plans and actual usage.

---

# 12. Data Lifecycle and Immutability

## 12.1 Master data

Customer, address, bank account, vehicle, and lender information are maintained as reusable master records.

Changes to master data must not silently rewrite historical transaction information.

## 12.2 Transaction references

Quotations and proposals reference the customer and asset records used in their workflows.

Applicants reference customer records where applicable.

The same customer can participate in multiple quotations and proposals without creating duplicate master records.

## 12.3 Historical snapshots

Where information may change and historical accuracy is required, the transaction version preserves the exact values used.

Examples:

- Applicant name and contact details.
- Applicant address.
- Applicant bank details.
- Vehicle details.
- Finance product version.
- Financial inputs and results.
- Selected lender relationship.

Snapshots are immutable transaction information, not reusable customer or vehicle master records.

The snapshot should contain only the information relevant to the transaction.

Do not create duplicate address or customer master tables for individual business workflows.

## 12.4 Submitted proposal versions

Once submitted externally, a proposal version is immutable.

Changes require a new version.

Submitted versions must not be:

- Edited.
- Overwritten.
- Recalculated in place.
- Physically deleted.
- Silently replaced by another version.

The original submission and package reference must remain traceable.

The database must protect submitted versions and their dependent snapshots against ordinary update and deletion operations.

Use appropriate database permissions, triggers or another robust enforcement mechanism.

## 12.5 Soft deletion

Soft deletion applies to appropriate primary business records.

Examples:

- Customers.
- Customer addresses.
- Customer bank accounts.
- Vehicles.
- Lender relationships.
- Quotations.
- Proposals.
- Templates.
- Organization settings.

Do not apply soft deletion mechanically to every supporting table.

Immutable transaction records, audit records, token records, and version-history records must follow their own lifecycle and retention rules.

---

# 13. Database Migrations and Seeding

Each service owns its Alembic migration history.

```text
AssetFlow-identity/
└── alembic/

AssetFlow-configuration/
└── alembic/

AssetFlow-financing/
└── alembic/

AssetFlow-document/
└── alembic/
```

Migrations must:

- Be version controlled.
- Be repeatable in local environments.
- Avoid destructive operations without explicit review.
- Include required indexes and constraints.
- Support safe upgrades.
- Be tested before production deployment.

## 13.1 Initial seed data

**Identity**

- System permissions.
- Default role templates, if implemented.

**Configuration**

- Hire Purchase.
- Finance Lease.
- Operating Lease.
- Vehicle asset category.
- Initial vehicle field definitions.
- Initial HP product version and rules when defined.

**Financing**

- No hardcoded dealership customer or lender data.

**Documents**

- Platform-provided document templates only when their content and ownership are finalized.

Finance Lease and Operating Lease may exist in the product catalog but remain unavailable to Release 1 organizations until enabled and implemented.

---

# 14. Explicitly Out of Scope

The following are not part of the Release 1 database design:

- Lender user accounts.
- Lender-side underwriting.
- Credit checks performed by lenders.
- Lender approval or rejection workflows.
- Lender agreement generation and management.
- Agreement servicing.
- Repayment schedules and collections.
- Payment tracking.
- Lender-side application processing.
- Branch management.
- Multi-asset-category operational workflows.
- Finance Lease calculation and proposal workflows.
- Operating Lease calculation and proposal workflows.
- Direct lender API submission integrations.

These may be introduced in future releases after their requirements are defined.

---

# 15. Design Decisions and Invariants

The following decisions are mandatory unless formally revised.

1. AssetFlow is one platform, not three separate customer-facing software products.
2. Finance products are separate records in the configuration database.
3. Release 1 supports vehicle dealerships and Hire Purchase only.
4. Four independently owned databases are used.
5. All four databases can run on one local PostgreSQL server.
6. No cross-database foreign keys or direct database queries.
7. Identity owns users, organizations, memberships, roles and permissions.
8. Email addresses are globally unique across user accounts.
9. Branches are excluded from Release 1.
10. Financing owns customers, addresses, bank accounts, assets, lenders, calculations, quotations, proposals and submissions.
11. Documents owns document files, templates, generated documents and package references.
12. A customer has one master record regardless of how many quotations or proposals reference them.
13. Customer addresses are maintained once in `customer_addresses`.
14. Customer bank accounts are maintained once in `customer_bank_accounts`.
15. Transactions reference existing customer, address and bank account records where applicable.
16. Historical transaction versions preserve the exact information used when necessary.
17. Historical snapshots are not separate reusable customer or address master records.
18. Financial application data uses explicit relational fields and related tables, not one large JSONB blob.
19. JSONB is allowed for genuinely configurable definitions and metadata.
20. Quotations and proposals preserve versioned transaction information.
21. Submitted proposal versions are immutable.
22. Applicable primary business entities have `is_active` and `is_deleted`.
23. Soft deletion does not physically remove historical business records.
24. A submission references the exact proposal version and registered proposal package.
25. Downloading a package is not equivalent to submitting it externally.
26. No lender-side workflow or payment tracking is included in Release 1.

---

# 16. Items Requiring Detailed Specification Before Implementation

This document establishes database ownership, entities and relationships. It does not mean every field is final.

The following must be resolved in detailed specifications before implementation.

| Area | Required detail | Specification |
|---|---|---|
| HP calculations | Exact inputs, formulas, outputs, rounding and validation | `08_FINANCIAL_RULES.md` |
| Proposal fields | Complete lender application field inventory | Business/domain specification |
| Customer data | Required individual and business fields | Domain specification |
| Co-applicants | Customer linking and handling of non-customer applicants | Business/domain specification |
| Customer addresses | Address types, residential history and selection rules | Domain specification |
| Bank details | UK account validation and required fields | Domain specification |
| Vehicle data | Required dealership vehicle fields | Domain specification |
| Transaction snapshots | Exact fields preserved for each quotation/proposal version | Domain specification |
| Proposal financial details | Exact relational columns and decimal precision | `08_FINANCIAL_RULES.md` |
| Package generation | Package contents and versioning rules | Documents specification |
| Retention | Financial and personal-data retention policies | Engineering/security specification |
| Deletion | Entity-specific soft-delete and recovery rules | Engineering specification |
| Authorization | Exact permission matrix | Identity/API specification |

## 16.1 Snapshot design acceptance

Before implementation, the snapshot schema must answer:

- Which customer values are required for a quotation?
- Which values must be preserved when a quotation is issued?
- Which values are required by an HP proposal?
- Which applicant values must be sent to the lender?
- Which address and bank account are selected for each applicant?
- Can an application require previous addresses?
- How are co-applicants represented when they are not existing dealership customers?
- Which vehicle details must be preserved?
- Which fields must become immutable upon external submission?

The answers determine the final snapshot columns and constraints.

---

# 17. Implementation Acceptance Criteria

The database design is ready for implementation when:

- All entities and ownership boundaries are agreed.
- Primary keys and relationships are finalized.
- Cross-service logical references are documented.
- Tenant isolation rules are defined.
- Required constraints and uniqueness rules are agreed.
- Financial fields and calculation precision are specified.
- Applicant and co-applicant handling is defined.
- Customer master data is not duplicated across quotation and proposal workflows.
- Address and bank account master data is maintained once.
- Historical transaction snapshot requirements are finalized.
- Proposal versioning and immutability are enforceable.
- Package references can be validated across Financing and Documents.
- Soft deletion and historical retention rules are defined.
- Migration and seed-data plans are documented.

**Important:** Do not generate all production database models from this document alone. Complete the HP financial field inventory, lender application field inventory, and transaction snapshot requirements before writing migrations.