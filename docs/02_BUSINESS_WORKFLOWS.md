# LeaseFlow — Business Workflows

**Document Version:** 1.0  
**Status:** Draft for workflow baseline  
**Market:** United Kingdom  
**Initial Organization Type:** Vehicle Dealership  
**Initial Finance Product:** Hire Purchase (HP)  
**Related Documents:** `00_PROJECT_CHARTER.md`, `01_PRD.md`

---

# 1. Purpose

This document defines the business workflows for LeaseFlow Release 1.

It describes how a vehicle dealership uses LeaseFlow to manage customers, vehicles, Hire Purchase calculations, quotations, finance proposals, documents, and external lender submissions.

It establishes:

- User actions.
- System responsibilities.
- Workflow sequences.
- Record statuses.
- Business validations.
- State transitions.
- Exception handling.
- Business boundaries.

Detailed database structures, API endpoints, and frontend screens will be defined in their respective documents.

# 2. Core Business Workflow

The primary LeaseFlow workflow is:

1. Dealership setup.
2. Customer creation or selection.
3. Vehicle creation or selection.
4. Finance calculation using Hire Purchase.
5. Quotation preparation.
6. Customer quotation review.
7. Finance proposal preparation.
8. Supporting document collection.
9. Proposal validation and finalization.
10. Application package generation.
11. Manual submission through the external lender's channel.
12. Recording the submission in LeaseFlow.

<escape>
Customer
   |
   v
Vehicle Selection
   |
   v
Hire Purchase Calculation
   |
   v
Quotation
   |
   v
Customer Review
   |
   v
Finance Proposal
   |
   v
Supporting Documents
   |
   v
Validation and Finalization
   |
   v
Application Package
   |
   v
External Lender Submission
   |
   v
Submission Record in LeaseFlow
</escape>

**End boundary:** LeaseFlow finishes its workflow when the dealership has submitted the prepared application package externally and recorded that submission.

LeaseFlow does not process the lender's application after submission.

---

# 3. Workflow 1 — Dealership Setup

## 3.1 Objective

Establish a dealership organization and configure the minimum information required to use LeaseFlow.

## 3.2 Actors

- Organization Owner.
- Organization Administrator.

## 3.3 Workflow

1. An organization is created or provisioned.
2. The organization owner account is established.
3. The owner completes the organization profile.
4. Organization branding is configured.
5. Branches are added if required.
6. Users are invited or created.
7. Roles and permissions are assigned.
8. Available finance products are configured.
9. External lender references are added.
10. Dealership-lender relationships are configured.
11. Relevant document templates are uploaded or configured.

## 3.4 Organization Information

The organization profile may contain:

- Legal/business name.
- Trading name.
- Business contact information.
- Address.
- Logo.
- Business registration information where applicable.
- Document branding information.
- Regional settings.

Exact fields will be defined in the domain model.

## 3.5 User Management

An administrator can:

- Invite a user.
- Assign a role.
- Assign permissions where supported.
- Activate or deactivate a membership.
- Update permitted user details.

A user must have an active membership in the relevant dealership to access its data.

## 3.6 Completion Conditions

Dealership setup is sufficiently complete when:

- An active organization exists.
- At least one authorized user can access it.
- Required organization settings are configured.
- Hire Purchase is available.
- The dealership can begin creating customers and vehicles.

Lender references and document templates may be configured when needed rather than being mandatory for initial account creation.

---

# 4. Workflow 2 — Customer Management

## 4.1 Objective

Create and maintain customer information required for dealership sales and finance preparation.

## 4.2 Actors

- Sales Officer.
- Finance Officer.
- Finance Manager.
- Other authorized users.

## 4.3 Create Customer

1. User opens customer management.
2. User starts a new customer record.
3. User enters the required customer information.
4. System validates mandatory fields and formats.
5. System checks for potential duplicate records.
6. User reviews the information.
7. System saves the customer under the active dealership organization.

## 4.4 Update Customer

1. User searches for an existing customer.
2. User opens the customer record.
3. User updates permitted information.
4. System validates the changes.
5. System saves the updated information.
6. Relevant changes are recorded in audit history.

