# Agile User Stories Backlog

This backlog was autonomously generated and signed off by the S/4HANA Agile core agent pipeline.

* **Generated At:** Wed, 28 Oct 2026 15:13:55 GMT
* **Total Backlog Items:** 4


---

### [STORY-001] - Validate Overall Spend Limits against Authorized Expected Value on Service POs (Item Category B)
* **Epic Group:** Procurement Governance & Financial Controls
* **Actor:** As an Operational Purchaser (SAP_BR_PURCHASER)
* **Action:** I want the system to validate that the Overall Limit does not exceed the Expected Spend Value on Item Category B purchase orders during pre-commit checks in ME21N and ME22N
* **Benefit:** so that unauthorized post-release spend inflation is prevented and compliance with corporate financial delegation of authority is strictly enforced

#### Acceptance Criteria

## SAP Components
- SAP Modules: MM-PUR (Purchasing), MM-ES (Service Procurement)
- Fiori App IDs: App ID F0842A - Manage Purchase Orders, SAP GUI Transactions ME21N / ME22N
- Database Structures: EKKO, EKPO, ESUC (Limit Data), Table T160M (System Messages)
- Message Class: ZTX_STP_MSG, Message 001

## Acceptance Criteria (Given-When-Then)
Scenario 1: Successful limit verification on new PO creation (Item Category B)
- Given an Operational Purchaser creates a purchase order with Document Type 'NB' and Item Category 'B' (Limit)
- When the purchaser sets Expected Value (ESUC-EXP_VALUE) to $50,000 and Overall Limit (ESUC-SUMLIMIT) to $50,000
- And triggers the pre-commit document check or save
- Then the system allows the save operation, commits the purchase order, and triggers standard release strategy based on the authorized expected value of $50,000.

Scenario 2: Hard stop rejection on post-release overall limit overrun (Control Gap Check)
- Given a purchase order with Item Category 'B' has been approved and released (EKKO-PROCSTAT = '05') with an Expected Value of $50,000 and Overall Limit of $50,000
- When a purchaser attempts in ME22N or Manage Purchase Orders to increase the Overall Limit to $75,000 without increasing the approved Expected Value
- And attempts to commit the changes
- Then the system aborts the database commit flow and raises a hard error message TYPE 'E' (ZTX_STP_MSG 001): 'Overall Limit ($75,000) exceeds approved expected value ($50,000). Document release reset required.'

## Dependencies
- Activated PO Release Strategy configuration for Service Document Types.
- Message Class ZTX_STP_MSG maintained in SE91.

## Definition of Ready (DoR)
- Functional approval matrix defined for service purchase orders.
- Pre-commit extension point ME_PROCESS_PO_CUST analyzed and confirmed active in development client.

## Definition of Done (DoD)
- BAdI ME_PROCESS_PO_CUST method CHECK fully implemented and passing unit tests.
- Clean Core automated testing verifies zero direct updates to EKKO/EKPO standard tables.
- QA sign-off confirming commit rollback upon error ZTX_STP_MSG 001.

#### Clean Core Compliance & Architecture

## Technical Notes
- Extension Hook: Standard BAdI ME_PROCESS_PO_CUST, method CHECK.
- Limit Structure Access: Cast line item instance using interface IF_PO_LIMIT_ITEM_MM and invoke GET_LIMIT_DATA() to extract ESUC fields without querying raw database tables.
- Error Propagation: Append error message via IF_PURCHASE_ORDER_MM->SET_ERROR_FLAG or macro MESSAGE ... TYPE 'E'.

## Clean Core Compliance & Architecture
- 100% Clean Core compliant via released SAP S/4HANA Cloud extensibility interface ME_PROCESS_PO_CUST.
- Zero modifications to core purchasing database tables (EKKO, EKPO) or standard function modules.

*Status: Approved and peer-validated.*

---

### [STORY-002] - Maintain Dynamic Output Dispatch Cross-Reference Table (ZTXA_XREF)
* **Epic Group:** Output Channel & Layout Configuration
* **Actor:** As a Procurement Data Specialist (SAP_BR_BUPA_MASTER_SPECIALIST)
* **Action:** I want a customizing cross-reference table to map Purchasing Organization and Document Type to target Adobe Form templates, branding logos, and output channels
* **Benefit:** so that purchase order documents automatically inherit the appropriate legal layout, branding assets, and transmission channels without hardcoded program logic

#### Acceptance Criteria

## SAP Components
- SAP Modules: MM-PUR (Purchasing), BC-SRV-OUT (Output Management)
- Transactions / Apps: SM30 (Table Maintenance), View Maintenance V_ZTXA_XREF
- Custom Table: ZTXA_XREF (Client-dependent, Delivery Class 'C')
- Authorization Object: S_TABU_DIS with authorization group '&NC&'

