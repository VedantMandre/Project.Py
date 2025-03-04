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
