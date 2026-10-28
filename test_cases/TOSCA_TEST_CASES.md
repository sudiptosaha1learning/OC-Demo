# Tosca Compliant Test Cases Suite

This suite of S/4HANA contextual test cases was collaboratively generated and certified by Lead QA Architects and Tosca Engineers.

* **Generated At:** Wed, 28 Oct 2026 15:16:51 GMT
* **Total Scenarios/Suites:** 4


---

## [STORY-001-9962] - Validate Overall Spend Limits against Authorized Expected Value on Service POs (Item Category B)
* **Epic Connection:** Procurement Governance & Financial Controls

### Automated TestCase Details
# Test Scenario 1: Positive Flow (Happy Path) - Creation and Processing of Item Category B PO with Compliant Spend Limits

**Description:** Comprehensive validation of the creation and release workflow for Item Category B (Limit) Purchase Orders where Overall Limit (ESUC-SUMLIMIT) is equal to or less than Expected Spend Value (ESUC-EXP_VALUE), ensuring successful database commit and release strategy trigger.
**Tosca Module Path:** SAP/MM/Purchasing/ME21N_ME22N/OrderProcessing
**Priority:** Critical
**Test Type:** E2E / Functional

---

### Test Case 1.1: Standard ME21N Service PO Creation with Equal Expected Value and Overall Limit
- **Tosca TestCase ID:** TC_STORY_001-9962_POS_ME21N_01
- **Preconditions:**
  - User assigned role `SAP_BR_PURCHASER` with authorization objects `M_BEST_BSA` (BSART='NB'), `M_BEST_EKG` (EKGRP='001'), and `M_BEST_EKO` (EKORG='1010').
  - Vendor/Supplier Business Partner `1000004` active in company code `1010` and purchasing org `1010`.
  - BAdI `ME_PROCESS_PO_CUST` method `CHECK` active in target client.
- **Test Steps:**
  1. Launch SAP GUI and execute transaction code `/nME21N`.
  2. In Document Header, select Order Type 'NB' (Standard PO), enter Vendor '1000004', Purchasing Org '1010', Purchasing Group '001', and Company Code '1010'.
  3. In Item Overview, create Line Item 10: enter Item Category 'B' (Limit), Short Text 'Facility Mechanical Maintenance', Plant '1010', and Storage Location '101A'.
  4. Navigate to Item Details tab 'Limits'. Enter Overall Limit '50000.00' and Expected Value '50000.00' (Currency USD).
  5. Select 'Check Document' (Ctrl+F9) button on the top toolbar to execute pre-commit checks.
  6. Select 'Save' (Ctrl+S) to persist the purchase order to the database.
- **Expected Results:**
  1. Transaction ME21N opens successfully.
  2. Header data is populated and verified.
  3. Item Category 'B' is accepted; standard material requirement is bypassed.
  4. ESUC-SUMLIMIT ($50,000.00) and ESUC-EXP_VALUE ($50,000.00) are entered without UI format errors.
  5. System returns informational message: 'No messages issued during check'.
  6. Purchase order is saved successfully; status bar displays message 'Standard PO created under the number 450000XXXX'; EKKO commit is validated.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | ME21N | S/4HANA PO Creation Transaction |
| BSART | NB | Document Type: Standard PO |
| LIFNR | 1000004 | Supplier / Service Provider BP |
| EKORG | 1010 | Purchasing Organization |
| EKGRP | 001 | Purchasing Group |
| BUKRS | 1010 | Company Code |
| EPSTP | B | Item Category: Limit |
| TXZ01 | Facility Mechanical Maintenance | Short Text Description |
| WERKS | 1010 | Plant |
| ESUC-SUMLIMIT | 50000.00 | Overall Limit |
| ESUC-EXP_VALUE | 50000.00 | Expected Value |

---

### Test Case 1.2: Fiori App F0842A PO Creation with Overall Limit Lower Than Expected Value
- **Tosca TestCase ID:** TC_STORY_001-9962_POS_FIORI_02
- **Preconditions:**
  - Fiori Launchpad accessible with active catalog `SAP_PRC_BC_PURCHASER_MC`.
  - User authenticated with role `SAP_BR_PURCHASER`.
  - Material Group `SERV_01` mapped to G/L account `400000` and Cost Center `10101101`.
- **Test Steps:**
  1. Log on to SAP Fiori Launchpad and open tile 'Manage Purchase Orders' (App ID F0842A).
  2. Click 'Create' button to open the Purchase Order Draft view.
  3. Populate Header: Supplier '1000004', Document Currency 'USD', Purchasing Organization '1010'.
  4. Add Item: Select Item Category 'Limit' (B), Material Group 'SERV_01', Delivery Date 'Today + 30 days'.
  5. Expand 'Service Limits' facet: Input Expected Value '40000.00' and Overall Limit '35000.00'.
  6. Click 'Apply' and then click 'Order' to trigger save.
- **Expected Results:**
  1. Manage Purchase Orders UI renders without script errors.
  2. Draft instance generated successfully.
  3. Header attributes accepted.
  4. Service Limit line initialized.
  5. Overall Limit ($35,000.00) is strictly less than Expected Value ($40,000.00), satisfying the spend limit constraint.
  6. Draft changes committed; PO ID generated; status badge displays 'In Approval' or 'Approved'.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| FIORI_APP_ID | F0842A | Manage Purchase Orders |
| SUPPLIER_ID | 1000004 | Supplier BP Identification |
| ITEM_CAT | B | Limit Item Category |
| MATKL | SERV_01 | Material Group for Services |
| ESUC-EXP_VALUE | 40000.00 | Expected Value (Authorized Target) |
| ESUC-SUMLIMIT | 35000.00 | Overall Limit (Cap Lower than Target) |
| KOSTL | 10101101 | Cost Center Allocation |

# Test Scenario 2: Negative/Error Flow - Hard Stop Rejection on Unauthorized Overall Limit Exceedance

**Description:** Validates hard-stop rejection when modifying an approved/released PO (EKKO-PROCSTAT = '05') in ME22N by setting Overall Limit (ESUC-SUMLIMIT) to a value higher than Expected Spend Value (ESUC-EXP_VALUE), ensuring ZTX_STP_MSG 001 type 'E' prevents commit.
**Tosca Module Path:** SAP/MM/Purchasing/ME22N/LimitValidation
**Priority:** Critical
**Test Type:** Functional / Negative Controls

---

### Test Case 2.1: ME22N Rejection of Overall Limit Increase on Approved Purchase Order
- **Tosca TestCase ID:** TC_STORY_001-9962_NEG_ME22N_01
- **Preconditions:**
  - Existing Purchase Order `4500009821` exists in system with Item Category 'B', ESUC-EXP_VALUE = $50,000, ESUC-SUMLIMIT = $50,000.
  - Document status is Released (`EKKO-PROCSTAT = '05'`, `EKKO-FRGKE = 'R'`).
  - User assigned standard purchaser authorizations.
