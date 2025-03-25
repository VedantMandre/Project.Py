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
BEGIN
    -- Update status to 'Finalized' in the rollover table where reference_number matches old_reference_number
    UPDATE deposit.test_recon_time_deposit_rollover tdr
    SET 
        status = 'Finalized'
    FROM deposit.test_recon_obs_time_deposit_data otd
    WHERE TRIM(otd.old_reference_number) = TRIM(tdr.reference_number)
    AND otd.old_reference_number IS NOT NULL
    AND tdr.reference_number IS NOT NULL;

    -- Commit the changes
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        -- Roll back on error
        ROLLBACK;
        RAISE EXCEPTION 'Error occurred: %', SQLERRM;
END;
$$;

```
