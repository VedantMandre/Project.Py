```
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
                   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
                   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.4.xsd">

    <!-- Bank Details Table -->
    <changeSet id="create-bank-details-table" author="developer">
        <sql>
CREATE TABLE IF NOT EXISTS payment.bank_details (
    bank_id BIGINT NOT NULL PRIMARY KEY,
    name VARCHAR NOT NULL,
    identifier VARCHAR,
    department VARCHAR,
    street_name VARCHAR,
    building_number VARCHAR,
    building_name VARCHAR,
    floor VARCHAR,
    po_box VARCHAR,
    postal_code VARCHAR,
    city VARCHAR NOT NULL,
    town_location VARCHAR,
    district VARCHAR,
    state_province VARCHAR,
    country VARCHAR NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_by VARCHAR,
    created_at TIMESTAMP DEFAULT now(),
    updated_by VARCHAR,
    updated_at TIMESTAMP DEFAULT now()
);
        </sql>
    </changeSet>

    <!-- Beneficiary Details Table -->
    <changeSet id="create-beneficiary-details-table" author="developer">
        <sql>
CREATE TABLE IF NOT EXISTS payment.beneficiaries (
    beneficiary_id BIGINT NOT NULL PRIMARY KEY,
    bank_id BIGINT NOT NULL,
    branch_id VARCHAR,
    sun_id VARCHAR UNIQUE NOT NULL,
    name VARCHAR NOT NULL,
    account_number VARCHAR NOT NULL,
    organization VARCHAR,
    department VARCHAR,
    street_name VARCHAR,
    building_number VARCHAR,
    building_name VARCHAR,
    floor VARCHAR,
    po_box VARCHAR,
    postal_code VARCHAR,
    city VARCHAR NOT NULL,
    town_location VARCHAR,
    district VARCHAR,
    state_province VARCHAR,
    country VARCHAR NOT NULL,
    status VARCHAR NOT NULL,
    tags JSONB,
    comments VARCHAR,
    CONSTRAINT fk_payment_beneficiaries_bank_id FOREIGN KEY (bank_id) REFERENCES payment.bank_details(bank_id)
);
        </sql>
    </changeSet>
</databaseChangeLog>

```
```
bank_id BIGINT REFERENCES payment.beneficiary_bank(bank_id) NOT NULL
```
```
SELECT 
    otd.trade_number,
    otd.reference_number AS new_reference_number,
    otd.old_reference_number AS old_reference_number,
    otd.time_deposit_amount,
    otd.maturity_date,
    otd.currency,
    tdr.trade_number AS rollover_trade_number,
    tdr.reference_number AS rollover_reference_number,
    tdr.principal_amount,
    tdr.maturity_date AS rollover_maturity_date,
    tdr.currency_code AS rollover_currency
FROM 
    deposit.test_recon_obs_time_deposit_data otd
LEFT JOIN 
    deposit.test_recon_time_deposit_rollover tdr
ON 
    otd.old_reference_number = tdr.reference_number
WHERE 
    otd.old_reference_number IS NOT NULL;
```
```
WITH RolledOverTDs AS (
    SELECT 
        otd.trade_number AS new_trade_number,
        otd.reference_number AS new_reference_number,
        otd.old_reference_number AS old_reference_number,
        otd.time_deposit_amount AS new_time_deposit_amount,
        otd.maturity_date AS new_maturity_date,
        otd.currency AS new_currency,
        otd.interest_accrued_till_date AS new_interest_accrued,
        otd.interest_at_maturity AS new_interest_at_maturity,
        
        tdr.trade_number AS old_trade_number,
        tdr.reference_number AS rollover_reference_number,
        tdr.principal_amount AS old_principal_amount,
        tdr.maturity_date AS old_maturity_date,
        tdr.currency_code AS old_currency,
        tdr.accrued_interest AS old_accrued_interest,
        tdr.interest_amount AS old_interest_amount,
        tdr.maturity_status AS rollover_status
    FROM 
        deposit.test_recon_obs_time_deposit_data otd
    LEFT JOIN 
        deposit.test_recon_time_deposit_rollover tdr
    ON 
        otd.old_reference_number = tdr.reference_number
    WHERE 
        otd.old_reference_number IS NOT NULL
)
UPDATE deposit.test_recon_time_deposit_rollover tdr
SET maturity_status = 'Finalized'
FROM RolledOverTDs rtd
WHERE tdr.reference_number = rtd.old_reference_number
AND tdr.maturity_status <> 'Finalized';  -- Only update if not already finalized

```
```
SELECT 
    otd.trade_number,
    otd.reference_number AS new_reference_number,
    otd.old_reference_number,
    otd.time_deposit_amount,
    otd.maturity_date,
    otd.currency,
    tdr.reference_number AS matched_reference_number,
    tdr.maturity_status,
    -- Mark as Finalized if old_reference_number matches reference_number
    CASE 
        WHEN otd.old_reference_number = tdr.reference_number THEN 'Finalized' 
        ELSE tdr.maturity_status 
    END AS updated_maturity_status
FROM deposit.test_recon_obs_time_deposit_data otd
LEFT JOIN deposit.test_recon_time_deposit_rollover tdr
    ON otd.old_reference_number = tdr.reference_number
WHERE otd.old_reference_number IS NOT NULL;
```
```
WITH updated_rows AS (
    UPDATE deposit.test_recon_time_deposit_rollover tdr
    SET 
        trade_number = otd.trade_number,
        principal_amount = otd.time_deposit_amount,
        maturity_date = otd.maturity_date,
        currency_code = otd.currency,
        accrued_interest = otd.interest_accrued_till_date,
        interest_amount = otd.interest_at_maturity,
        branch_code = otd.branch,
        funding_source = otd.funding_source,
        obs_number = otd.obs_code,
        account_number = otd.time_deposit_account_number,
        settlement_account = otd.settlement_account_number,
        maturity_status = otd.maturity_status, -- Keep original status
        status = 'Finalized'
    FROM deposit.test_recon_obs_time_deposit_data otd
    WHERE tdr.reference_number = otd.old_reference_number
    RETURNING tdr.reference_number
)
INSERT INTO deposit.test_recon_time_deposit_rollover (
    trade_number, reference_number, principal_amount, 
    maturity_date, currency_code, accrued_interest, interest_amount, 
    branch_code, funding_source, obs_number, account_number, 
    settlement_account, maturity_status, status
)
SELECT 
    otd.trade_number,
    otd.old_reference_number,  -- Track only the old reference number
    otd.time_deposit_amount,
    otd.maturity_date,
    otd.currency,
    otd.interest_accrued_till_date,
    otd.interest_at_maturity,
    otd.branch,
    otd.funding_source,
    otd.obs_code,
    otd.time_deposit_account_number,
    otd.settlement_account_number,
    otd.maturity_status,
    'Finalized' AS status  -- Mark rolled-over TDs as Finalized
FROM deposit.test_recon_obs_time_deposit_data otd
WHERE otd.old_reference_number IS NOT NULL
AND NOT EXISTS (
    SELECT 1 FROM updated_rows ur WHERE ur.reference_number = otd.old_reference_number
);

```