- **Test Steps:**
  1. Launch SAP GUI and execute transaction `/nME22N`.
  2. Click 'Other Purchase Order' (Shift+F5), input PO Number '4500009821', and hit Enter.
  3. Expand Item Details for Item 10 and select tab 'Limits'.
  4. Update field 'Overall Limit' from '50000.00' to '75000.00', leaving 'Expected Value' unchanged at '50000.00'.
  5. Click 'Check' button (Ctrl+F9).
  6. Attempt to commit by clicking 'Save' (Ctrl+S).
- **Expected Results:**
  1. Transaction ME22N displays initial screen.
  2. PO 4500009821 loads in edit mode.
  3. Limits subscreen shows ESUC values.
  4. Overall Limit field modified to 75000.00.
  5. System raises error message in SAP GUI Error Log modal: Message Class 'ZTX_STP_MSG', Number '001', Type 'E': 'Overall Limit ($75,000) exceeds approved expected value ($50,000). Document release reset required.'
  6. Database commit is aborted; PO returns to edit mode; subsequent query on table ESUC shows SUMLIMIT remains 50000.00.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | ME22N | Change Purchase Order Transaction |
| EBELN | 4500009821 | Target Service PO (Released) |
| EBELP | 00010 | Item Line Number |
| OLD_SUMLIMIT | 50000.00 | Prior Approved Limit |
| NEW_SUMLIMIT | 75000.00 | Unauthorized Delta Limit |
| ESUC-EXP_VALUE | 50000.00 | Static Approved Expected Value |
| EXPECTED_MSG_ID | ZTX_STP_MSG | Message Class |
| EXPECTED_MSG_NO | 001 | Message Number |
| EXPECTED_MSG_TYP | E | Hard Error Severity Type |

---

### Test Case 2.2: F0842A OData Pre-Commit Validation Failure on Spend Limit Inflation
- **Tosca TestCase ID:** TC_STORY_001-9962_NEG_FIORI_02
- **Preconditions:**
  - Existing Released PO `4500009822` accessible via Fiori.
  - Browser Developer Tools / Tosca API Engine listener active on network endpoints.
- **Test Steps:**
  1. Open Fiori App F0842A (Manage Purchase Orders) and search for PO '4500009822'.
  2. Click 'Edit' button to trigger OData draft creation (`C_PURCHASEORDER_FS_SRV`).
  3. Drill down to Item 10 -> Service Limits.
  4. Change Overall Limit from '10000.00' to '25000.00' while Expected Value remains '10000.00'.
  5. Click 'Save' button in Fiori footer toolbar.
  6. Inspect UI message popover and assert error response.
- **Expected Results:**
  1. Purchase order 4500009822 details loaded.
  2. Draft state created successfully.
  3. Service Limits values displayed.
  4. Target value 25000.00 entered.
  5. HTTP POST/PATCH request to `/sap/opu/odata/sap/C_PURCHASEORDER_FS_SRV/PurchaseOrder` returns HTTP 400 Bad Request with error body mapping to `ZTX_STP_MSG/001`.
  6. Fiori Message Popover displays red error banner: 'Overall Limit ($25,000) exceeds approved expected value ($10,000). Document release reset required.'; PO remains uncommitted.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| FIORI_APP_ID | F0842A | Manage Purchase Orders |
| EBELN | 4500009822 | Released Purchase Order ID |
| ESUC-EXP_VALUE | 10000.00 | Fixed Baseline Value |
| ESUC-SUMLIMIT | 25000.00 | Infracted Overall Limit |
| HTTP_STATUS | 400 | OData Protocol Error Response |
| UI_SEVERITY | Error | Fiori Notification Badge |

# Test Scenario 3: Boundary/Edge Case Validation - Precision Decimals, Zero Limits, and Threshold Tolerances

**Description:** Evaluates limit validation behavior under extreme boundary values, including micro-cent limit discrepancies ($0.01 overrun), zero values, blank inputs, and maximum field length thresholds (999,999,999,999.99).
**Tosca Module Path:** SAP/MM/Purchasing/ME21N_ME22N/BoundaryChecks
**Priority:** High
**Test Type:** Boundary / Robustness Testing

---

### Test Case 3.1: Micro-Threshold Limit Overrun by One Cent ($0.01)
- **Tosca TestCase ID:** TC_STORY_001-9962_BND_CENT_01
- **Preconditions:**
  - Active S/4HANA test environment with base currency USD configured to 2 decimal places in TCURX.
  - Authority to run ME21N.
- **Test Steps:**
  1. Execute transaction `/nME21N`.
  2. Populate mandatory Header attributes: Vendor '1000004', Purchase Org '1010', Purchasing Group '001', Company Code '1010'.
  3. Add Item 10 with Category 'B', Material Group 'SERV_01', Plant '1010'.
  4. Navigate to 'Limits' tab: Set Expected Value to '10000.00'.
  5. Set Overall Limit to '10000.01' (1 cent higher than Expected Value).
  6. Click 'Check' (Ctrl+F9).
- **Expected Results:**
  1. ME21N initial screen loaded.
  2. Header data populated.
  3. Limit Item initialized.
  4. ESUC-EXP_VALUE set to exactly 10,000.00.
  5. ESUC-SUMLIMIT set to exactly 10,000.01.
  6. Validation triggers hard error `ZTX_STP_MSG 001`; verification succeeds that no round-off error allows the $0.01 overrun.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | ME21N | Standard PO Creation |
| EPSTP | B | Limit Category |
| ESUC-EXP_VALUE | 10000.00 | Expected Value Boundary Baseline |
| ESUC-SUMLIMIT | 10000.01 | Overrun by $0.01 Decimal |
| EXPECTED_STATUS | Abort | Commit Interrupted |
| MSG_CLASS | ZTX_STP_MSG | Custom Message Class |

---

### Test Case 3.2: Null Expected Value with Active Non-Zero Overall Limit
- **Tosca TestCase ID:** TC_STORY_001-9962_BND_NULL_02
- **Preconditions:**
  - User executing ME21N.
  - BAdI implementation validates zero/blank handling logic.
- **Test Steps:**
  1. Execute transaction code `/nME21N`.
  2. Enter Header details (Vendor '1000004', Org '1010', Group '001', CoCode '1010').
  3. Create Line Item 10: Item Category 'B', Short Text 'Emergency Generator Maintenance', Plant '1010'.
  4. In 'Limits' tab: Leave Expected Value blank or set to '0.00'.
  5. In 'Overall Limit' field: Enter '1.00'.
  6. Click 'Check Document' (Ctrl+F9).
- **Expected Results:**
  1. ME21N transaction ready.
  2. Header configured.
  3. Item Category 'B' recognized.
  4. Expected Value registered as 0.00 USD.
  5. Overall Limit registered as 1.00 USD.
  6. System issues error message `ZTX_STP_MSG 001`; system prohibits non-zero Overall Limit against zero authorized Expected Value.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | ME21N | Standard PO Creation |
