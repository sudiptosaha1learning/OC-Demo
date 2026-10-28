# Agile User Stories Backlog

This backlog was autonomously generated and signed off by the S/4HANA Agile core agent pipeline.

* **Generated At:** Wed, 28 Oct 2026 16:31:07 GMT
* **Total Backlog Items:** 4


---

### [STORY-001] - Configure Custom Output Cross-Reference Table (ZTXA_XREF) for Item Category B Documents
* **Epic Group:** Procurement Spend Governance
* **Actor:** Configuration Expert / Business Process Configuration Expert (SAP_BR_BPC_EXPERT)
* **Action:** maintain purchasing document output routing parameters in custom table ZTXA_XREF by Purchasing Organization and Purchasing Document Type
* **Benefit:** dynamic branding logos, Adobe Form layouts, and dispatch communication channels are systematically determined without hardcoding logic in core printing programs

#### Acceptance Criteria

## SAP Components
- SAP Modules: MM-PUR, Cross-Application Components (Output Control)
- Fiori App IDs: App ID F1481 - Custom Tables Maintenance (SM30/Custom CDS maintenance), App ID F0842A - Manage Purchase Orders
- Custom Tables: ZTXA_XREF (Purchasing Document Output Cross-Reference Table)
- S/4HANA Authorization Objects: S_TABU_DIS (Activity 02, 03), S_TABU_NAM

## Acceptance Criteria (Given-When-Then)
Scenario 1: Successful Maintenance of Custom Table Entry
- Given the Configuration Expert has authorization object S_TABU_DIS with Activity 02 for table ZTXA_XREF
- When the user creates an entry with Purchasing Org '1010', Document Type 'NB', Logo ID 'CORP_LOGO_US', Form Layout 'ZPO_LIMIT_LAYOUT_US', Dispatch Medium 'MAIL', and Is Active = 'X'
- Then the record is successfully saved in table ZTXA_XREF with valid UTC timestamp and sy-uname audit stamp.

Scenario 2: Validation of Mandatory Primary Key Fields
- Given the user attempts to insert an entry in ZTXA_XREF
- When either Purchasing Organization (EKORG) or Purchasing Document Type (BSART) is left blank
- Then the system rejects the entry with an error message preventing database commitment.

## Dependencies
- Purchasing Organizations defined in table T024E.
- Purchasing Document Types defined in table T161.
- MIME repository graphic tokens registered in SE78/Adobe Document Services.

## Definition of Ready (DoR)
- Custom data dictionary structure approved by SAP Basis and Enterprise Architecture.
- Transport Request created and assigned to application package Z_PROC_EXT.
- Baseline list of corporate purchasing organizations and document types documented in functional design.

## Definition of Done (DoD)
- Table ZTXA_XREF created in S/4HANA development tenant with delivery class C and data maintenance allowed.
- Table maintenance generator or custom maintenance app verified in development and QA environments.
- Unit test transport imported and validated across standard QA clients without syntax errors.

#### Clean Core Compliance & Architecture

## Technical Notes
- Table ZTXA_XREF created under customer namespace (Z*) following ABAP Cloud guidelines.
- Key fields: MANDT (CLNT 3), EKORG (CHAR 4), BSART (CHAR 4); Non-key attributes: LOGO_ID (CHAR 30), FORM_LAYOUT (CHAR 30), DISPATCH_MEDIUM (CHAR 4), IS_ACTIVE (CHAR 1), CREATED_BY (CHAR 12), CREATED_AT (DEC 15, UTC timestamp).
- Delivery Class C ensures table content is managed via transportable customizing.

## Clean Core Compliance & Architecture
- Zero modification to standard purchasing configuration tables (T024E, T161).
- Custom table resides strictly in custom customer package decoupled from standard SAP core repository objects.
- Accessible via ABAP Cloud language version for on-stack developer extensibility without violating clean core upgrade boundaries.

*Status: Approved and peer-validated.*

---