## 4.5 Customer Information

The customer record may include:

- Customer name.
- Contact information.
- Email address.
- Phone number.
- Residential address.
- Date of birth where required.
- Employment information where required.
- Income information where required.
- Other information required by the selected finance application.

The customer record should distinguish general dealership information from additional information collected for a particular finance proposal.

Proposal-specific information must not unnecessarily overwrite the customer's general profile.

## 4.6 Customer Status

Initial customer statuses:

| Status | Meaning |
|---|---|
| Active | Available for normal dealership operations |
| Archived | Retained for historical reference but not available for normal new-workflow selection |

Archiving a customer must not delete historical quotations, proposals, calculations, or documents.

## 4.7 Business Rules

- Every customer belongs to one dealership organization.
- A customer cannot be accessed by another dealership unless that dealership independently creates its own record.
- Customer records may be reused across multiple quotations and proposals within the same organization.
- Potential duplicates should be identified without automatically merging records.
- A customer with historical finance records cannot be hard-deleted through ordinary user operations.
- Changes to customer information must not silently alter finalized proposal snapshots.

## 4.8 Exceptions

- Missing mandatory information: prevent saving or clearly identify incomplete fields, depending on the operation.
- Potential duplicate: show the existing matching records and allow the user to review them.
- Invalid contact information: reject the invalid field.
- Archived customer: prevent starting a new quotation or proposal unless the customer is reactivated.

---

# 5. Workflow 3 — Vehicle Management

## 5.1 Objective

Maintain dealership vehicle inventory and vehicle information used in finance quotations and proposals.

## 5.2 Actors

- Sales Officer.
- Finance Officer.
- Finance Manager.
- Other authorized users.

## 5.3 Create Vehicle

1. User opens vehicle management.
2. User creates a vehicle record.
3. User enters vehicle identification and specification details.
4. User enters its condition and relevant classification.
5. User enters the relevant financial values.
6. User attaches images or supporting documents if required.
7. System validates the information.
8. System saves the vehicle under the dealership organization.

## 5.4 Update Vehicle

1. User searches for a vehicle.
2. User opens the vehicle record.
3. User updates permitted information.
4. System validates the changes.
5. System saves the changes.
6. Relevant changes are recorded.

## 5.5 Vehicle Information

Potential information includes:

- Manufacturer.
- Model.
- Variant.
- Model year.
- Registration number.
- VIN or other vehicle identifier.
- New or used classification.
- Mileage.
- Vehicle condition.
- Vehicle specifications.
- Purchase cost.
- Sale price.
- Vehicle images.
- Supporting documents.

The exact required fields will be defined in the domain model.

## 5.6 Vehicle Status

| Status | Meaning |
|---|---|
| Available | Available for a new sales/finance workflow |
| Reserved | Temporarily reserved for a customer |
| Sold | Sale has been completed |
| Archived | Retained for historical reference but unavailable for new workflows |

The exact inventory rules and whether additional statuses are required will be determined during domain modelling.

## 5.7 Business Rules

- Every vehicle belongs to one dealership organization.
- A vehicle can be associated with multiple historical quotations or proposals where the business history requires it.
- A sold or archived vehicle cannot normally be selected for a new quotation.
- Changes to vehicle details must not silently change finalized quotation or proposal values.
- Historical records must preserve the relevant vehicle information used when they were created.
- Vehicle identifiers should be validated and checked for potential duplicates within the dealership.

## 5.8 Exceptions

- Duplicate registration or VIN: warn the user and request review.
- Missing required vehicle information: identify the missing fields.
- Sold vehicle: prevent use in a new finance workflow.
- Vehicle referenced by historical records: retain its history when its status changes.

---

# 6. Workflow 4 — External Lender Management

## 6.1 Objective

Allow a dealership to maintain its external lender references and define which lenders it works with.

Lenders are external counterparties. They do not log into LeaseFlow in Release 1.

## 6.2 Actors

- Organization Owner.
- Organization Administrator.
- Authorized Finance Manager.

## 6.3 Add Lender