| EPSTP | B | Limit Category |
| ESUC-EXP_VALUE | 0.00 | Null / Zero Value |
| ESUC-SUMLIMIT | 1.00 | Nominal Positive Spend Limit |
| MSG_TYPE | E | Error Type Validation |
| COMMIT_STATUS | Rolled Back | Database Integrity Preserved |


*Status: Approved and mapped by Tosca Automation Engineer.*

---

## [STORY-002-9962] - Maintain Dynamic Output Dispatch Cross-Reference Table (ZTXA_XREF)
* **Epic Connection:** Output Channel & Layout Configuration

### Automated TestCase Details
# Test Scenario 1: Positive Flow (Happy Path)
**Description:** Validate the standard end-to-end creation, modification, and activation of output routing mappings in table ZTXA_XREF via SM30 view maintenance with successful assignment to a Customizing Transport Request.
**Tosca Module Path:** SAP/Basis/SM30_TableMaintenance/V_ZTXA_XREF
**Priority:** Critical
**Test Type:** Functional

### Test Case 1.1: Create New Purchasing Cross-Reference Entry with Valid Parameters and Transport Recording
- **Tosca TestCase ID:** TC_STORY_002-9962_01
- **Preconditions:**
  - User assigned role with S_TCODE = SM30 and S_TABU_DIS (ACTVT = 02, 03; DICBERCLS = '&NC&').
  - Purchasing Org '1010' exists in T024E and Document Type 'NB' exists in T161.
  - Modifiable customizing transport request (e.g., S4HK900123) is available.
- **Test Steps:**
  1. Launch SAP GUI and navigate to transaction code SM30 in the command field.
  2. In Table/View field enter 'V_ZTXA_XREF' and click 'Maintain' (F5).
  3. Click 'New Entries' (F5) button on the SM30 toolbar.
  4. Populate EKORG = '1010', BSART = 'NB', LOGO_ID = 'BRAND_A_LOGO', FORM_LAYOUT = 'Z_FORM_SRV_PO', MEDIUM = 'EMAIL', and ACTIVE = 'X'.
  5. Click 'Save' (Ctrl+S) and on the Customizing Request prompt modal, specify Transport Request 'S4HK900123' and click Confirm (Enter).
- **Expected Results:**
  1. Transaction SM30 initial screen renders successfully without authorization error.
  2. Overview maintenance screen for V_ZTXA_XREF opens in change mode.
  3. Editable new row appears in the SAP Table Control.
  4. All target values populate into the corresponding table columns without truncation.
  5. Status bar confirms 'Data saved' (Message SV018) and customizing request logs the table key entry 'MANDT, 1010, NB'.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | SM30 | Table View Maintenance Transaction |
| VIEWNAME | V_ZTXA_XREF | Custom Cross-Reference Maintenance View |
| EKORG | 1010 | Purchasing Organization Germany |
| BSART | NB | Standard Purchase Order Document Type |
| LOGO_ID | BRAND_A_LOGO | Master Branding Asset Identifier |
| FORM_LAYOUT | Z_FORM_SRV_PO | Active Adobe Form Template ID |
| MEDIUM | EMAIL | Output Dispatch Channel Medium |
| ACTIVE | X | Active Routing Configuration Flag |
| TRANSPORT_REQ | S4HK900123 | Customizing Transport Request |

### Test Case 1.2: Update Existing Routing Entry Medium from EMAIL to PRINT
- **Tosca TestCase ID:** TC_STORY_002-9962_02
- **Preconditions:**
  - Entry for EKORG '1010' and BSART 'NB' exists in ZTXA_XREF with MEDIUM 'EMAIL'.
  - S_TABU_DIS write authorization active for user.
- **Test Steps:**
  1. Execute transaction SM30, set Table/View to 'V_ZTXA_XREF', and click 'Maintain'.
  2. Locate row with EKORG '1010' and BSART 'NB' using 'Position...' (F6).
  3. Modify the 'MEDIUM' field from 'EMAIL' to 'PRINT'.
  4. Click 'Save' (Ctrl+S).
  5. Assign change to transport request 'S4HK900123' and confirm.
- **Expected Results:**
  1. Table maintenance screen displays existing record.
  2. Target row is highlighted and editable.
  3. Column MEDIUM value reflects 'PRINT'.
  4. Save prompt triggered and transport request modal displays.
  5. System status bar displays 'Data saved' and table ZTXA_XREF stores updated medium 'PRINT'.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | SM30 | Table Maintenance Transaction |
| VIEWNAME | V_ZTXA_XREF | Maintenance View |
| POSITION_EKORG | 1010 | Target Filter Key Purchasing Org |
| POSITION_BSART | NB | Target Filter Key Document Type |
| MEDIUM | PRINT | Updated Output Medium Value |
| TRANSPORT_REQ | S4HK900123 | Open Customizing Transport Request |

# Test Scenario 2: Negative/Error Flow
**Description:** Validate rejection mechanisms when mandatory configuration rules are violated, such as blank required layout fields or when a user lacks S_TABU_DIS write authorization.
**Tosca Module Path:** SAP/Basis/SM30_TableMaintenance/ValidationErrors
**Priority:** High
**Test Type:** Functional

### Test Case 2.1: Enforce Mandatory FORM_LAYOUT Field Validation on Active Configuration
- **Tosca TestCase ID:** TC_STORY_002-9962_03
- **Preconditions:**
  - User has S_TABU_DIS authorization to maintain V_ZTXA_XREF.
  - Purchasing Org '2010' and Document Type 'NB' exist in standard configuration.
- **Test Steps:**
  1. Execute transaction SM30 and open 'V_ZTXA_XREF' in Maintain mode.
  2. Click 'New Entries' button.
  3. Input EKORG = '2010', BSART = 'NB', LOGO_ID = 'BRAND_B_LOGO', leave FORM_LAYOUT blank, set MEDIUM = 'EMAIL', and ACTIVE = 'X'.
  4. Click 'Save' (Ctrl+S).
  5. Inspect the status bar and cursor position.
- **Expected Results:**
  1. Maintenance grid accepts entry mode.
  2. New row is ready for data entry.
  3. Values entered with FORM_LAYOUT explicitly empty.
  4. Save action is rejected by the maintenance view event module.
  5. An error message displays: 'Form Layout Name is mandatory for active configurations' and cursor focuses on FORM_LAYOUT.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | SM30 | Table Maintenance Transaction |
| VIEWNAME | V_ZTXA_XREF | Maintenance View Name |
| EKORG | 2010 | Purchasing Org US |
| BSART | NB | Standard PO Document Type |
| LOGO_ID | BRAND_B_LOGO | Corporate Brand B Logo |
| FORM_LAYOUT | {EMPTY} | Left intentionally blank |
| MEDIUM | EMAIL | Dispatch Medium |
| ACTIVE | X | Active Flag Checked |
| EXPECTED_MSG_TYPE | E | Error message type |