## Acceptance Criteria (Given-When-Then)
Scenario 1: Successful creation and activation of cross-reference routing mapping
- Given a Procurement Data Specialist has authorization to maintain customizing tables
- When they access table ZTXA_XREF and add an entry for Purchasing Organization '1010' and Document Type 'NB' with Logo ID 'BRAND_A_LOGO', Form Layout 'Z_FORM_SRV_PO', Medium 'EMAIL', and Active Flag 'X'
- And clicks Save within a customizing transport request
- Then the record is successfully stored in table ZTXA_XREF and is immediately available for runtime dispatch lookup.

Scenario 2: Validation of mandatory key and layout fields
- Given a Procurement Data Specialist creates a new entry in ZTXA_XREF
- When they enter Purchasing Organization '1010' and Document Type 'NB' but leave Form Layout Name blank
- And attempt to save
- Then the maintenance dialog rejects the save with a validation error indicating that 'FORM_LAYOUT_NAME' is mandatory for active configurations.

## Dependencies
- DDIC domain and data elements created for ZTXA_LOGO_ID and ZTXA_FORM_LAYOUT.
- Transport route configured between development and quality assurance tenants.

## Definition of Ready (DoR)
- Data dictionary schema for ZTXA_XREF finalized with check tables T024E (EKORG) and T161 (BSART).
- Transport request allocated and SM30 maintenance view generated.

## Definition of Done (DoD)
- Table ZTXA_XREF active in DDIC with Delivery Class 'C'.
- Authorization checks verified for unauthorized users (read-only vs write access).
- Initial routing entries populated in QA staging environment.

#### Clean Core Compliance & Architecture

## Technical Notes
- Delivery Class: C (Customizing Table), Data Class APPL2.
- Foreign Key Relationships: EKORG bound to T024E, BSART bound to T161.
- Maintenance View: Custom maintenance view with standard lock management (VIEW_MAINTENANCE_CALL).

## Clean Core Compliance & Architecture
- Customizing table maintained in customer namespace (ZTXA*).
- Adheres to clean core extensibility guidelines by decoupling routing rules from core application logic.

*Status: Approved and peer-validated.*

---

### [STORY-003] - Automated Dynamic Output Determination and Fallback Dispatch Engine via S/4HANA Output Control
* **Epic Group:** Automated Document Dispatch Framework
* **Actor:** As an S/4HANA Integration Developer
* **Action:** I want to implement BAdI APOC_DISPATCH_DOCUMENT to dynamically evaluate ZTXA_XREF and dispatch purchase orders with target Adobe layouts, logos, and channels, including default fallback routines
* **Benefit:** so that purchase orders are automatically delivered to suppliers with accurate corporate branding and layouts, ensuring guaranteed delivery even if configuration mappings are missing

#### Acceptance Criteria

## SAP Components
- SAP Modules: BC-SRV-OUT (Output Management), MM-PUR
- Fiori App: App ID F2229 - Output Parameter Determination
- BAdI Definition: APOC_DISPATCH_DOCUMENT / IF_BADI_APOC_DISPATCH
- API Classes: CL_APOC_OUTPUT_API, Application Log Object 'OUTPUT_PO'
- Fallback Defaults: Logo 'DEFAULT_LOGO', Layout 'Z_STANDARD_PO_FORM', Medium 'EMAIL'

## Acceptance Criteria (Given-When-Then)
Scenario 1: Dynamic selection of layout and logo based on ZTXA_XREF match
- Given a purchase order is approved with EKORG = '1010' and BSART = 'NB'
- And an active record exists in ZTXA_XREF mapping EKORG '1010' and BSART 'NB' to Logo 'BRAND_A_LOGO' and Form 'Z_FORM_SRV_PO'
- When the output determination framework triggers BAdI APOC_DISPATCH_DOCUMENT
- Then CL_APOC_OUTPUT_API=>CREATE_OUTPUT_ITEM creates an output item utilizing Form Template 'Z_FORM_SRV_PO' and Logo ID 'BRAND_A_LOGO' with channel 'EMAIL'.

Scenario 2: Graceful fallback when ZTXA_XREF configuration is missing (EXP-002)
- Given a purchase order is approved with EKORG = '9999' and BSART = 'ZSPO' which does not exist in ZTXA_XREF
- When the output determination framework executes the dynamic channel determination logic
- Then the system applies default fallback assets: Logo 'DEFAULT_LOGO', Form 'Z_STANDARD_PO_FORM', Channel 'EMAIL'
- And creates an entry in Application Log (SLG1) under Object 'OUTPUT_PO' with a warning status indicating missing mapping.

## Dependencies
- Custom table ZTXA_XREF deployed and populated.
- ADS (Adobe Document Services) communication RFC active.

