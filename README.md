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
    ) AND (tdr.status IS NULL OR tdr.status <> 'Finalized');
END;
$$;



SELECT reference_number, old_reference_number
FROM deposit.test_recon_time_deposit_rollover tdr
JOIN deposit.test_recon_obs_time_deposit_data obs
ON tdr.reference_number = obs.old_reference_number;

```