### Test Case 2.2: Reject Maintenance Execution for User Lacking S_TABU_DIS Write Authorization
- **Tosca TestCase ID:** TC_STORY_002-9962_04
- **Preconditions:**
  - Test user account configured with S_TABU_DIS ACTVT = '03' (Display only) for authorization group '&NC&'.
  - Transaction SM30 accessible via S_TCODE.
- **Test Steps:**
  1. Log on to SAP GUI with restricted user credentials.
  2. Execute transaction code SM30.
  3. In the Table/View field, enter 'V_ZTXA_XREF'.
  4. Click the 'Maintain' (F5) button.
  5. Capture system status bar message.
- **Expected Results:**
  1. User authentication succeeds.
  2. SM30 entry screen displays.
  3. Value 'V_ZTXA_XREF' is entered.
  4. Maintenance mode access is blocked immediately.
  5. System raises authorization message 'You have no authorization for maintenance of table ZTXA_XREF' (Message SV043 or SV042).
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | SM30 | Table Maintenance Transaction |
| VIEWNAME | V_ZTXA_XREF | Maintenance View |
| AUTH_OBJECT | S_TABU_DIS | Table Maintenance Authorization Object |
| DICBERCLS | &NC& | Authorization Group Without Class |
| ACTVT | 03 | Display-only authorization level |
| EXPECTED_MSG_ID | SV043 | Authorization check failure code |

# Test Scenario 3: Boundary/Edge Case Validation
**Description:** Validate foreign key referential integrity against check tables T024E and T161, and test boundary string lengths and special characters within custom cross-reference attribute fields.
**Tosca Module Path:** SAP/Basis/SM30_TableMaintenance/BoundaryChecks
**Priority:** Medium
**Test Type:** Integration

### Test Case 3.1: Foreign Key Check Failure on Non-Existent Purchasing Organization and PO Document Type
- **Tosca TestCase ID:** TC_STORY_002-9962_05
- **Preconditions:**
  - Purchasing Org '9999' does NOT exist in table T024E.
  - Document Type 'ZZ' does NOT exist in table T161.
  - User has full maintain access to V_ZTXA_XREF.
- **Test Steps:**
  1. Open transaction SM30 for view 'V_ZTXA_XREF' and click 'New Entries'.
  2. Enter EKORG = '9999' (invalid) and BSART = 'NB'.
  3. Press Enter.
  4. Change EKORG to '1010' (valid) and enter BSART = 'ZZ' (invalid).
  5. Press Enter.
- **Expected Results:**
  1. New row ready for input.
  2. System evaluates check table T024E upon input.
  3. Error raised: 'Entry 9999 does not exist in T024E (check entry)' (Message 00058).
  4. Input values updated to valid EKORG and invalid BSART.
  5. Error raised: 'Entry ZZ does not exist in T161 (check entry)' preventing record commitment.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | SM30 | Table Maintenance |
| VIEWNAME | V_ZTXA_XREF | Target View |
| INVALID_EKORG | 9999 | Non-existent Purchasing Organization |
| VALID_EKORG | 1010 | Plant 1010 Pur Org |
| INVALID_BSART | ZZ | Non-existent Document Type |
| VALID_BSART | NB | Standard PO Type |
| EXPECTED_ERR | 00058 | Foreign key validation standard error |

### Test Case 3.2: Boundary Field Length and Special Character Verification in LOGO_ID and FORM_LAYOUT
- **Tosca TestCase ID:** TC_STORY_002-9962_06
- **Preconditions:**
  - ZTXA_LOGO_ID defined as CHAR30, ZTXA_FORM_LAYOUT defined as CHAR30.
  - User has authorization for V_ZTXA_XREF maintenance.
- **Test Steps:**
  1. Navigate to SM30, enter view 'V_ZTXA_XREF', and click 'New Entries'.
  2. Enter EKORG = '1010', BSART = 'NB'.
  3. Input maximum length string with underscores and hyphens in LOGO_ID: 'BRAND_ALPHA-EXTENDED_ASSET_01' (30 characters).
  4. Input maximum length string in FORM_LAYOUT: 'Z_PURCHASE_ORDER_EXP_V2_LAYOUT' (30 characters).
  5. Set MEDIUM = 'EMAIL', ACTIVE = 'X', save record to transport 'S4HK900123', and query table ZTXA_XREF via SE16N.
- **Expected Results:**
  1. Input grid displays new editable record.
  2. Key fields populated successfully.
  3. Exactly 30 characters accepted without clipping or invalid character errors.
  4. Form layout name accepted without truncation.
  5. Data saves cleanly; SE16N lookup verifies persisted string values match exactly without character corruption.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | SM30 | View Maintenance |
| VIEWNAME | V_ZTXA_XREF | Maintenance View |
| EKORG | 1010 | Purchasing Org Germany |
| BSART | NB | Document Type |
| LOGO_ID | BRAND_ALPHA-EXTENDED_ASSET_01 | Max 30-char Asset Key with delimiters |
| FORM_LAYOUT | Z_PURCHASE_ORDER_EXP_V2_LAYOUT | Max 30-char Layout Name |
| MEDIUM | EMAIL | Output Channel |
| ACTIVE | X | Active Flag |
| VERIFY_TCODE | SE16N | DDIC Table Data Browser |


*Status: Approved and mapped by Tosca Automation Engineer.*

---

## [STORY-003-9962] - Automated Dynamic Output Determination and Fallback Dispatch Engine via S/4HANA Output Control
* **Epic Connection:** Automated Document Dispatch Framework

### Automated TestCase Details
# Test Scenario 1: Positive Flow (Happy Path) - Dynamic Match in ZTXA_XREF

**Description:** Validates end-to-end dynamic output determination where purchase order parameters match an active record in ZTXA_XREF. Confirms BAdI APOC_DISPATCH_DOCUMENT successfully assigns custom Adobe Form template 'Z_FORM_SRV_PO', logo 'BRAND_A_LOGO', and channel 'EMAIL' via CL_APOC_OUTPUT_API.
**Tosca Module Path:** SAP/S4HANA_BR/MM/OutputControl/F2229_OPD_Dispatch
**Priority:** Critical
**Test Type:** E2E

---

### Test Case 1.1: End-to-End PO Creation and Dynamic Output Item Generation
**Tosca TestCase ID:** TC_STORY_003-9962_POS_01
**Preconditions:**
- User has authorization objects M_BEST_BSA, M_BEST_EKG, S_APOC_OC, S_TABU_DIS (DICBERCLS &00&).
- Table ZTXA_XREF contains active record: EKORG = '1010', BSART = 'NB', LOGO_ID = 'BRAND_A_LOGO', FORM_NAME = 'Z_FORM_SRV_PO', ACTIVE = 'X'.
- RFC connection to Adobe Document Services (ADS) is green (FP_TEST_00).
- Supplier master 'SUPP-1010' has a valid default dispatch email.