1. User opens lender management.
2. User adds or selects a lender reference.
3. User enters the lender's relevant business information.
4. User records its external submission portal URL if available.
5. User configures the dealership's relationship with that lender.
6. User selects the finance products supported by the relationship.
7. System saves the dealership-lender relationship.

## 6.4 Lender Relationship

The relationship may contain:

- Lender reference.
- Dealership-specific lender account/reference.
- Supported finance products.
- External portal URL.
- Relationship status.
- Effective dates.
- Contact details.
- Internal notes.

A dealership can work with multiple lenders.

The same lender can be associated with multiple dealerships, but each dealership's relationship configuration and private information must remain isolated.

## 6.5 Relationship Status

| Status | Meaning |
|---|---|
| Active | Available for new proposal preparation |
| Inactive | Retained for history but unavailable for new proposals |

## 6.6 Business Rules

- Only active lender relationships can be selected for new proposals.
- A lender relationship must support the selected finance product.
- Existing proposals must retain their selected lender reference even if the relationship later becomes inactive.
- A lender's external portal URL is informational and does not mean LeaseFlow integrates with that portal.
- No lender-side user or decision workflow is created.

---

# 7. Workflow 5 — Hire Purchase Finance Calculation

## 7.1 Objective

Calculate an HP finance illustration using the dealership's configured product rules and required finance inputs.

## 7.2 Actors

- Finance Officer.
- Finance Manager.
- Other authorized users.

## 7.3 Start Calculation

1. User opens the finance calculator.
2. User selects Hire Purchase.
3. User selects a customer.
4. User selects a vehicle.
5. System loads the relevant vehicle information.
6. User enters the required finance inputs.
7. System validates the inputs.
8. System performs the calculation.
9. System displays the results.
10. User reviews the results.
11. User saves the calculation if required.

## 7.4 Calculation Inputs

The HP calculation may require:

- Vehicle cash price.
- Deposit.
- Finance amount or values from which it is derived.
- Interest/rate assumptions.
- Finance term.
- Payment frequency.
- Applicable fees and charges.
- Other product-specific inputs.

The exact required inputs and formulas belong in `08_FINANCIAL_RULES.md`.

## 7.5 Calculation Outputs

The calculation may produce:

- Amount financed.
- Periodic payment.
- Total payable.
- Total finance charge.
- Applicable fees.
- Other relevant financial outputs.

All outputs must follow the approved calculation rules and rounding behavior.

## 7.6 Calculation Lifecycle

| Status | Meaning |
|---|---|
| Draft | Inputs are being entered or edited |
| Calculated | A valid calculation result has been produced |
| Saved | The calculation has been retained for future reference |
| Superseded | A newer calculation replaces it for the relevant workflow |

A calculation may be saved without being used in a quotation.

## 7.7 Business Rules

- A calculation must identify its finance product and product version.
- A calculation must retain the calculation-rule version and engine version.
- Inputs and outputs must be stored for saved calculations.
- Recalculating changed inputs must create a new result rather than silently changing an existing saved result.
- Calculations must not be described as lender approval or a guaranteed offer.
- Invalid input combinations must not produce a valid calculation.
- A calculation used in a finalized quotation must remain historically reproducible.

## 7.8 Exceptions

- Missing required input: identify the missing value.
- Negative or out-of-range values: reject according to the financial rules.
- Unsupported term or payment frequency: reject the calculation.
- Calculation engine error: report failure without saving a partial result.
- Product rule version unavailable: prevent calculation and report the configuration issue.

---

# 8. Workflow 6 — Quotation Management

## 8.1 Objective

Prepare a customer-facing dealership quotation based on a vehicle and a valid finance calculation.

A quotation is not a lender decision or a lender-issued agreement.

## 8.2 Actors

- Sales Officer.
- Finance Officer.
- Finance Manager.

## 8.3 Create Quotation

1. User starts a new quotation.
2. User selects a customer.
3. User selects a vehicle.
4. User selects Hire Purchase.
5. User selects a valid saved calculation or performs a new calculation.
6. System retrieves the relevant calculation values.
7. User reviews the quotation information.
8. System validates required information.
9. User saves the quotation as a draft.

