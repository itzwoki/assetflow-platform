# AssetFlow — Decision Log

**Purpose:** Records approved decisions, corrections to the specification set, and open items.

**Status:** Living document. Every decision that changes scope, ownership, architecture, data, or financial behaviour must be recorded here before implementation.

---

## 1. Naming and repository structure

| ID | Decision | Status |
|---|---|---|
| D1 | The product and codebase are named **AssetFlow** throughout. All prior `LeaseFlow` references are superseded. | Approved |
| D2 | The document service is named **`assetflow-document`** (singular), with Python package `assetflow_document` and database `assetflow_document`. | Approved |
| D3 | The project uses **six repositories** and **no separate contracts repository**. Each service owns and publishes its own OpenAPI specification and event definitions. | Approved |
| D4 | Repository branch and release model. | **Open** |

Repository list:

```text
assetflow-platform
assetflow-identity
assetflow-configuration
assetflow-financing
assetflow-document
assetflow-web
```

## 2. Domain and service ownership

| ID | Decision | Status |
|---|---|---|
| D5 | **Identity owns roles and permissions.** `users`, `organizations`, `organization_members`, `roles`, `permissions`, `role_permissions`, `member_roles`, sessions and authentication records all live in `assetflow_identity`. Configuration does not own roles or permissions. | Approved |
| D6 | **Branches are excluded from Release 1.** No branch table, endpoint, permission, or user interface. Future branch support must not change the meaning of existing records. | Approved |

## 3. Database naming

| ID | Decision | Status |
|---|---|---|
| D7 | Databases use an `assetflow_` prefix to avoid collisions on a shared local PostgreSQL server. | Approved |

```text
assetflow_identity
assetflow_configuration
assetflow_financing
assetflow_document
```

All four may run on one local PostgreSQL server. Logical ownership boundaries remain unchanged.

## 4. Currency and rounding

| ID | Decision | Status |
|---|---|---|
| D8 | **Currency is an organization-level display setting.** It is stored in `organization_settings`, selected from a dropdown in the organization settings screen, defaults to `GBP`, and supports `GBP` and `USD`. | Approved |
| D8.1 | **Currency affects formatting only.** It must not change calculation logic, decimal precision, or rounding. Rounding behaviour is fixed by the financial rules specification once approved. | Approved |
| D8.2 | `organization_profiles` does not store currency, locale, or timezone. Those belong to `organization_settings`. | Approved |
| D8.3 | Each calculation, quotation, proposal, and submission copies the active `currency_code` at creation time and retains it for historical accuracy. | Approved |

## 5. Organization provisioning and user access

| ID | Decision | Status |
|---|---|---|
| D16.1 | **Organizations are provisioned by platform administration.** A platform-administration area, accessible only to platform administrators, creates an organization together with its initial owner account and applies default organization configuration. | Approved |
| D16.2 | Self-service registration (`POST /auth/register`) creates **only a user account**. It does not create an organization. | Approved |
| D16.3 | The organization owner receives the account email and signs in to configure the organization, settings, branding, and users. | Approved |
| D16.4 | Organization owners and administrators may add users in **both** ways: (a) send an email invitation that assigns one or more roles, and (b) create a user account directly with assigned roles. | Approved |
| D16.5 | Default organization configuration is applied through a dedicated, idempotent operation so that provisioning can be retried safely. The exact cross-service mechanism remains an Identity implementation decision. | Approved |

## 6. Corrections applied to the specification set

The following contradictions were corrected in the documents:

| Document | Correction |
|---|---|
| `00_PROJECT_CHARTER.md` | Repository count and names aligned to six lowercase `assetflow-*` repositories. Database names prefixed. Branch reference removed from module description. |
| `01_PRD.md` | Branch requirements marked out of scope (`FR-ORG-005`). Branch category removed from configuration categories. Dealership setup corrected to platform provisioning and the two user-creation paths. Contracts repository replaced with per-service OpenAPI. |
| `02_BUSINESS_WORKFLOWS.md` | Branch step removed from dealership setup. User creation step reflects invitation and direct creation. |
| `03_DOMAIN_MODEL.md` | Roles and permissions moved to Identity. Contracts row removed. Branch section marked deferred. Branch references removed from organization responsibilities and relationships. |
| `04_DATABASE_DESIGN.md` | All database names prefixed. Currency ownership clarified (settings, not profiles) with display-only semantics. |
| `05_SYSTEM_ARCHITECTURE.md` | Shared contracts repository removed. Contract constraint changed to per-service OpenAPI. Repository count and names aligned. Database names prefixed. |
| `06_API_CONTRACTS.md` | Registration description corrected. Registration/org-creation open question resolved. Currency guidance updated. |

## 7. Open decisions

These are not yet resolved and must be closed before the dependent work begins.

| Area | Decision required | Blocks |
|---|---|---|
| D4 | Branch and release model across the six repositories (trunk-based vs `main` + `develop`), and whether services are versioned independently. | CI configuration, release tagging, platform version pinning |
| Financial engine | Rate convention, rounding, fees, term limits, APR, validation boundaries, output field list. | `assetflow-financing` calculation engine, `08_FINANCIAL_RULES.md` |
| Identity | Password hashing, token signing, token lifetimes, refresh transport, email verification, MFA scope, org provisioning mechanism, permission seed set. | `assetflow-identity` implementation |
| Authorization | Exact permission names and endpoint-level permission matrix. | All services |
| Proposal data | Complete proposal and applicant field inventory. | `assetflow-financing` proposals, snapshots |

## 8. Change control

Any future change affecting product scope, business boundaries, finance calculations, service ownership, database ownership, or core workflows must be recorded here with:

1. What is changing.
2. Why it is changing.
3. Which requirements and documents are affected.
4. Whether existing data or architecture is affected.
5. Whether the change is approved for the current release or deferred.
