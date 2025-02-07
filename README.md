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

#### **Payment Fields:**
- `DEBIT_ACCOUNT_CURRENCY_CODE`
- `DEBIT_ACCOUNT_NUMBER`
- `DELIVER_CURRENCY_CODE`
- `DETAILS_OF_PAYMENT_LINE1` - `DETAILS_OF_PAYMENT_LINE4`
- `DETAILS_OF_CHARGES`
- `CNAPS_CODE`
- `PURPOSE_CODE`
- `TRANSACTION_AS_IN_IMPFILE`
- `PRE_ADVICE`
- `WIRE_LIMIT`
- `GUARANTEE_OUR`

---

### **3. GCMSOWNER.WIRE_PAYEE_IMP_PARAMS**
#### **Beneficiary Fields:**
- `IS_BENEFICIARYBANK_POPULATED`

#### **Payment Fields:**
- `PAYEE_TEMPLATE_ID`
- `HAS_SPECIAL_CHARS`
- `CHARGES_UPDATED`
- `IS_AMBIGUOUS_ACCOUNT_NUMBER`
- `FAILURE_KEY`

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

---

### **5. GCMSOWNER.GCMS_WIRE_AMENDMENT**
#### **Payment Details:**
- `DEBIT_ACCOUNT_NUMBER`
- `FUNDS_TYPE`
- `VALUE_DATE`
- `DETAILS_OF_CHARGES`
- `OUR_REFERENCE`
- `DETAILS_OF_PAYMENT_LINE1` - `DETAILS_OF_PAYMENT_LINE4`

#### **Beneficiary Details:**
- `BENEF_CUST_NAME`
- `BENEF_CUST_ID`
- `BENEF_CUST_ADDRESS_LINE1 - LINE3`

#### **Other Key Details:**
- `BY_ORDER_OF_CUST_NAME`
- `BY_ORDER_OF_CUST_ADDR1 - ADDR3`
- `BENEF_INN`
- `BENEF_BANK_RU_CODE`

---

### **6. GCMSOWNER.GCMS_WIRE_AUTHORISE_DETAIL**
- No direct payment or beneficiary fields.
- Contains approval-related metadata.

---

### **7. GCMSOWNER.GCMS_WIRE_AUTHORISE_MASTER**
- No direct payment or beneficiary fields.
- Contains authorization-related metadata.

---

### **8. GCMSOWNER.GCMS_WIRE_CURRENCY_CATEGORY**
#### **Payment Details:**
- `CURRENCY`

#### **Beneficiary Details:**
- None

---

### **9. GCMSOWNER.GCMS_WIRE_TFR_PAYEE_AUDIT**
#### **Payment Fields:**
- `FROM_ACCOUNT`
- `FROM_CURRENCY_CODE`
- `DELIVER_CURRENCY_CODE`
- `FUNDS_TYPE`
- `CHARGES_CODE`
- `DETAILS_OF_PAYMENT_LINE1` - `DETAILS_OF_PAYMENT_LINE3`
- `BANK_TO_BANK_LINE1` - `BANK_TO_BANK_LINE3`

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

#### **Other Key Details:**
- `BY_ORDER_OF_NAME`
- `BY_ORDER_OF_ADDRESS1 - ADDRESS3`
- `RECV_CORR_NAME`
- `RECV_CORR_ADDRESS_LINE1 - LINE3`
- `TRANSACTION_REFERENCE_NUMBER`
- `WIRE_LIMIT`
- `CNAPS_CODE`
- `BUSINESS_CATEGORY`

---

## **Key Findings**
### **Payment Information:**
1. **Transaction References:**  
   - `TRANSACTION_ID`, `AMEND_TRANSACTION_ID`, `TRANSACTION_REFERENCE_NUMBER`
2. **Account and Currency Details:**  
   - `FROM_ACCOUNT`, `DEBIT_ACCOUNT_NUMBER`, `DEBIT_ACCOUNT_CURRENCY_CODE`, `DELIVER_CURRENCY_CODE`
3. **Charges and Limits:**  
   - `DETAILS_OF_CHARGES`, `CHARGES_CODE`, `WIRE_LIMIT`, `GUARANTEE_OUR`
4. **Compliance and Routing:**  
   - `CNAPS_CODE`, `PURPOSE_CODE`, `PRE_ADVICE`
5. **Payment Instructions:**  
   - `DETAILS_OF_PAYMENT_LINE1` - `DETAILS_OF_PAYMENT_LINE4`

---

### **Beneficiary Information:**
1. **Core Beneficiary Info:**  
   - `PARTY_NAME`, `PARTY_ID`, `PARTY_ID_TYPE`, `PAYEE_TEMPLATE_NAME`, `BENEF_NAME`
2. **Address and Location:**  
   - `STREET_NAME`, `BUILDING_NO`, `POST_CODE`, `COUNTRY_CODE`, `BENEF_ADDRESS_LINE1` - `LINE3`
3. **Bank Details:**  
   - `BENEF_BANK_ACCOUNT_NUMBER`, `BENEF_BANK_RU_CODE`, `IS_BENEFICIARYBANK_POPULATED`
4. **Compliance Info:**  
   - `PARTY_LEI`, `BENEF_INN`
