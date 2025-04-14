```
host=psql-cms-data-sbx-use2.postgres.database.azure.com;
port=5432;
database=postgres;
uid=adfuser;
pwd=YourSecurePassword;
encryptionmethod=1;
validateservercertificate=0;
```
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
CREATE OR REPLACE PROCEDURE deposit.sync_time_deposit_rollover()
LANGUAGE plpgsql
AS $$
DECLARE
    match_count INTEGER;
    updated_count INTEGER;
BEGIN
    -- Step 1: Debug - Count the number of matches
    SELECT COUNT(*)
    INTO match_count
    FROM deposit.test_recon_time_deposit_rollover tdr
    WHERE TRIM(tdr.reference_number) IN (
        SELECT TRIM(old_reference_number)
        FROM deposit.test_recon_obs_time_deposit_data
        WHERE old_reference_number IS NOT NULL
    );

    RAISE NOTICE 'Number of matches found: %', match_count;

    -- Step 2: Update status to 'Pending' for matching records
    UPDATE deposit.test_recon_time_deposit_rollover tdr
    SET status = 'Pending'
    WHERE TRIM(tdr.reference_number) IN (
        SELECT TRIM(old_reference_number)
        FROM deposit.test_recon_obs_time_deposit_data
        WHERE old_reference_number IS NOT NULL
    ) AND (tdr.status IS NULL OR tdr.status <> 'Pending');

    -- Step 3: Debug - Count the number of updated rows
    GET DIAGNOSTICS updated_count = ROW_COUNT;
    RAISE NOTICE 'Number of rows updated: %', updated_count;

    -- Step 4: Commit the changes
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        -- Roll back on error
        ROLLBACK;
        RAISE EXCEPTION 'Error occurred: %', SQLERRM;
END;
$$;
```
```
SELECT tdr.reference_number, tdr.status, otd.old_reference_number
FROM deposit.test_recon_time_deposit_rollover tdr
INNER JOIN deposit.test_recon_obs_time_deposit_data otd
ON otd.old_reference_number = tdr.reference_number
WHERE otd.old_reference_number IS NOT NULL;
```
```
UPDATE deposit.test_recon_time_deposit_rollover tdr
SET status = 'Finalized'
WHERE TRIM(tdr.reference_number) IN (
    SELECT TRIM(old_reference_number)
    FROM deposit.test_recon_obs_time_deposit_data
    WHERE old_reference_number IS NOT NULL
) AND (tdr.status IS NULL OR tdr.status <> 'Finalized');

```
```
UPDATE deposit.test_recon_time_deposit_rollover tdr
SET status = 'Pending'
WHERE TRIM(tdr.reference_number) IN (
    SELECT TRIM(old_reference_number)
    FROM deposit.test_recon_obs_time_deposit_data
    WHERE old_reference_number IS NOT NULL
) AND tdr.status = 'Finalized';

```
```
SELECT reference_number, status
FROM deposit.test_recon_time_deposit_rollover
WHERE status = 'Finalized';
```
```
CREATE OR REPLACE PROCEDURE deposit.sync_time_deposit_rollover()
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE deposit.test_recon_time_deposit_rollover tdr
    SET status = 'Finalized'
    WHERE tdr.reference_number IN (
        SELECT old_reference_number
        FROM deposit.test_recon_obs_time_deposit_data
        WHERE old_reference_number IS NOT NULL
    );
END;
$$;

UPDATE deposit.test_recon_time_deposit_rollover 
SET status = NULL 
WHERE status = 'Finalized';


SELECT reference_number, old_reference_number
FROM deposit.test_recon_time_deposit_rollover tdr
JOIN deposit.test_recon_obs_time_deposit_data obs
ON tdr.reference_number = obs.old_reference_number;