## 8.4 Review and Issue Quotation

1. User opens the draft quotation.
2. User reviews the customer, vehicle, product, and financial information.
3. User confirms the values.
4. System creates or updates the quotation version.
5. User generates the quotation document using the dealership's assigned template.
6. User issues the quotation to the customer through the dealership's chosen communication method.

Sending the quotation externally is not the same as submitting a finance proposal to a lender.

Email delivery automation is not required in Release 1.

## 8.5 Quotation Statuses

| Status | Meaning |
|---|---|
| Draft | Quotation is being prepared |
| Issued | Quotation has been provided to the customer |
| Accepted | Customer has accepted the dealership quotation |
| Declined | Customer has declined the quotation |
| Expired | Quotation validity period has passed |
| Cancelled | Dealership has cancelled the quotation |
| Superseded | A newer quotation version replaces this one |

Customer acceptance does not mean that the lender has approved finance.

## 8.6 Business Rules

- A quotation must reference one customer, one vehicle, and one finance product.
- An HP quotation must reference a valid HP calculation.
- A quotation must preserve the financial values used when it was issued.
- Changes to underlying customer or vehicle records must not silently alter an issued quotation.
- A material change to an issued quotation must produce a new version or a replacement quotation.
- An expired, declined, or cancelled quotation cannot directly proceed to proposal preparation without an appropriate new or revised quotation.
- The dealership's template must be used for quotation generation.
- A quotation must belong to its originating dealership organization.

## 8.7 Exceptions

- Invalid calculation: prevent quotation issuance.
- Missing customer or vehicle information: identify the missing fields.
- Inactive vehicle: prevent new quotation creation.
- Inactive finance product: prevent new quotation creation.
- Template unavailable: allow draft retention but report document-generation failure.

---

# 9. Workflow 7 — Finance Proposal Preparation

## 9.1 Objective

Prepare the dealership's complete finance application information for submission to an external lender.

A proposal is the dealership's prepared application package, not a lender decision.

## 9.2 Actors

- Finance Officer.
- Finance Manager.
- Other authorized users.

## 9.3 Start Proposal

1. User selects an eligible quotation.
2. User starts a new finance proposal.
3. System copies or references the relevant customer information.
4. System copies or references the relevant vehicle information.
5. System references the applicable finance product and calculation.
6. User selects an active external lender relationship.
7. System checks whether the lender relationship supports Hire Purchase.
8. User completes the proposal-specific application information.
9. System saves the proposal as a draft.

## 9.4 Proposal Information

The proposal may include:

- Customer information.
- Customer contact and address information.
- Customer employment/income information where required.
- Vehicle details.
- Finance product details.
- Finance calculation values.
- Deposit and finance amount.
- Selected lender.
- Application-specific information.
- Required declarations or information fields defined for the submission process.

The precise information required must be defined by the relevant dealership workflow and selected lender's publicly provided or dealership-supplied application requirements.

LeaseFlow must not invent lender requirements.

## 9.5 Proposal Data Snapshot

When a proposal is prepared, it must preserve the relevant business information used at that point.

This includes:

- Customer information used by the proposal.
- Vehicle information used by the proposal.
- Finance product and version.
- Calculation inputs and outputs.
- Selected lender relationship.
- Proposal-specific information.

The proposal must not change silently if the original customer or vehicle record is later updated.

## 9.6 Proposal Statuses

| Status | Meaning |
|---|---|
| Draft | Proposal has been created and is being prepared |
| In Progress | Proposal information or supporting documents are incomplete |
| Ready for Submission | Required information and documents have passed validation and the proposal is finalized |
| Submitted Externally | Dealership has confirmed that it submitted the package to the lender |
| Cancelled | Dealership has cancelled the proposal before submission |
| Superseded | A newer proposal version has replaced this one |

There is deliberately no lender approval, rejection, underwriting, or agreement status in Release 1.

## 9.7 Proposal Completion Validation

Before a proposal can become Ready for Submission, the system must verify:

1. Customer information required by the proposal is present.
2. Vehicle information required by the proposal is present.
3. The selected finance product is valid.
4. The calculation is valid and reproducible.
5. The selected lender relationship is active.
6. The lender relationship supports the selected product.
7. Required proposal fields are complete.
8. Required supporting documents are attached.
9. Required generated documents have been successfully created.
10. No unresolved validation errors remain.

Validation requirements may differ by product and submission configuration.

## 9.8 Finalization

1. User opens the completed proposal.
2. System displays validation results.
3. User resolves any missing information or documents.
4. User reviews the proposal summary.
5. User finalizes the proposal.
6. System creates an immutable finalized proposal version.
7. System generates or confirms the required application package.
8. System marks the proposal Ready for Submission.

Finalization means the dealership's package is prepared. It does not mean the lender has received or accepted it.

## 9.9 Business Rules

- A proposal must reference a customer, vehicle, finance product, calculation, and selected lender.
- The selected lender must be active for the dealership.
- The lender must support the selected product.
- A proposal cannot be finalized while mandatory validation errors remain.
- Finalized proposal data must not be modified in place.
- A proposal revision must preserve the previous finalized version.
- A submitted proposal must retain the exact package version that was submitted.
- The dealership may cancel a proposal before external submission.
- The system must not create a lender-side application decision or agreement record.

---

# 10. Workflow 8 — Supporting Document Collection

## 10.1 Objective

Collect, manage, and attach the supporting documents required for a finance proposal.

## 10.2 Actors

- Finance Officer.
- Document Manager.
- Other authorized users.

## 10.3 Document Collection

1. User opens a finance proposal.
2. System identifies the required document categories for the proposal.
3. User reviews the required documents.
4. User uploads a document or selects an existing authorized document.
5. System validates file type, size, and other upload constraints.
6. System stores the file in the dealership's document storage.
7. System creates or updates the document metadata.
8. User associates the document with the proposal.
9. System checks whether all required document categories are satisfied.

## 10.4 Document Categories

Examples may include:

- Customer identification documents.
- Proof of address.
- Income or employment evidence.
- Vehicle documentation.
- Supporting proposal documents.
- Other documents required by the selected submission process.

The exact requirements must not be hardcoded globally when they may vary by finance product or lender.

## 10.5 Document Statuses

| Status | Meaning |
|---|---|
| Uploaded | File has been uploaded |
| Processing | File is being processed |
| Available | File is ready for authorized use |
| Failed | Upload or processing failed |
| Archived | File is retained but unavailable for normal new use |

A document's availability status is separate from whether a proposal considers its document requirement satisfied.

## 10.6 Business Rules

- Every document belongs to one dealership organization.
- A proposal may reference multiple documents.
- One document may be reused across multiple authorized business records when appropriate.
- A document must be accessible only to authorized users of its organization.
- A document must pass required processing checks before it can satisfy a mandatory proposal requirement.
- Replacing a document must preserve the applicable document version history.
- Documents attached to finalized proposals must remain available for historical reference.
- Deleting or archiving a document must not silently remove it from an already submitted package.
- Generated proposal documents must be associated with the correct proposal version.

## 10.7 Exceptions

- Unsupported file type: reject upload.
- File exceeds configured size: reject upload.
- Processing failure: mark as failed and allow safe retry.
- Missing required category: prevent proposal finalization.
- Unauthorized document access: deny access and record relevant security activity.
- Document belongs to another dealership: reject attachment.

---

# 11. Workflow 9 — Application Package Generation

## 11.1 Objective

Generate a complete, organized package containing the finalized proposal and its supporting documents.

## 11.2 Actors

- Finance Officer.
- Finance Manager.
- Authorized users.

## 11.3 Package Generation

1. User opens a Ready for Submission proposal.
2. User requests the application package.
3. System loads the finalized proposal version.
4. System retrieves the associated customer and vehicle snapshots.
5. System retrieves the applicable calculation details.
6. System selects the dealership's assigned document templates.
7. System generates required proposal documents.
8. System gathers the supporting documents attached to the proposal version.
9. System verifies that the required files are available.
10. System creates an organized package.
11. System records the package version and generation result.
12. User reviews or downloads the package.