**Test Steps:**
1. Launch SAP GUI or Fiori Launchpad and access transaction ME21N (Create Purchase Order).
2. Enter Purchasing Organization '1010', Purchasing Group '001', Document Type 'NB', and Vendor 'SUPP-1010'.
3. Add line item with Material 'MAT-SRV-01', Plant '1010', Quantity '10', and Net Price '150.00'.
4. Save the Purchase Order and capture generated PO Number from status bar.
5. Navigate to 'Messages' / Output screen via ME23N or Fiori App F0842A (Manage Purchase Orders).
6. Verify that an output item is generated with Form Template 'Z_FORM_SRV_PO', Logo 'BRAND_A_LOGO', and Channel 'EMAIL' in status 'To Be Output' / 'Successful'.

**Expected Results:**
1. ME21N interface loads successfully without authorization faults.
2. Header and line item attributes accept valid data inputs.
3. Line item validation passes with zero critical messages.
4. Purchase Order is committed to database (EKKO/EKPO) with distinct EBELN.
5. Output control framework executes BAdI APOC_DISPATCH_DOCUMENT.
6. Output item displays Form Template 'Z_FORM_SRV_PO', Logo ID 'BRAND_A_LOGO', and Dispatch Medium 'EMAIL'.

**Test Data Sheet:**
| Parameter | Value | Description |
| :--- | :--- | :--- |
| EKORG | 1010 | Purchasing Organization |
| BSART | NB | Standard Purchase Order Document Type |
| EKGRP | 001 | Purchasing Group |
| LIFNR | SUPP-1010 | Domestic Approved Supplier |
| MATNR | MAT-SRV-01 | Standard Service Material ID |
| EXPECTED_FORM | Z_FORM_SRV_PO | Dynamic Adobe Form Template |
| EXPECTED_LOGO | BRAND_A_LOGO | Resolved Corporate Logo Key |
| DISPATCH_CHANNEL | EMAIL | Communication Medium Channel |

---

### Test Case 1.2: Direct Simulation of CL_APOC_OUTPUT_API with Explicit Cross-Reference Payload
**Tosca TestCase ID:** TC_STORY_003-9962_POS_02
**Preconditions:**
- Developer test authorization S_DEVELOP (DEBUG/EXECUTE) and S_APOC_OC.
- Active implementation ZCL_STP_PO_LIMIT_OUTPUT enabled for BAdI APOC_DISPATCH_DOCUMENT.

**Test Steps:**
1. Open SE24/SE38 test harness for CL_APOC_OUTPUT_API=>CREATE_OUTPUT_ITEM.
2. Instantiate payload structure with APPL_OBJECT = 'PURCHASE_ORDER', EKORG = '1010', BSART = 'NB'.
3. Execute method call triggering dynamic determination.
4. Query the returned output item table for field values FORM_TEMPLATE, LOGO_ID, and DISPATCH_STATUS.
5. Validate that sy-subrc is equal to 0.

**Expected Results:**
1. API workbench initializes test instance correctly.
2. Payload structure validates against dictionary DDIC types.
3. Execution finishes within SLA (< 250ms) without unhandled exceptions.
4. Returned record contains FORM_TEMPLATE = 'Z_FORM_SRV_PO' and LOGO_ID = 'BRAND_A_LOGO'.
5. Execution return code sy-subrc is 0.

**Test Data Sheet:**
| Parameter | Value | Description |
| :--- | :--- | :--- |
| APPL_OBJECT | PURCHASE_ORDER | S/4HANA Output Business Object |
| INPUT_EKORG | 1010 | Evaluation Purchasing Org |
| INPUT_BSART | NB | Standard Order Type |
| EXP_SUBRC | 0 | Expected System Return Code |
| EXP_STATUS | 1 | Status Initial / Determined |

# Test Scenario 2: Negative/Error Flow - Graceful Fallback on Missing Configuration

**Description:** Validates dynamic determination failure behavior when an approved PO contains an unmapped EKORG/BSART combination. Verifies that the BAdI catches the missing table record (sy-subrc <> 0), applies the hardcoded fallback defaults ('Z_STANDARD_PO_FORM', 'DEFAULT_LOGO', 'EMAIL'), and logs an official warning message into Application Log (SLG1) under Object 'OUTPUT_PO'.
**Tosca Module Path:** SAP/S4HANA_BR/Basis/Logging/SLG1_ApplicationLog
**Priority:** High
**Test Type:** Functional

---

### Test Case 2.1: Missing XREF Triggering Standard Fallback and SLG1 Warning Log
**Tosca TestCase ID:** TC_STORY_003-9962_NEG_01
**Preconditions:**
- Authorization for SLG1 (Object 'OUTPUT_PO', Subobject 'DISPATCH').
- Ensure combination EKORG = '9999' and BSART = 'ZSPO' is absent from table ZTXA_XREF.
- Default fallback assets exist in MIME/ADS repository: 'DEFAULT_LOGO' and 'Z_STANDARD_PO_FORM'.

**Test Steps:**
1. Launch transaction ME21N or invoke BAPI_PO_CREATE1 via automated interface test runner.
2. Populate header with EKORG = '9999', BSART = 'ZSPO', EKGRP = '999', and Supplier 'SUPP-9999'.
3. Populate line items and execute POST/SAVE to generate PO Number.
4. Open transaction ME23N and inspect the Output Control tab.
5. Verify that output item uses Form Template 'Z_STANDARD_PO_FORM', Logo 'DEFAULT_LOGO', and Channel 'EMAIL'.
6. Launch transaction SLG1, enter Object 'OUTPUT_PO', Subobject 'DISPATCH', External ID = <Captured_PO_Number>, and Date = Current Date.
7. Execute search and assert log tree contains a Warning message: 'No mapping found in ZTXA_XREF for EKORG 9999 / BSART ZSPO. Default fallback applied.'

**Expected Results:**
1. Transaction initializes and accepts non-standard organizational values.
2. PO saves successfully without terminating with a runtime ABAP dump.
3. Output determination invokes fallback branch.
4. ME23N Output tab reflects default form and logo settings.
5. SLG1 records the execution under Object 'OUTPUT_PO'.
6. SLG1 log level shows yellow warning icon indicating missing cross-reference configuration.

**Test Data Sheet:**
| Parameter | Value | Description |
| :--- | :--- | :--- |
| EKORG | 9999 | Unmapped Test Purchasing Org |
| BSART | ZSPO | Custom Service Order Document Type |
| FALLBACK_FORM | Z_STANDARD_PO_FORM | Fallback Adobe Layout Template |
| FALLBACK_LOGO | DEFAULT_LOGO | Fallback Corporate Logo Identifier |
| LOG_OBJECT | OUTPUT_PO | SLG1 Application Log Object |
| LOG_SUBOBJECT | DISPATCH | SLG1 Application Log Subobject |
| EXPECTED_MSG_TYP | W | Warning Message Severity |

---

### Test Case 2.2: Channel Fallback when Output Parameter Determination (BRFplus) Channel Rule is Blank
**Tosca TestCase ID:** TC_STORY_003-9962_NEG_02
**Preconditions:**
- App F2229 (Output Parameter Determination) configured with no matching channel rule for combination EKORG '9999' / BSART 'ZSPO'.
- S_APOC_OC authorization assigned.