```
```
INSERT INTO deposit.test_recon_time_deposit_rollover (reference_number, status)
SELECT old_reference_number, 'Finalized'
FROM deposit.test_recon_obs_time_deposit_data
WHERE old_reference_number IS NOT NULL
ON CONFLICT (reference_number) 
DO UPDATE SET status = 'Finalized';



```
```
CREATE OR REPLACE PROCEDURE sp_upsert_finalize_status()
LANGUAGE plpgsql
AS $$
BEGIN
    -- Update status to 'Finalized' if reference number exists in the other table
    UPDATE deposit.test_recon_time_deposit_rollover tdr
    SET status = 'Finalized'
    WHERE tdr.reference_number IN (
        SELECT old_reference_number
        FROM deposit.test_recon_obs_time_deposit_data
        WHERE old_reference_number IS NOT NULL
    );

    -- If the reference_number does not exist in the main table, insert it with 'Finalized' status
    INSERT INTO deposit.test_recon_time_deposit_rollover (reference_number, status)
    SELECT old_reference_number, 'Finalized'
    FROM deposit.test_recon_obs_time_deposit_data
    WHERE old_reference_number IS NOT NULL
    ON CONFLICT (reference_number) DO NOTHING;
    
END;
$$;
```
```
CREATE OR REPLACE PROCEDURE sp_finalize_matched_records()
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE deposit.test_recon_time_deposit_rollover tdr
    SET status = 'Finalized'
    WHERE EXISTS (
        SELECT 1 
        FROM deposit.test_recon_obs_time_deposit_data obs
        WHERE obs.old_reference_number = tdr.reference_number
    )
    AND (tdr.status IS DISTINCT FROM 'Finalized'); -- Only update if it's not already 'Finalized'
END;
$$;
```
```
INSERT INTO deposit.test_recon_time_deposit_rollover (reference_number, status)
SELECT old_reference_number, 'Pending'  -- Setting initial status to 'Pending'
FROM deposit.test_recon_obs_time_deposit_data
WHERE old_reference_number IS NOT NULL
LIMIT 5;  -- Adjust the limit as needed
```
```
UPDATE deposit.test1_recon_time_deposit_rollover tdr
SET status = 'FINALIZED',
    updated_by = 'ADF_TD_RECONCILE',
    updated_at = CURRENT_TIMESTAMP -- Ensure the column name is correct
WHERE EXISTS (
    SELECT 1 
    FROM deposit.test_recon_obs_time_deposit_data obs
    WHERE obs.old_reference_number = tdr.reference_number
)
AND (tdr.status IS DISTINCT FROM 'FINALIZED');
```
```
UPDATE deposit.test1_recon_time_deposit_rollover tdr
SET 
    status = CASE 
        WHEN tdr.status IS DISTINCT FROM 'FINALIZED' THEN 'FINALIZED'
        ELSE tdr.status 
    END,
    updated_by = CASE 
        WHEN tdr.status IS DISTINCT FROM 'FINALIZED' THEN 'ADF_TD_RECONCILE'
        ELSE tdr.updated_by 
    END,
    updated_date = CASE 
        WHEN tdr.status IS DISTINCT FROM 'FINALIZED' THEN CURRENT_TIMESTAMP
        ELSE tdr.updated_date 
    END
WHERE EXISTS (
    SELECT 1 
    FROM deposit.test_recon_obs_time_deposit_data obs
    WHERE obs.old_reference_number = tdr.reference_number
)
AND tdr.status IS DISTINCT FROM 'FINALIZED';

```
```
INSERT INTO deposit.time_deposit_rollover (
    reference_number,
    account_no,
    deposit_amount,
    interest_rate,
    start_date,
    maturity_date,
    interest_amount,
    status
)
SELECT
    old_reference_number AS reference_number,
    account_no,
    deposit_amount,
    interest_rate,
    start_date,
    maturity_date,
    interest_amount,
    'PENDING' AS status
FROM deposit.obs_time_deposit_data
WHERE old_reference_number IS NOT NULL
LIMIT 1;
```