## 11.4 Package Contents

Depending on the selected submission requirements, the package may contain:

- Finance proposal/application document.
- Customer information.
- Vehicle information.
- Finance calculation summary.
- Quotation document.
- Required supporting customer documents.
- Required vehicle documents.
- Other relevant attachments.

The package must not include unrelated documents or information from another dealership.

## 11.5 Package Statuses

| Status | Meaning |
|---|---|
| Pending | Package generation has not started |
| Processing | Package is being generated |
| Ready | Package has been generated and is available |
| Failed | Package generation failed |
| Superseded | A newer package version has replaced this one |

## 11.6 Business Rules

- A package must be generated from a finalized proposal version.
- The package must identify the proposal version from which it was generated.
- The package must contain only authorized documents belonging to the dealership.
- A failed package-generation job must not produce a package marked Ready.
- Retrying generation must not create duplicate business submissions.
- Regenerating a package after proposal changes must create a new package version.
- The exact package used for external submission must be retained or reproducible.

---

# 12. Workflow 10 — External Lender Submission

## 12.1 Objective

Allow the dealership to submit its prepared application package through an external lender portal or submission channel and record that action in LeaseFlow.

This is the final workflow boundary of the initial product.

## 12.2 Actors

- Finance Officer.
- Finance Manager.
- Other authorized users with submission-recording permission.

## 12.3 Submission Process

1. User opens a Ready for Submission proposal.
2. User reviews the selected lender.
3. User downloads or accesses the prepared application package.
4. User opens the lender's external submission channel.
5. User submits the application through the lender's own system or process.
6. User returns to LeaseFlow.
7. User records that the proposal was submitted externally.
8. User enters the submission date/time and external reference number if available.
9. System records the submitting dealership user.
10. System links the submission record to the exact proposal and package version.

LeaseFlow does not perform the external submission in Release 1.

## 12.4 Submission Record

The submission record may contain:

- Proposal ID.
- Finalized proposal version.
- Package version.
- Selected lender reference.
- Submission date/time.
- Submitting user.
- External lender reference number, if available.
- Optional dealership notes.

## 12.5 Submission Status

The only submission status required for Release 1 is:

| Status | Meaning |
|---|---|
| Submitted Externally | The dealership user has confirmed submission through the lender's external channel |

The proposal's overall status becomes Submitted Externally after a valid submission record is created.

A draft or ready proposal is not considered submitted merely because its package was downloaded.

## 12.6 Business Rules

- Only a Ready for Submission proposal can be recorded as submitted.
- The selected lender must be recorded.
- The submitted proposal version and package version must be retained.
- Submission must record the acting user and timestamp.
- An external reference number is optional because lenders may not always provide one immediately.
- A submission record cannot be silently edited to point to a different proposal version.
- If a correction is needed, it must be recorded as an auditable amendment or additional submission record according to the final domain design.
- A proposal must not be marked submitted by an ordinary package-download action.
- The system must not infer lender receipt, review, approval, rejection, or agreement issuance.

## 12.7 End of Workflow

After the dealership records the external submission:

- LeaseFlow retains the proposal and submitted package history.
- Authorized users can view the submission record.
- The dealership can retain any lender-provided reference information.
- The lender independently manages the application.

**No further lender-side workflow is implemented in Release 1.**

---

# 13. Workflow 11 — Proposal Revision and Cancellation

## 13.1 Objective

Preserve business history when a proposal needs to be changed or cancelled.

## 13.2 Revision Before Submission

1. User identifies an issue in a draft or finalized proposal.
2. If the proposal is still a draft, the user may edit it.
3. If the proposal is finalized but not submitted, the user creates a revised version.
4. System preserves the earlier version.
5. User updates the required information.
6. System validates the revised proposal.
7. User finalizes the new version.
8. System generates a new application package.

The previous finalized version remains available for audit/history.

## 13.3 Revision After Submission

A submitted proposal version must not be overwritten.