**Test Steps:**
1. Launch Fiori Launchpad and open App F2229 (Output Parameter Determination).
2. Select Show Rules for 'Purchase Order' and Determination Step 'Channel'.
3. Ensure condition table evaluates to empty channel for EKORG '9999'.
4. Trigger BAdI execution via test program ZTEST_PO_DISPATCH_TRIGGER with PO under EKORG '9999'.
5. Inspect the generated output line in DB table APOC_D_OUTPUT_IT.
6. Verify dispatch channel defaults to 'EMAIL' and status does not enter ERROR.

**Expected Results:**
1. F2229 loads BRFplus decision table rules correctly.
2. Decision table exhibits empty channel match.
3. Test trigger runs BAdI logic cleanly.
4. Database table APOC_D_OUTPUT_IT captures row item with fallback values.
5. Channel is set to 'EMAIL' and system remains resilient without unhandled exceptions.

**Test Data Sheet:**
| Parameter | Value | Description |
| :--- | :--- | :--- |
| APP_ID | F2229 | Output Parameter Determination App |
| BRF_RULE_STEP | CHANNEL | BRFplus Rule Evaluation Step |
| FALLBACK_CHANNEL | EMAIL | Mandatory Default Dispatch Medium |
| DB_TABLE | APOC_D_OUTPUT_IT | Output Item Persistent Table |

# Test Scenario 3: Boundary/Edge Case Validation - Inactive Flags and Formatting Edge Cases

**Description:** Validates edge cases including records present in ZTXA_XREF with ACTIVE flag set to blank/' ' (inactive), as well as PO items containing extended characters in purchase order text lines to ensure the dynamic Adobe layout rendering engine does not fail during rendering.
**Tosca Module Path:** SAP/S4HANA_BR/MM/OutputControl/ADS_PDF_Verification
**Priority:** Medium
**Test Type:** Integration

---

### Test Case 3.1: Inactive Record in ZTXA_XREF Triggers Fallback Routine
**Tosca TestCase ID:** TC_STORY_003-9962_EDG_01
**Preconditions:**
- Record exists in ZTXA_XREF for EKORG = '2010' and BSART = 'NB', but field ACTIVE is explicitly set to ' ' (False/Inactive).
- Form 'Z_CUSTOM_2010' and Logo 'LOGO_2010' configured in that inactive row.

**Test Steps:**
1. Validate table ZTXA_XREF via SE16N to confirm entry EKORG '2010' has ACTIVE = ' '.
2. Create Purchase Order in EKORG '2010' with BSART 'NB'.
3. Save PO and trigger output processing.
4. Access transaction ME23N Output Items view.
5. Verify that system does NOT use 'Z_CUSTOM_2010' or 'LOGO_2010'.
6. Validate that output item adopted 'Z_STANDARD_PO_FORM' and 'DEFAULT_LOGO'.
7. Verify SLG1 contains warning entry for EKORG '2010' stating mapping is inactive.

**Expected Results:**
1. Database row verified as inactive.
2. PO created successfully without error.
3. BAdI filters out rows where ACTIVE <> 'X'.
4. Output items view reflects fallback configuration.
5. Custom assets from inactive row are strictly ignored.
6. SLG1 application log records appropriate inactive-mapping warning message.

**Test Data Sheet:**
| Parameter | Value | Description |
| :--- | :--- | :--- |
| EKORG | 2010 | Purchasing Org with Deactivated Row |
| BSART | NB | Standard Order Type |
| ACTIVE_FLAG | ' ' | Deactivated Indicator |
| EXPECTED_FORM | Z_STANDARD_PO_FORM | Default Layout Fallback |
| EXPECTED_LOGO | DEFAULT_LOGO | Default Logo Fallback |

---

### Test Case 3.2: Special Character Item Description Payload Rendered in Dynamic Form
**Tosca TestCase ID:** TC_STORY_003-9962_EDG_02
**Preconditions:**
- Active mapping in ZTXA_XREF for EKORG = '1010' and BSART = 'NB'.
- RFC connection to ADS active.

**Test Steps:**
1. Execute transaction ME21N for EKORG '1010' and BSART 'NB'.
2. Add line item with free-text description containing UTF-8/XML control symbols: `Test <B&W> Material "Limit" & Spec #100% ~ [Euro €]`.
3. Save PO and allow dynamic output item generation with 'Z_FORM_SRV_PO'.
4. Open transaction APOC_DISPLAY_ITEMS or ME23N and select 'Display PDF Document'.
5. Validate that PDF renders without ADS parsing error (RFC_ERROR_SYSTEM_FAILURE / XML parser crash).
6. Inspect rendered PDF stream to confirm special characters are displayed intact.

**Expected Results:**
1. ME21N accepts special characters in item short text.
2. Document saves and output determination assigns 'Z_FORM_SRV_PO'.
3. CL_APOC_OUTPUT_API serializes data without XML escaping defects.
4. ADS compiles the Adobe Form and returns binary PDF stream.
5. PDF preview displays string `Test <B&W> Material "Limit" & Spec #100% ~ [Euro €]` without truncation.

**Test Data Sheet:**
| Parameter | Value | Description |
| :--- | :--- | :--- |
| EKORG | 1010 | Standard Purchasing Org |
| BSART | NB | Document Type |
| TXZ01 | Test <B&W> Material "Limit" & Spec #100% ~ [Euro €] | Short Text with Boundary XML Symbols |
| EXPECTED_STATUS | Completed | Output Processing Status |
| ADS_RFC_DEST | ADS | RFC Destination to Adobe Server |


*Status: Approved and mapped by Tosca Automation Engineer.*

---

## [STORY-004-9962] - Supplier Master Business Partner EDI Communication Preference Resolution and Exception Handling
* **Epic Connection:** Automated Document Dispatch Framework

### Automated TestCase Details
# Test Scenario 1: Positive Flow (Happy Path) - BP FLVN01 EDI Communication Preference Resolution

**Description:** Validates that when a purchase order is committed for a Business Partner with role FLVN01 containing communication preference REC_COMM_TYPE = 'EDI' and valid WE20 partner profile configuration, S/4HANA APOC output channel determination overrides the default channel to 'EDI' and successfully dispatches an electronic XML payload.
**Tosca Module Path:** SAP/MM/Purchasing/PurchaseOrder/OutputControl
**Priority:** Critical
**Test Type:** E2E

---

### Test Case 1.1: Standard PO Creation with Automatic EDI Channel Dispatch for Role FLVN01
- **Tosca TestCase ID:** TC_STORY_004-9962_POS_01
- **Preconditions:**
  - User assigned authorization roles: SAP_BR_BUPA_MASTER_SPECIALIST, SAP_BR_PURCHASER.
  - Authorization objects: M_BEST_BSA (ACTVT=01, BSA=NB), B_BUPA_RLT (RLTYP=FLVN01), S_TABU_DIS (DICBERCLS=MD).
  - Vendor 'V10023' created with BP Role FLVN01 in Purchasing Org '1010' with REC_COMM_TYPE = 'EDI'.
  - Partner Profile configured in WE20 for Partner V10023, Type LI, Message Type ORDERS, Process Code ME10.
