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
ALTER TABLE deposit.test2_td_rollover 
ADD COLUMN IF NOT EXISTS id TEXT;
ALTER TABLE deposit.test2_td_rollover 
ADD COLUMN IF NOT EXISTS distribution_key TEXT;
```
