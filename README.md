```
UPDATE deposit.test_td_rollover 
SET id = encode(digest(
        contract_number::TEXT || COALESCE(modify_version::TEXT, '') || 
        COALESCE(leg_no::TEXT, '') || COALESCE(as_of_date::TEXT, '') || 
        COALESCE(obs_number::TEXT, '') || COALESCE(data_date::TEXT, '') || 
        COALESCE(contract_date::TEXT, '') || COALESCE(bank_ref::TEXT, '') || 
        COALESCE(currency_code::TEXT, '') || clock_timestamp()::TEXT || random()::TEXT, 
        'sha256'), 'hex');
```
```
SELECT 
    encode(digest(random()::TEXT || clock_timestamp()::TEXT, 'sha256'), 'hex') AS id, -- FIPS-compliant unique key
    encode(digest(contract_number::TEXT || modify_version::TEXT || leg_no::TEXT || random()::TEXT, 'sha256'), 'hex') AS distribution_key, 
    contract_number,
    modify_version,
    leg_no,
    as_of_date,
    obs_number,
    data_date,
    contract_date,
    bank_ref,
    currency_code
FROM deposit.test2_td_rollover;
```
```
SELECT 
    -- Generate a unique id by combining random, timestamp, and row-specific columns
    encode(digest(
        contract_number::TEXT || 
        COALESCE(modify_version::TEXT, '') || 
        COALESCE(leg_no::TEXT, '') || 
        random()::TEXT || 
        clock_timestamp()::TEXT, 
        'sha256'), 'hex') AS id,  -- FIPS-compliant unique key
    
    -- Distribution key based on the combination of relevant fields
    encode(digest(
        contract_number::TEXT || 
        COALESCE(modify_version::TEXT, '') || 
        COALESCE(leg_no::TEXT, '') || 
        random()::TEXT, 
        'sha256'), 'hex') AS distribution_key, 
    
    contract_number,
    modify_version,
    leg_no,
    as_of_date,
    obs_number,
    data_date,
    contract_date,
    bank_ref,
    currency_code
FROM deposit.test2_td_rollover;
```

```
-- DELETE records from destination if they don't exist in source
DELETE FROM deposit.test_td_rollover d
USING deposit.test2_td_rollover s
WHERE d.id = s.id AND s.id IS NULL;

-- INSERT new records from staging into destination
INSERT INTO deposit.test_td_rollover
SELECT * FROM deposit.test2_td_rollover s
WHERE NOT EXISTS (SELECT 1 FROM deposit.test_td_rollover d WHERE d.id = s.id);

-- INSERT updated records (treated as new versions)
INSERT INTO deposit.test_td_rollover
SELECT s.*
FROM deposit.test2_td_rollover s
JOIN deposit.test_td_rollover d 
ON s.contract_number = d.contract_number
AND s.modify_version = d.modify_version
AND s.leg_no = d.leg_no
WHERE s.id <> d.id;  -- If hash changed, it's an update (new version)

```
```
-- Create a function to generate a unique, FIPS-compliant ID
CREATE OR REPLACE FUNCTION generate_unique_id(
    p_contract_number TEXT, 
    p_modify_version TEXT DEFAULT NULL, 
    p_leg_no TEXT DEFAULT NULL
) RETURNS TEXT AS $$
DECLARE
    v_unique_id TEXT;
BEGIN
    v_unique_id := encode(
        digest(
            p_contract_number::TEXT || 
            COALESCE(p_modify_version, '') || 
            COALESCE(p_leg_no, '') || 
            random()::TEXT || 
            clock_timestamp()::TEXT, 
            'sha256'
        ), 
        'hex'
    );
    
    RETURN v_unique_id;
END;
$$ LANGUAGE plpgsql;

-- Alter the table to add the id column if it doesn't exist
ALTER TABLE deposit.test_td_rollover 
ADD COLUMN IF NOT EXISTS id VARCHAR(64);

-- Update the existing rows with unique IDs
UPDATE deposit.test_td_rollover
SET id = generate_unique_id(
    contract_number, 
    modify_version, 
    leg_no
);

-- Optional: Create an index on the new id column for performance
CREATE UNIQUE INDEX IF NOT EXISTS idx_test_td_rollover_id 
ON deposit.test_td_rollover (id);

-- Query to generate and select unique IDs
SELECT 
    generate_unique_id(
        contract_number, 
        modify_version, 
        leg_no
    ) AS id,
    encode(digest(
        contract_number::TEXT || 
        COALESCE(modify_version::TEXT, '') || 
        COALESCE(leg_no::TEXT, '') || 
        random()::TEXT, 
        'sha256'), 'hex') AS distribution_key,
    contract_number,
    modify_version,
    leg_no,
    as_of_date,
    obs_number,
    data_date,
    contract_date,
    bank_ref,
    currency_code
FROM deposit.test_td_rollover;
```

```
-- 1. Unique ID Generation Approach
SELECT 
    -- Generate a deterministic unique ID for each row
    MD5(
        COALESCE(contract_number::TEXT, '') || '|' ||
        COALESCE(modify_version::TEXT, '') || '|' ||
        COALESCE(leg_no::TEXT, '') || '|' ||
        COALESCE(as_of_date::TEXT, '') || '|' ||
        COALESCE(obs_number::TEXT, '')
    ) AS unique_record_id,
    contract_number,
    modify_version,
    leg_no,
    as_of_date,
    obs_number
FROM deposit.test_td_rollover;

-- 2. Pre-Copy Script for Upsert Logic (Pseudo-SQL)
MERGE INTO destination_table AS target
USING source_table AS source
ON (target.unique_record_id = source.unique_record_id)
WHEN MATCHED AND (
    -- Check if any significant columns have changed
    target.contract_number != source.contract_number OR
    target.modify_version != source.modify_version OR
    target.leg_no != source.leg_no OR
    target.as_of_date != source.as_of_date
) THEN
    -- Update existing row if changes detected
    UPDATE SET 
        contract_number = source.contract_number,
        modify_version = source.modify_version,
        leg_no = source.leg_no,
        as_of_date = source.as_of_date
WHEN NOT MATCHED THEN
    -- Insert new rows
    INSERT (unique_record_id, contract_number, modify_version, leg_no, as_of_date)
    VALUES (
        source.unique_record_id, 
        source.contract_number, 
        source.modify_version, 
        source.leg_no, 
        source.as_of_date
    )
WHEN NOT MATCHED BY SOURCE THEN
    -- Delete rows not present in source
    DELETE;

-- 3. Deletion Detection Query
WITH source_ids AS (
    SELECT 
        MD5(
            COALESCE(contract_number::TEXT, '') || '|' ||
            COALESCE(modify_version::TEXT, '') || '|' ||
            COALESCE(leg_no::TEXT, '') || '|' ||
            COALESCE(as_of_date::TEXT, '') || '|' ||
            COALESCE(obs_number::TEXT, '')
        ) AS unique_record_id
    FROM deposit.test_td_rollover
),
destination_ids AS (
    SELECT unique_record_id
    FROM destination_table
)
SELECT 
    d.unique_record_id AS deleted_record_id
FROM destination_ids d
LEFT JOIN source_ids s ON d.unique_record_id = s.unique_record_id
WHERE s.unique_record_id IS NULL;
```
