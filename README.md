# Payment and Beneficiary Data Discovery

## **Overview**
This document outlines the payment and beneficiary details available in on-prem Oracle DB tables. It supports the Jira story for discovering data sources needed to extract, process, and send account information to the navigation page.

## **Objective**
As a microservices developer, I need to:
- Identify available data in CMS and external systems.
- Extract relevant data from correct sources.
- Display the extracted data on the navigation page.

## **Assumptions**
- The microservices team will query the category of accounts to be displayed.
- The business team will confirm the list of source systems.

## **Data Breakdown by Table**

### **1. GCMSOWNER.GCMS_WIRE_AMENDMENT**
#### **Payment Details:**
- `DEBIT_ACCOUNT_NUMBER`
- `FUNDS_TYPE`
- `VALUE_DATE`
- `DETAILS_OF_CHARGES`
- `OUR_REFERENCE`
- `DETAILS_OF_PAYMENT_LINE1 - LINE4`

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

### **2. GCMSOWNER.GCMS_WIRE_AUTHORISE_DETAIL**
#### **Payment Details:**
- No direct payment fields.
#### **Beneficiary Details:**
- No direct beneficiary fields.
#### **Other Key Details:**
- Approval-related metadata.

---

### **3. GCMSOWNER.GCMS_WIRE_AUTHORISE_MASTER**
#### **Payment Details:**
- No direct payment fields.
#### **Beneficiary Details:**
- No direct beneficiary fields.
#### **Other Key Details:**
- Authorization-related metadata.

---

### **4. GCMSOWNER.GCMS_WIRE_CURRENCY_CATEGORY**
#### **Payment Details:**
- `CURRENCY`
#### **Beneficiary Details:**
- No direct beneficiary fields.

---

### **5. GCMSOWNER.GCMS_WIRE_TFR_PAYEE_AUDIT**
#### **Payment Details:**
- `FROM_ACCOUNT`
- `FROM_CURRENCY_CODE`
- `DELIVER_CURRENCY_CODE`
- `FUNDS_TYPE`
- `CHARGES_CODE`
- `DETAILS_OF_PAYMENT_LINE1 - LINE3`
- `BANK_TO_BANK_LINE1 - LINE3`

#### **Beneficiary Details:**
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

## **Next Steps**
- **Identify missing data:** Verify if additional tables contain supplementary details.
- **Confirm source systems:** Determine if CMS holds all required data or if integration with other systems is needed.
- **Define data flow:** Establish how data will be extracted, processed, and sent to the front end.