- **Test Steps:**
  1. Launch SAP GUI and navigate to transaction code ME21N (Create Purchase Order).
  2. Enter Order Type 'NB', Vendor 'V10023', Purchasing Org '1010', Purchasing Group '001', and Company Code '1010'.
  3. Add line item: Material 'RM-101', Plant '1010', Quantity '100', Net Price '45.00', Delivery Date '+5 days'. Buffer Purchase Order Number from status bar.
  4. Navigate to Header -> Messages / Output tab to inspect preliminary determination parameters before save.
  5. Click 'Save' to commit PO and trigger synchronous APOC dispatch execution.
  6. Execute transaction code ME23N with buffered PO, navigate to Header -> Output, and inspect channel determination and dispatch status.
  7. Execute transaction code WE02 / WE05 to verify generation of outbound IDoc/XML payload.
- **Expected Results:**
  1. Transaction ME21N opens with editable purchase order document template.
  2. Header data populates without error; vendor master reads BP role FLVN01 correctly.
  3. Line item processes and line totals calculate accurately.
  4. Output determination rule ZCL_STP_PO_LIMIT_OUTPUT=>DETERMINE_OUTPUT_CHANNEL sets Channel to 'EDI'.
  5. Purchase Order is saved with a generated document number; no update termination occurs.
  6. Output item displays Channel 'EDI', Dispatch Status '1' (Successfully Processed), Program 'CL_APOC_OUTPUT_API'.
  7. Outbound IDoc (ORDERS05) created with Status '03' (Data passed to port) referencing target partner V10023.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE | ME21N | S/4HANA PO Creation Transaction |
| BSART | NB | Standard Purchase Order Document Type |
| LIFNR | V10023 | Supplier BP Number with FLVN01 role |
| EKORG | 1010 | Purchasing Organization |
| EKGRP | 001 | Purchasing Group |
| BUKRS | 1010 | Company Code |
| MATNR | RM-101 | Standard Raw Material |
| REC_COMM_TYPE | EDI | Business Partner FLVN01 communication preference |
| OUTPUT_CHANNEL | EDI | Expected APOC Target Channel |
| STATUS_CODE | 1 | APOC Output Successfully Processed |

---

### Test Case 1.2: BP FLVN01 Communication Preference Update and Output Re-determination Verification
- **Tosca TestCase ID:** TC_STORY_004-9962_POS_02
- **Preconditions:**
  - Business Partner 'V10024' configured initially with REC_COMM_TYPE = 'LET' (Letter) or 'PRT' (Printer).
  - Purchasing Org '1010' active for supplier.
  - WE20 EDI profile pre-established for V10024.
- **Test Steps:**
  1. Launch transaction BP, switch to role FLVN01, and navigate to Purchasing Org '1010' Vendor Data.
  2. Update Default Communication Method / Output Preference to 'EDI' (REC_COMM_TYPE = 'EDI') and Save.
  3. Verify table LFM1 via SE16N to confirm database update of supplier purchasing data.
  4. Launch ME21N and create purchase order for Vendor 'V10024'.
  5. Save Purchase Order and buffer document number.
  6. Inspect output determination via transaction ME22N/ME23N.
- **Expected Results:**
  1. BP transaction loads partner V10024 in change mode.
  2. Preference saved successfully with message 'Changes have been saved'.
  3. Table LFM1 confirms field REC_COMM_TYPE = 'EDI' for Vendor V10024.
  4. PO created without field validation errors.
  5. Document committed to database without APOC dispatch runtime errors.
  6. Output channel automatically routes to 'EDI' matching updated FLVN01 preference.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| TCODE_BP | BP | S/4HANA Business Partner Maintenance |
| BP_ROLE | FLVN01 | BP Role Supplier - Purchasing Org Data |
| PARTNER_ID | V10024 | Supplier Master Business Partner ID |
| FIELD_UPDATE | REC_COMM_TYPE | Communication preference attribute |
| FIELD_VAL | EDI | New preference value |
| TCODE_PO | ME21N | Purchase Order Creation |
| VERIFY_CHAN | EDI | Confirmed APOC Channel Override |

---

# Test Scenario 2: Negative/Error Flow - Fallback to SMTP Email on Missing or Defective EDI Profile

**Description:** Validates that when a purchase order is committed for a Business Partner with role FLVN01 containing communication preference REC_COMM_TYPE = 'EDI', but the WE20 partner profile is missing, deactivated, or defective, the system intercepts CX_APOC_DISPATCH_FAILURE, executes fallback routing to standard SMTP email, writes warning message ZTX_STP_MSG 005 to application log SLG1, and sets output status to 'To Be Processed (Manual Review)'.
**Tosca Module Path:** SAP/MM/Purchasing/PurchaseOrder/ExceptionHandling
**Priority:** High
**Test Type:** Integration

---

### Test Case 2.1: Missing Partner Profile in WE20 Triggers Automated Email Fallback & Warning 005
- **Tosca TestCase ID:** TC_STORY_004-9962_NEG_01
- **Preconditions:**
  - Vendor 'V10099' maintained with BP role FLVN01 and REC_COMM_TYPE = 'EDI'.
  - Valid default Email address 'orders@vendor99.com' maintained in BP General Data (BAPI_BUPA_ADDRESS).
  - WE20 partner profile for 'V10099' is intentionally deleted or non-existent.
  - Authorization objects: S_APPL_LOG (ACTVT=03, ALG_OBJECT=ZTX_STP), M_BEST_BSA.
- **Test Steps:**
  1. Launch ME21N and enter Vendor 'V10099', Purchasing Org '1010', Purchasing Group '001', Company Code '1010'.
  2. Enter PO line item with Material 'SRV-01', Quantity '10', Net Price '120.00'.
  3. Commit PO by clicking 'Save'. Capture generated PO Number.
  4. Launch transaction ME23N for the created PO and inspect Header -> Output.
  5. Verify Channel change, Output Status, and Processing Log.
  6. Open transaction SLG1; query Object 'ZTX_STP', Subobject 'DISPATCH', External ID matching PO Number.
  7. Verify message class and message number in SLG1 details.
- **Expected Results:**
  1. ME21N populates header data correctly without syntax blocks.
  2. Line item entry completes successfully.
  3. PO saves successfully; APOC engine intercepts missing WE20 profile exception.
  4. Output channel displays fallback to 'EMAIL' with recipient address 'orders@vendor99.com'.
  5. Output Status is set to 'To Be Processed (Manual Review)' (Status indicator Amber/2).
  6. SLG1 application log displays entry under Object ZTX_STP.
  7. Application log records Warning Message: 'ZTX_STP_MSG 005: EDI partner profile missing; falling back to SMTP email'.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| LIFNR | V10099 | Vendor BP with EDI preference but no WE20 profile |
