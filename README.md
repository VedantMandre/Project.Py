# **Payment and Beneficiary Data Discovery**

## **Overview**
This document outlines the payment and beneficiary details available in on-prem Oracle DB tables. It supports the Jira story for discovering data sources needed to extract, process, and send account information to the navigation page.

---

## **Objective**
As a microservices developer, I need to:
- Identify available data in CMS and external systems.
- Extract relevant data from correct sources.
- Display the extracted data on the navigation page.

---

## **Assumptions**
- The microservices team will query the category of accounts to be displayed.
- The business team will confirm the list of source systems.

---

## **Data Breakdown by Table**

### **1. GCMSOWNER.WIRE_AMEND_PARTY_DETAIL**
#### **Beneficiary Fields:**
- `PARTY_NAME`
- `PARTY_ID`
- `PARTY_ID_TYPE`
- `PARTY_LEI`
- `PARTY_BRANCH_ID`
- `DEPARTMENT_NAME`, `SUB_DEPARTMENT`
- `STREET_NAME`, `BUILDING_NO`, `BUILDING_NAME`, `FLOOR`, `POST_BOX`, `ROOM`
- `POST_CODE`, `TOWN_NAME`, `TOWN_LOCATION_NAME`, `DISTRICT_NAME`, `COUNTRY_SUB_DIVISION`, `COUNTRY_CODE`

#### **Sample Data:**
- PARTY_LEI:  
  - `SMBCHY5`  
  - `SMBCNY645P`  
  - `SMBCGB2P`  
  - `ABC12312312312312398`  
  - `01234567890123456789`  
  - `77777777777777777777`
- DEPARTMENT_NAME, SUB_DEPARTMENT:  
  - `Payee Department, Payee Sub department`  
  - `Dept A, Dept AB`

#### **Payment Fields:**
- `TRANSACTION_ID`
- `AMEND_TRANSACTION_ID`
- `PARTY_LOOKUP_TYPE_CODE`

---

### **2. GCMSOWNER.WIRE_PAYEE_IMP_DETAIL**
#### **Beneficiary Fields:**
- `BENEF_INN`
- `BENEF_BANK_RU_CODE`
- `PAYEE_TEMPLATE_NAME`

#### **Sample Data:**
- BENEF_INN:  
  - `INN123`  
  - `INN324`
- BENEF_BANK_RU_CODE:  
  - `BENRU1212`  
  - `BENRU3434`

#### **Payment Fields:**
- `DEBIT_ACCOUNT_CURRENCY_CODE`
- `DEBIT_ACCOUNT_NUMBER`
- `DELIVER_CURRENCY_CODE`
- `DETAILS_OF_PAYMENT_LINE1 - DETAILS_OF_PAYMENT_LINE4`
- `DETAILS_OF_CHARGES`
- `CNAPS_CODE`
- `PURPOSE_CODE`
- `TRANSACTION_AS_IN_IMPFILE`
- `PRE_ADVICE`
- `WIRE_LIMIT`
- `GUARANTEE_OUR`

#### **Sample Data:**
- DEBIT_ACCOUNT_CURRENCY_CODE:  
  - `USD`  
  - `EUR`  
  - `CNY`
- DETAILS_OF_CHARGES:  
  - `Vendor Payment`  
  - `Invoice Payment`  
  - `Supplier`
- PRE_ADVICE:  
  - `Y`  
  - `N`

---

### **3. GCMSOWNER.WIRE_PAYEE_IMP_PARAMS**
#### **Beneficiary Fields:**
- `IS_BENEFICIARYBANK_POPULATED`

#### **Sample Data:**
- IS_BENEFICIARYBANK_POPULATED:  
  - `Yes`  
  - `No`

#### **Payment Fields:**
- `PAYEE_TEMPLATE_ID`
- `HAS_SPECIAL_CHARS`
- `CHARGES_UPDATED`
- `IS_AMBIGUOUS_ACCOUNT_NUMBER`
- `FAILURE_KEY`