### [STORY-002] - Enforce Release Strategy Reset on PO Limit Item (Category B) Value Increases
* **Epic Group:** Procurement Spend Governance
* **Actor:** Operational Purchaser (SAP_BR_PURCHASER)
* **Action:** have the system validate Item Category B Overall Limit increases against approved Expected Value thresholds and reset PO release status when limits are increased post-approval
* **Benefit:** post-approval spend threshold circumvention is eliminated, enforcing strict fiscal governance and corporate financial delegation of authority

#### Acceptance Criteria

## SAP Components
- SAP Modules: MM-PUR, MM-PUR-PO
- Fiori App IDs: App ID F0842A - Manage Purchase Orders, App ID F2229 - My Inbox (All Approvals)
- Standard Business Objects / APIs: BUS2012, BAdI BADI_ME_PROCESS_PO_CUST (Methods: PROCESS_ITEM, CHECK)
- Key Database Tables & Fields: EKKO (FRGKE, PROCSTAT), EKPO (EPSTP, NETWR, ZWERT), ESUH (LIMIT, EXP_VALUE)

## Acceptance Criteria (Given-When-Then)
Scenario 1: Creation of Limit Item with Equal Values
- Given an Operational Purchaser creates a new Purchase Order with Document Type 'NB' and Item Category 'B' (Limit)
- When Expected Value (EKPO-ZWERT) is set to 50,000 USD and Overall Limit (ESUH-LIMIT) is set to 50,000 USD
- Then the PO Net Value (EKPO-NETWR) is assigned as 50,000 USD and standard release strategy thresholds evaluate commitment at 50,000 USD.

Scenario 2: Post-Approval Limit Increase Forces Release Reset
- Given an Item Category B Purchase Order has been fully approved with release indicator set to 'Released'
- When an Operational Purchaser modifies the PO in ME22N / F0842A, increasing Overall Limit (ESUH-LIMIT) from 50,000 USD to 75,000 USD while Expected Value remains 50,000 USD
- Then BAdI BADI_ME_PROCESS_PO_CUST intercepts the save event, resets release indicator EKKO-FRGKE to unapproved ('B'/'X'), sets PO output status to pending, and displays informational message: 'Overall limit change exceeds approved spend commitment. Release reset.'

Scenario 3: Boundary Limit Modification within Approved Tolerance
- Given an approved PO with Overall Limit 50,000 USD and Expected Value 50,000 USD
- When the purchaser modifies administrative text or non-spend delivery attributes without altering spend limits
- Then the release status remains 'Released' and no release strategy workflow is re-triggered.

## Dependencies
- PO Release Strategy configured with classification characteristics for total order value (CEKKO-GNETW).
- BAdI BADI_ME_PROCESS_PO_CUST active and assigned to development package Z_PROC_EXT.

## Definition of Ready (DoR)
- Functional approval workflow criteria signed off by corporate finance delegation of authority authority.
- Configured release groups, release codes, and release strategies in customizing client.
- Pre-existing service master data and limit item categories enabled for company code 1010.

## Definition of Done (DoD)
- ABAP implementation of BAdI BADI_ME_PROCESS_PO_CUST verified via automated ABAP Unit test classes covering both positive, boundary, and negative spend updates.
- Release reset logic confirmed end-to-end in S/4HANA QA environment with workflow inbox notification updates verified.
- Zero degradation to standard material line item PO processing.

#### Clean Core Compliance & Architecture

## Technical Notes
- Implemented via standard released developer extensibility hook BADI_ME_PROCESS_PO_CUST.
- In method PROCESS_ITEM, read limit details via interface IF_PURCHASE_ORDER_ITEM_MM.
- Validation logic checks: IF ls_item_data-epstp = 'B' AND ls_limit_data-limit > ls_limit_data-exp_value. In method CHECK, append error/warning messages using interface IF_MESSAGE_CONTAINER.

## Clean Core Compliance & Architecture
- Avoids direct modifications to SAP core table structures (EKKO, EKPO, ESUH) or core code in include ZXM06U43 (old user exits).
- Uses exclusively public released interfaces (IF_PURCHASE_ORDER_MM, IF_PURCHASE_ORDER_ITEM_MM) compliant with S/4HANA Cloud extensibility standards.

*Status: Approved and peer-validated.*

---