| BP_EMAIL | orders@vendor99.com | Default SMTP recipient address |
| WE20_EXISTS | FALSE | Partner profile flag in WE20 |
| FALLBACK_CHAN | EMAIL | Automated fallback channel |
| OUT_STATUS | 2 | Output Status 'To Be Processed (Manual Review)' |
| LOG_OBJECT | ZTX_STP | Application Log Object |
| MSG_CLASS | ZTX_STP_MSG | Custom Message Class |
| MSG_NUMBER | 005 | Warning message code for EDI fallback |

---

### Test Case 2.2: Defective WE20 Partner Profile Status Handled Gracefully with Email Fallback
- **Tosca TestCase ID:** TC_STORY_004-9962_NEG_02
- **Preconditions:**
  - Vendor 'V10088' exists with BP role FLVN01 and REC_COMM_TYPE = 'EDI'.
  - Partner Profile exists in WE20 for 'V10088', but status is set to 'I' (Inactive) or outbound process code is corrupted.
  - BP has active standard email address maintained.
- **Test Steps:**
  1. Launch transaction WE20, select Partner 'V10088', and toggle Outbound Parameter ORDERS to 'Deactivated'.
  2. Launch ME21N, create and save a new Purchase Order for Vendor 'V10088'.
  3. Buffer generated PO document number.
  4. Open ME22N, navigate to Header Output tab, and inspect Output Item Status and Log.
  5. Verify that output channel automatically switches from EDI to EMAIL.
  6. Query SLG1 with PO number to verify logging of exception CX_APOC_DISPATCH_FAILURE and warning ZTX_STP_MSG 005.
- **Expected Results:**
  1. WE20 reflects deactivated EDI status for partner V10088.
  2. ME21N saves purchase order document.
  3. PO document number is successfully buffered in Tosca.
  4. Output dispatch engine detects disabled EDI route and catches runtime exception.
  5. Channel dynamically switches to 'EMAIL' and status reflects 'To Be Processed (Manual Review)'.
  6. Application log registers warning ZTX_STP_MSG 005 indicating defective EDI profile routing.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| LIFNR | V10088 | Vendor BP with deactivated WE20 profile |
| WE20_STATUS | Inactive | Outbound parameter status in WE20 |
| FALLBACK_CHAN | EMAIL | Target fallback channel |
| LOG_MSG_CLASS | ZTX_STP_MSG | Message class evaluated |
| LOG_MSG_NR | 005 | Warning output logged in SLG1 |
| EXP_STATUS | 2 | To Be Processed (Manual Review) |

---

# Test Scenario 3: Boundary/Edge Case Validation - Communication Attributes & Fallback Constraints

**Description:** Validates boundary conditions and edge cases including: null/unconfigured REC_COMM_TYPE defaulting to standard email channel, and compound failures where both EDI profile and vendor email address are defective, ensuring system integrity and definitive error capture.
**Tosca Module Path:** SAP/MM/Purchasing/PurchaseOrder/EdgeCases
**Priority:** Medium
**Test Type:** Functional

---

### Test Case 3.1: Null or Blank Communication Preference Defaults to Standard Email
- **Tosca TestCase ID:** TC_STORY_004-9962_BND_01
- **Preconditions:**
  - Vendor 'V10050' configured with BP Role FLVN01 in Org '1010' with REC_COMM_TYPE = '' (SPACE / Null).
  - Default SMTP email address exists in BP general address data.
- **Test Steps:**
  1. Verify table LFM1 via SE16N for LIFNR = 'V10050' and confirm REC_COMM_TYPE is initial/blank.
  2. Launch ME21N and enter header details for Vendor 'V10050', Purchasing Org '1010'.
  3. Enter line item with valid Material and Quantity; save document.
  4. Open ME23N to display output determination details.
  5. Check output item channel, transmission medium, and processing status.
- **Expected Results:**
  1. Database check confirms REC_COMM_TYPE is empty.
  2. ME21N creates PO successfully without channel determination error.
  3. Document saves and document number is generated.
  4. APOC logic evaluates blank REC_COMM_TYPE and assigns default channel 'EMAIL' without triggering EDI override.
  5. Output channel is 'EMAIL', status is '1' (Processed) or '0' (In Preparation), and no fallback warning 005 is logged.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| LIFNR | V10050 | Vendor BP with blank communication preference |
| REC_COMM_TYPE | NULL | Unset communication preference in LFM1 |
| DEFAULT_CHAN | EMAIL | Standard default APOC output channel |
| EXPECTED_WARN | NONE | No warning message expected in SLG1 |
| OUTPUT_STATUS | 1 | Successfully processed standard email |

---

### Test Case 3.2: Compound Failure - Defective EDI Profile with Malformed Fallback Email Address
- **Tosca TestCase ID:** TC_STORY_004-9962_BND_02
- **Preconditions:**
  - Vendor 'V10077' configured with BP Role FLVN01 and REC_COMM_TYPE = 'EDI'.
  - WE20 partner profile is missing.
  - Vendor BP general address data contains a malformed or invalid SMTP address (e.g., 'invalid-email-format@@vendor..com').
- **Test Steps:**
  1. Launch ME21N, enter Vendor 'V10077', Purchasing Org '1010', Purchasing Group '001', Company Code '1010'.
  2. Add line item: Material 'RM-101', Quantity '50', Net Price '15.00'.
  3. Save purchase order.
  4. Open ME23N and inspect output processing status and channel.
  5. Execute transaction SLG1 to inspect all logged entries for the generated PO document.
  6. Validate that primary EDI failure warning 005 is recorded, followed by secondary dispatch validation error.
- **Expected Results:**
  1. Document parameters populated in ME21N.
  2. Line item added successfully.
  3. Purchase Order commits to database; APOC engine triggers fallback from EDI to EMAIL.
  4. Output channel shows 'EMAIL', but Output Status reflects '3' (Error) due to malformed recipient address.
  5. Application log SLG1 contains warning ZTX_STP_MSG 005 confirming fallback initiation.
  6. Application log SLG1 captures subsequent email validation error CX_APOC_DISPATCH_FAILURE without S/4HANA core dump.
- **Test Data Sheet:**

| Parameter | Value | Description |
| :--- | :--- | :--- |
| LIFNR | V10077 | Vendor BP with EDI preference and malformed email |
| REC_COMM_TYPE | EDI | Maintained communication preference in FLVN01 |
| WE20_EXISTS | FALSE | EDI partner profile missing |
| SMTP_ADDR | invalid-email-format@@vendor..com | Malformed email in BP Master |
| EXPECTED_STATUS | 3 | Output Status Error / Transmission Failed |
| LOG_WARN_MSG | ZTX_STP_MSG 005 | Fallback triggered warning |
| SEC_LOG_ERR | CX_APOC_DISPATCH_FAILURE | Malformed address dispatch error |


*Status: Approved and mapped by Tosca Automation Engineer.*

---

