# README: Payment & Beneficiary Data Discovery for On-Prem Oracle DB

## Description
As a **Microservices Developer**, I need to extract, process, and send account information to the frontend navigation page. This document provides an analysis of available data in **CMS** and identifies data that needs to be fetched from other systems.

## Assumptions
- The **Microservices Team** will query the required account categories.
- The **list of source systems** will be confirmed by the business team.

---

## Tables Analyzed

### 1. **GCMSOWNER.VIRE_TRANSFER_TEMPLATE**
**Relevant Fields:**
- `FROM_ACCOUNT`, `FROM_CURRENCY_CODE`, `DELIVER_CURRENCY_CODE` → Source and destination account details.
- `BENEF_ID`, `BENEF_ACCOUNT_NUMBER`, `BENEF_NAME`, `BENEF_BANK_ID`, `BENEF_BANK_ACCOUNT_NUMBER` → Beneficiary details.
- `DETAILS_OF_PAYMENT_LINE1-4` → Payment purpose and instructions.
- `BANK_TO_BANK_LINE1-3`, `BANK_TO_BANK_CODE1-6`, `BANK_TO_BANK_VALUE1-6` → Interbank transfer details.
- `TRANSACTION_ID`, `WIRE_LIMIT` → Transactional constraints.

### 2. **GCMSOWNER.WIRE_AMEND_PARTY_DETAIL**
**Relevant Fields:**
- `PARTY_TYPE`, `PARTY_NAME`, `PARTY_ID`, `PARTY_ID_TYPE` → Party information.
- `STREET_NAME`, `BUILDING_NO`, `POST_CODE`, `COUNTRY_CODE` → Address details.
- `CREATED_BY`, `CREATION_DATE`, `LAST_UPDATED_BY`, `LAST_UPDATED_DATE` → Audit trail.

### 3. **GCMSOWNER.WIRE_PAYEE_IMP_DETAIL**
**Relevant Fields:**
- `IMPORT_FILE_ID`, `IMPORT_FILE_STATUS`, `IMPORTED_BY` → Import file details.
- `DEBIT_ACCOUNT_NUMBER`, `DEBIT_ACCOUNT_CURRENCY_CODE`, `DELIVER_CURRENCY_CODE` → Transactional details.
- `DETAILS_OF_PAYMENT_LINE1-4` → Payment description.
- `PAYEE_TEMPLATE_ID`, `PAYEE_TEMPLATE_NAME`, `BENEF_INN`, `BENEF_BANK_RU_CODE` → Beneficiary details.
- `PRE_ADVICE`, `WIRE_LIMIT` → Transactional constraints.
- `PROCESSING_STATUS`, `ERROR_DESCRIPTION` → Error handling and validation.

### 4. **GCMSOWNER.WIRE_PAYEE_IMP_PARAMS**
**Relevant Fields:**
- `PAYEE_TEMPLATE_ID` → Template reference.
- `HAS_SPECIAL_CHARS`, `IS_AMBIGUOUS_ACCOUNT_NUMBER` → Data quality flags.
- `CHARGES_UPDATED`, `IS_BENEFICIARYBANK_POPULATED` → Payment processing indicators.

---

## Key Findings
### **Data Available in CMS:**
- **Beneficiary Details:** Name, account number, bank details, addresses.
- **Transaction Details:** Account numbers, currency codes, limits.
- **Payment Instructions:** Purpose of payment, interbank details.
- **Import & Processing Information:** Payee templates, error handling, validation statuses.

### **Data to Be Fetched from Other Systems:**
- **Regulatory Data:** Compliance codes, country-specific reporting.
- **Real-time Balances:** May need integration with a core banking system.
- **Cross-border Payment Data:** Additional enrichment required from external banking networks.

---

## Next Steps
1. **Confirm the source systems** required for missing data with business stakeholders.
2. **Validate data consistency** across tables and determine transformation requirements.
3. **Identify integration points** for fetching additional data.
4. **Develop microservices** to extract, process, and send required data to the frontend.

This structured analysis will serve as a foundation for the Confluence page and ensure alignment with the Jira story requirements.