### [STORY-003] - Implement S/4HANA Output Determination BAdI with Dynamic Layout Lookup and Robust Fallback
* **Epic Group:** Procurement Document Output
* **Actor:** Purchasing Administrator (SAP_BR_ADMINISTRATOR)
* **Action:** dynamically resolve Adobe Form layout, corporate logo ID, and dispatch channel from custom table ZTXA_XREF during PO output issuance, applying automated fallback parameters if no record is found
* **Benefit:** multi-entity brand layouts and print forms are dispatched without manual user selection, while built-in fallback rules prevent order dispatch aborts during transaction processing

#### Acceptance Criteria

## SAP Components
- SAP Modules: Cross-Application Components (CA-GTF-OC), MM-PUR
- Fiori App IDs: App ID F0842A - Manage Purchase Orders, App ID F2065 - Output Parameter Determination
- Standard APIs / Classes: CL_APOC_OUTPUT_API, BAdI BADI_APOC_OUTPUT_DETERMINATION
- Custom Tables / Logs: ZTXA_XREF, Application Log Object 'APOC_OR_MESSAGES'

## Acceptance Criteria (Given-When-Then)
Scenario 1: Dynamic Layout and Logo Resolution from Mapping Table
- Given a Purchase Order is approved for Purchasing Org '1010' and Document Type 'NB'
- And an active entry exists in table ZTXA_XREF for (1010, NB) with Logo 'LOGO_US_CORP' and Form Layout 'ZPO_LIMIT_LAYOUT_US'
- When the output determination hook is executed by the S/4HANA Output Control framework
- Then the output item is generated with Form Layout 'ZPO_LIMIT_LAYOUT_US' and Logo 'LOGO_US_CORP'.

Scenario 2: Fallback Execution when Mapping Entry is Missing
- Given a Purchase Order is approved for Purchasing Org '9999' and Document Type 'NB'
- And no matching record exists in table ZTXA_XREF for tuple ('9999', 'NB')
- When the output determination hook queries table ZTXA_XREF and encounters sy-subrc <> 0
- Then the system automatically applies fallback parameters: Logo ID 'DEFAULT_LOGO', Form Layout 'ZSTANDARD_PURCHASE_ORDER', and Dispatch Medium 'MAIL'
- And writes a warning message to application log APOC_OR_MESSAGES indicating fallback profile applied.

Scenario 3: Database Exception Handling
- Given a transient database lock or SQL exception occurs during table ZTXA_XREF query
- When the exception CX_SY_OPEN_SQL_ERROR is caught
- Then the system defaults to the baseline fallback profile without rolling back PO document commit or terminating user transaction.

## Dependencies
- Standard S/4HANA Output Control (CA-GTF-OC) configured and active for Purchase Orders.
- Adobe Document Services (ADS) active and licensed.
- Default fallback Adobe Form layout 'ZSTANDARD_PURCHASE_ORDER' deployed in target environment.

## Definition of Ready (DoR)
- ADS connection verified via program FP_CHECK_DESTINATION_SERVICE.
- Sample PDF templates configured for corporate brand variants.
- Table ZTXA_XREF populated with verified master records for active business units.

## Definition of Done (DoD)
- BAdI BADI_APOC_OUTPUT_DETERMINATION implementation active and tested across all valid Purchasing Orgs.
- Negative fallback test cases successfully executed and verified in application log.
- Code inspected via ABAP Test Cockpit (ATC) with zero high-priority findings.

#### Clean Core Compliance & Architecture

## Technical Notes
- Implemented in BAdI BADI_APOC_OUTPUT_DETERMINATION method DETERMINE_CHANNEL / DETERMINE_FORM.
- Database query accesses ZTXA_XREF using ABAP SQL with clean error handling: TRY. SELECT SINGLE ... CATCH cx_sy_open_sql_error INTO DATA(lx_db). ... ENDTRY.
- Logs system diagnostics via standard class CL_BAL_LOGGING or output log integration APOC_OR_MESSAGES.

## Clean Core Compliance & Architecture
- Adheres strictly to standard S/4HANA Output Control framework (CA-GTF-OC) without altering NAST or using obsolete SAPscript/SmartForms.
- Invokes standard public APIs (CL_APOC_OUTPUT_API) ensuring 100% upgrade safety for future S/4HANA Cloud upgrades.

