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