If the dealership needs to prepare corrected information:

1. User opens the submitted proposal.
2. User starts a new revision or replacement proposal.
3. System preserves the original submitted version and submission record.
4. User updates the required information.
5. System validates and finalizes the new version.
6. System generates a new package.
7. User can submit the new package externally.
8. System records the additional submission separately.

This records dealership activity only. It does not represent a lender amendment or lender-side application update.

## 13.4 Cancellation

A proposal may be cancelled before submission.

Cancellation requires:

- An authorized user.
- A cancellation action.
- A cancellation timestamp.
- A recorded actor.
- An optional reason.

A submitted proposal cannot be treated as if it was never submitted. Any subsequent action must preserve its submission history.

---

# 14. Workflow 12 — Document Template Management

## 14.1 Objective

Allow each dealership to manage its own document templates for quotations and finance proposals.

## 14.2 Actors

- Organization Administrator.
- Document Manager.
- Other authorized users.

## 14.3 Template Creation and Update

1. User opens document template management.
2. User uploads or creates a template.
3. User selects its document type.
4. User assigns the relevant finance product where applicable.
5. User configures the supported placeholders.
6. System validates the template structure and placeholders.
7. User saves and activates the template.
8. System makes the template available for relevant document-generation workflows.

## 14.4 Template Information

A template may include:

- Template name.
- Document type.
- Finance product.
- Template file.
- Version.
- Placeholder definitions.
- Status.
- Effective dates where needed.

## 14.5 Template Statuses

| Status | Meaning |
|---|---|
| Draft | Template is being configured |
| Active | Available for document generation |
| Inactive | Retained but unavailable for new generation |

## 14.6 Business Rules

- Templates belong to one dealership organization.
- Only authorized users can manage templates.
- A template must be validated before activation.
- A document-generation operation must identify the template version used.
- Updating a template must not alter previously generated documents.
- A template assigned to one dealership must not be accessible to another dealership.
- Required placeholders must be available before generation.

---

# 15. Workflow 13 — Reporting and Audit

## 15.1 Objective

Provide dealership users with visibility into business activity and traceability of important operations.

## 15.2 Reporting Workflow

1. User opens a dashboard or report.
2. System identifies the user's organization and permissions.
3. User selects available filters.
4. System retrieves permitted records.
5. System displays the relevant information.

Initial reports may include:

- Customer counts.
- Vehicle counts.
- Quotations by status.
- Proposals by status.
- Proposals ready for submission.
- Proposals submitted externally.
- Activity by user.
- Finance product usage.

Reports must not display lender-side decisions because LeaseFlow does not manage those decisions.

## 15.3 Audit Events

Important events include:

- Organization configuration changes.
- User and permission changes.
- Customer creation and updates.
- Vehicle creation and updates.
- Lender relationship changes.
- Finance calculation creation and saving.
- Quotation creation, issuance, and revision.
- Proposal creation, finalization, and revision.
- Document upload and replacement.
- Template activation and update.
- Application package generation.
- External submission recording.
- Proposal cancellation.

Audit events must identify the acting user, organization, affected record, action, and timestamp where applicable.

---

# 16. Global Business Rules

The following rules apply across LeaseFlow Release 1.

## 16.1 Organization Isolation

- Every dealership-owned record must have an organization context.
- A user must only access records belonging to an authorized organization.
- Backend services must enforce organization isolation.
- Document storage must follow the same isolation rules.
- Cross-dealership access must never be allowed merely because a user knows a record ID.

## 16.2 Historical Data Preservation

- Historical quotations must retain their issued values.
- Historical calculations must retain their inputs and outputs.
- Finalized proposals must retain their versions.
- Submitted packages must retain their exact version.
- Changes to master records must not silently alter historical business records.

## 16.3 Finance Calculation Integrity

- Calculations must be deterministic for the same inputs and rule version.
- Product and rule versions must be recorded.
- Rounding must be defined.
- Financial outputs must be testable.
- A calculation is not a lender decision.

## 16.4 Document Ownership