*Status: Approved and peer-validated.*

---

### [STORY-004] - Automate Supplier Master (BP FLVN01) Electronic Data Interchange (EDI) Channel Override
* **Epic Group:** Procurement Document Output
* **Actor:** Purchasing Administrator (SAP_BR_ADMINISTRATOR)
* **Action:** evaluate Business Partner supplier profile communication parameters in role FLVN01 during output channel determination to automatically override table dispatch medium to EDI when configured
* **Benefit:** integrated suppliers automatically receive electronic purchase orders without manual intervention, while non-EDI vendors receive automated SMTP PDF emails seamlessly

#### Acceptance Criteria

## SAP Components
- SAP Modules: Cross-Application Components (Master Data / Output Control), MM-PUR
- Fiori App IDs: App ID BP - Maintain Business Partner, App ID F0842A - Manage Purchase Orders
- S/4HANA CDS Views / APIs: CDS View I_BusinessPartnerSupplier, Class CL_MD_BP_MAINTAIN
- Roles & Partner Functions: BP Role FLVN01 (Supplier), Partner Function VN (Vendor)

## Acceptance Criteria (Given-When-Then)
Scenario 1: EDI Override for Configured Digital Supplier
- Given a Purchase Order created for Supplier 'SUPP01' who is maintained in BP Role 'FLVN01' with EDI Partner indicator active
- And table ZTXA_XREF specifies Dispatch Medium 'MAIL' for the corresponding Purchasing Org and Document Type
- When the output determination logic processes the PO communication channel
- Then the system overrides the dispatch channel from 'MAIL' to 'EDI' and routes the payload via the standard electronic interface.

Scenario 2: Preservation of SMTP Dispatch for Standard Non-EDI Supplier
- Given a Purchase Order created for Supplier 'SUPP02' who is maintained in BP Role 'FLVN01' with valid email address but no EDI profile
- And table ZTXA_XREF specifies Dispatch Medium 'MAIL'
- When output channel determination executes
- Then the dispatch medium remains 'MAIL' and the generated PDF is dispatched to the supplier's primary email address.

Scenario 3: Missing Supplier Communication Exception Handling
- Given a Purchase Order created for Supplier 'SUPP03' whose BP Master record in role FLVN01 has neither email address nor EDI profile
- When output channel determination evaluates the supplier profile
- Then the PO document is saved successfully, but the Output Item status is flagged as 'To Be Checked' (Error) with message ERR_BP_COMM_NOT_FOUND recorded in Output Log APOC_OR_MESSAGES.

## Dependencies
- Customer-Vendor Integration (CVI) synchronization operational between BP and ERP vendor master.
- Communication systems and communication arrangements configured for EDI integration scenarios.
- BP Role FLVN01 active with maintained partner communication data.

## Definition of Ready (DoR)
- Electronic Data Interchange logical ports and destination agreements maintained in QA environment.
- Test Business Partners created covering: (a) EDI active, (b) Email active, (c) Neither active.

## Definition of Done (DoD)
- Dynamic override logic integrated into BADI_APOC_OUTPUT_DETERMINATION implementation.
- End-to-end integration verified: EDI IDoc/SOAP payload generation confirmed for EDI partner; SMTP PDF spool confirmed for standard partner.
- Security scan passed and functional sign-off obtained from procurement operations.

#### Clean Core Compliance & Architecture

## Technical Notes
- Supplier data queried using released CDS view I_BusinessPartnerSupplier or standard class CL_MD_BP_MAINTAIN.
- Direct SELECT queries against legacy tables LFA1 or LFB1 are strictly prohibited under Clean Core guidelines.
- Logic extracts BP GUID and resolves communication preferences via released CDS view I_Supplier.

## Clean Core Compliance & Architecture
- Eliminates legacy customer modifications in include programs or user exits (e.g., EXIT_SAPLLMDE_001).
- Uses SAP S/4HANA released Core Data Services (CDS) views (I_BusinessPartnerSupplier) to ensure decoupling of database persistence and business application layers.

*Status: Approved and peer-validated.*

---