## Definition of Ready (DoR)
- Target Adobe Forms (Z_STANDARD_PO_FORM, Z_FORM_SRV_PO) compiled and accessible in ADS.
- Output determination parameters registered in BRFplus / Output Management framework.

## Definition of Done (DoD)
- ABAP Class ZCL_STP_PO_LIMIT_OUTPUT implementing IF_BADI_APOC_DISPATCH deployed.
- ABAP Unit test suite achieving 100% statement coverage on fallback branch (sy-subrc <> 0).
- Verification in transaction SLG1 confirming warning logs for unmapped EKORG/BSART combinations.

#### Clean Core Compliance & Architecture

## Technical Notes
- Interface: IF_BADI_APOC_DISPATCH method DISPATCH_DOCUMENT.
- Standard API: CL_APOC_OUTPUT_API=>CREATE_OUTPUT_ITEM for cloud-compliant output creation.
- Logging: CL_BALI_LOG (Clean Core Application Log API) or standard SLG1 Object OUTPUT_PO.

## Clean Core Compliance & Architecture
- Uses standard S/4HANA Output Management extensibility BAdIs without legacy NAST modifications.
- All database queries utilize modern ABAP SQL syntax with strict host variable escaping.

*Status: Approved and peer-validated.*

---

### [STORY-004] - Supplier Master Business Partner EDI Communication Preference Resolution and Exception Handling
* **Epic Group:** Automated Document Dispatch Framework
* **Actor:** As a Supplier Integration Specialist (SAP_BR_BUPA_MASTER_SPECIALIST)
* **Action:** I want the output dispatch logic to evaluate Business Partner Supplier Role FLVN01 communication preferences and route to EDI when configured, falling back to email when unconfigured
* **Benefit:** so that vendor EDI transmission preferences are respected while preventing dispatch transmission failures through automated fallback

#### Acceptance Criteria

## SAP Components
- SAP Modules: MM-PUR, LO-MD-BP (Business Partner Master Data)
- Business Partner Role: FLVN01 (Supplier - Purchasing Org Data)
- Partner Tables / APIs: LFM1 (Vendor Master Purchasing Data), CL_MD_BP_MAINTAIN, BAPI_BUPA_ADDRESS
- Output Engine: CL_APOC_OUTPUT_API, Exception Class CX_APOC_DISPATCH_FAILURE
- Message Class: ZTX_STP_MSG, Warning 005

## Acceptance Criteria (Given-When-Then)
Scenario 1: Override to EDI transmission for suppliers with EDI preference maintained
- Given a purchase order is committed for vendor 'V10023' with Purchasing Organization '1010'
- And the Business Partner role FLVN01 for vendor 'V10023' has communication preference REC_COMM_TYPE = 'EDI'
- When the output channel determination executes
- Then the dispatch channel is overridden from 'EMAIL' to 'EDI' and dispatched via XML payload to the electronic data interchange port.

Scenario 2: Fallback to SMTP Email when EDI routing is unconfigured or defective (EXP-003)
- Given a purchase order is committed for vendor 'V10099' with communication preference 'EDI' in role FLVN01
- But the partner EDI transmission profile in WE20 is missing or defective
- When the output dispatch engine attempts EDI channel routing
- Then the system catches the routing exception, triggers fallback to standard SMTP email, writes warning ZTX_STP_MSG 005 to the application log, and sets output status to 'To Be Processed (Manual Review)'.

## Dependencies
- Business Partner CVI synchronization active for role FLVN01.
- EDI partner profiles maintained in WE20 for target electronic trading partners.

## Definition of Ready (DoR)
- Test vendor master records established with both EDI and standard email communication preferences.
- S/4HANA Business Partner CDS views (I_SupplierPurchasingOrg) verified for clean-core read access.

## Definition of Done (DoD)
- Channel override logic implemented within ZCL_STP_PO_LIMIT_OUTPUT=>DETERMINE_OUTPUT_CHANNEL.
- Integration test validated for EDI-enabled vendor and non-EDI vendor.
- Exception handling for missing EDI profile verified with warning message ZTX_STP_MSG 005.

#### Clean Core Compliance & Architecture

## Technical Notes
- Master Data Access: Read supplier purchasing data using CDS view I_SupplierPurchasingOrg or standard API rather than direct uncontrolled SELECT on LFM1.
- Communication Field: REC_COMM_TYPE evaluated for 'EDI'.
- Exception Handling: Catch CX_APOC_DISPATCH_FAILURE and log warning using SAP-released log APIs.

## Clean Core Compliance & Architecture
- Complies with Clean Core Tier 1 guidelines by reading Business Partner data via released master data interfaces.
- Robust exception handling ensures document output does not crash purchase order commit transactions.

*Status: Approved and peer-validated.*

---

