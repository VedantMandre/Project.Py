```
SELECT 
    md5(random()::TEXT || clock_timestamp()::TEXT) AS id,  -- Generate Unique ID
    md5(contract_number::TEXT || modify_version::TEXT || leg_no::TEXT || random()::TEXT) AS distribution_key, 
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
ALTER TABLE deposit.test2_td_rollover 
ADD COLUMN IF NOT EXISTS id TEXT;
ALTER TABLE deposit.test2_td_rollover 
ADD COLUMN IF NOT EXISTS distribution_key TEXT;
```