#### **Sample Data:**
- HAS_SPECIAL_CHARS:  
  - `Y`  
  - `N`
- CHARGES_UPDATED:  
  - `Yes`  
  - `No`

---

### **4. GCMS_WIRE_TRANSFER_PAYEE**
#### **Payment Fields:**
- `WIRE_TRANSFER_TEMPLATE_ID`
- `FROM_ACCOUNT`
- `FROM_CURRENCY_CODE`
- `DELIVER_CURRENCY_CODE`
- `FUNDS_TYPE`
- `CHARGES_CODE`
- `TRANSACTION_REFERENCE_NUMBER`
- `WIRE_LIMIT`
- `PRE_ADVICE`

#### **Sample Data:**
- FROM_ACCOUNT:  
  - `689515`  
  - `287669`
- FROM_CURRENCY_CODE:  
  - `JPY`  
  - `GBP`  
  - `USD`
- FUNDS_TYPE:  
  - `B`  
  - `M`  
  - `S`  
  - `I`

#### **Beneficiary Fields:**
- `PAYEE_TEMPLATE_ID`
- `BENEF_ACCOUNT_NUMBER`
- `BENEF_NAME`
- `BENEF_ADDRESS_LINE1/LINE2/LINE3`
- `BENEF_BANK_ACCOUNT_NUMBER`
- `BENEF_BANK_FUNDS_TYPE`
- `BENEF_BANK_RU_CODE`
- `IBLC_COUNTRY_CODE`
- `CATEGORY_PURPOSE`
- `PURPOSE_CODE`
- `CNAPS_CODE`

#### **Sample Data:**
- BENEF_BANK_FUNDS_TYPE:  
  - `S`  
  - `ACCOUNT`

---

### **5. GCMSOWNER.GCMS_WIRE_AMENDMENT**
#### **Payment Fields:**
- `DEBIT_ACCOUNT_NUMBER`
- `FUNDS_TYPE`
- `VALUE_DATE`
- `DETAILS_OF_CHARGES`
- `OUR_REFERENCE`
- `DETAILS_OF_PAYMENT_LINE1 - DETAILS_OF_PAYMENT_LINE4`

#### **Sample Data:**
- VALUE_DATE:  
  - `01-JAN-19 12.00.00.000000000 AM`  
  - `03-JUL-24 11.00.00.000000000 AM`
- DETAILS_OF_CHARGES:  
  - `OUR`  
  - `BEN`
- OUR_REFERENCE:  
  - `Testing123`  
  - `POP PXC`

---

### **9. GCMSOWNER.GCMS_WIRE_TFR_PAYEE_AUDIT**
#### **Payment Fields:**
- `FROM_ACCOUNT`
- `FROM_CURRENCY_CODE`
- `DELIVER_CURRENCY_CODE`
- `FUNDS_TYPE`
- `CHARGES_CODE`
- `DETAILS_OF_PAYMENT_LINE1 - DETAILS_OF_PAYMENT_LINE3`
- `BANK_TO_BANK_LINE1 - BANK_TO_BANK_LINE3`

#### **Sample Data:**
- FROM_ACCOUNT:  
  - `151515`  
  - `317074`
- DETAILS_OF_PAYMENT_LINE1:  
  - `Invoice Payment`  
  - `Supplier Payment`

#### **Beneficiary Fields:**
- `BENEF_ID`
- `BENEF_ACCOUNT_NUMBER`
- `BENEF_IBAN`
- `BENEF_NAME`
- `BENEF_ADDRESS_LINE1 - LINE3`
- `BENEF_BANK_ID`
- `BENEF_BANK_ACCOUNT_NUMBER`
- `BENEF_BANK_NAME`
- `BENEF_BANK_ADDRESS_LINE1 - LINE3`

#### **Sample Data:**
- BENEF_NAME:  
  - `LN Account Title 1 OEAITCERPY`  
  - `NY Account Title 2 SSTEIUGESL`
- RECV_CORR_NAME:  
  - `JPMORGAN CHASE BANK, LA`  
  - `CHIPS TEST ABA`