- Each dealership owns its document library and templates.
- Every document operation must be authorized.
- Generated files must be linked to their originating business record.
- Historical submitted documents must remain traceable.

## 16.5 External Lender Boundary

LeaseFlow may:

- Store lender reference information.
- Maintain dealership-lender relationships.
- Select a lender for a proposal.
- Prepare lender submission information.
- Generate and organize application documents.
- Record that the dealership submitted a package externally.
- Store an external reference number supplied by the dealership.

LeaseFlow will not:

- Log in to lender systems.
- Submit applications automatically in Release 1.
- Perform credit checks.
- Perform underwriting.
- Approve or reject finance applications.
- Generate lender agreements.
- Track lender-side processing.
- Collect repayments.
- Manage lender servicing.

---

# 17. Cross-Workflow Status Summary

This section provides the initial status reference for domain modelling.

| Record | Status |
|---|---|
| Customer | Active, Archived |
| Vehicle | Available, Reserved, Sold, Archived |
| Lender relationship | Active, Inactive |
| Calculation | Draft, Calculated, Saved, Superseded |
| Quotation | Draft, Issued, Accepted, Declined, Expired, Cancelled, Superseded |
| Proposal | Draft, In Progress, Ready for Submission, Submitted Externally, Cancelled, Superseded |
| Document | Uploaded, Processing, Available, Failed, Archived |
| Application package | Pending, Processing, Ready, Failed, Superseded |
| Template | Draft, Active, Inactive |

Statuses should be implemented as controlled state transitions rather than unrestricted text values.

Not every record requires every status transition. The detailed domain model and API contracts must define valid transitions.

---

# 18. Important Workflow Invariants

The following conditions must always remain true:

1. A proposal cannot be finalized without valid customer and vehicle information.
2. An HP proposal must use a valid HP calculation.
3. A proposal cannot be finalized without an active lender relationship that supports its finance product.
4. A proposal cannot be finalized while required information or documents are missing.
5. A finalized proposal version cannot be silently modified.
6. A submitted proposal must identify the exact version and package submitted.
7. Downloading a package does not count as external submission.
8. Recording external submission does not mean the lender received or approved the application.
9. No lender-side processing status is required.
10. No lender-side agreement or payment workflow exists in Release 1.
11. A dealership cannot access another dealership's business records or documents.
12. Historical financial and document records must remain traceable.

---

# 19. Open Decisions for Later Specifications

The following details require further definition but do not prevent this workflow baseline from being established:

1. Exact customer fields required for HP applications.
2. Exact vehicle fields and inventory rules.
3. Exact HP calculation inputs, formulas, and rounding.
4. Quotation validity period and expiration behavior.
5. Whether quotation acceptance is recorded manually or through another dealership process.
6. Exact proposal-specific fields and lender submission requirements.
7. Required document categories and whether they vary by lender.
8. Whether proposal validation is entirely system-driven or allows explicitly documented dealership overrides.
9. Whether a proposal may be submitted more than once to the same lender and how resubmissions are represented.
10. Exact cancellation and revision rules.
11. Exact document package format and file organization.
12. Which reporting metrics are required in Release 1.

These decisions must be documented in the relevant specification before implementation of the affected functionality.

---

# 20. Workflow Completion Criteria

The Release 1 workflow is complete when a dealership user can:

1. Access the correct dealership organization.
2. Create or select a customer.
3. Create or select a vehicle.
4. Calculate Hire Purchase finance.
5. Prepare and issue a quotation.
6. Create a finance proposal from the relevant quotation.
7. Select an external lender.
8. Complete required proposal information.
9. Upload and attach supporting documents.
10. Validate and finalize the proposal.
11. Generate and access the complete application package.
12. Submit that package through the lender's external channel.
13. Record the external submission in LeaseFlow.
14. Retrieve the historical proposal, package, and submission record.

All of this must work locally, with appropriate authorization, tenant isolation, audit history, and automated tests.

---

**Release 1 workflow boundary:** LeaseFlow takes a dealership from customer and vehicle information to a finalized Hire Purchase application package and a dealership-recorded external submission. The lender's process begins outside LeaseFlow.