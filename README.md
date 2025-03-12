```
CREATE TABLE branch_information (
    branch_id INTEGER,
    branch_code VARCHAR,
    description VARCHAR
);
```
```
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (7832, 'BRZ', 'Banco Sumitomo Mitsui Brasileiro S.A.');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (7860, 'CDA', 'Canada Branch');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8801, 'USA', 'NEW YORK BRANCH');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8802, 'LDN', 'SMBC Bank International plc');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8804, 'DUS', 'SMBC DUSSELDORF');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8807, 'BRU', 'BRUSSELS BRANCH');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8100, 'FFT', 'SMBC Bank EU - Frankfurt');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8792, 'LDN', 'SMBC LONDON BRANCH');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8815, NULL, 'CAYMAN');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8816, NULL, 'SMBC BANK EU - Milan');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8872, 'PAR', 'Paris Branch');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8874, 'UAE', 'SMBC DIFC BRANCH - DUBAI');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8881, NULL, 'SMFD IRELAND');
INSERT INTO branch_information (branch_id, branch_code, description) VALUES (8902, NULL, 'THE SUMITOMO MITSUI BANKING CORP.');
```

```
CREATE OR REPLACE PROCEDURE usp_delta_load_rollover()
LANGUAGE plpgsql
AS
$$
BEGIN
    -- Insert with NOT EXISTS for better reliability and performance
    INSERT INTO deposit.new_td_rollover
    (trade_number, investment_type, start_date, maturity_date, tenor, currency, interest_rate,
     maturity_status, reference_number, old_reference_number, time_deposit_amount,
     time_deposit_account_number, settlement_account_number, frequency, interest_accrued_till_date,
     interest_at_maturity, branch, trade_type, funding_source,
     obs_code, sun_id, client_name, account_official_name, done_time, update_time)
    
    SELECT 
        s.trade_number,
        s.investment_type,
        s.start_date::date AS start_date,
        s.maturity_date::date AS maturity_date,
        s.tenor::integer AS tenor,
        s.currency,
        s.interest_rate::decimal(20,2) AS interest_rate,
        s.maturity_status::varchar AS maturity_status,
        s.reference_number::integer AS reference_number,
        s.old_reference_number,
        s.time_deposit_amount::integer AS time_deposit_amount, -- Fixed typo in alias
        s.time_deposit_account_number,
        s.settlement_account_number,
        s.frequency,
        s.interest_accrued_till_date::numeric AS interest_accrued_till_date,
        s.interest_at_maturity::integer AS interest_at_maturity,
        s.branch,
        s.trade_type,
        s.funding_source,
        s.obs_code,
        s.sun_id,
        s.client_name,
        s.account_official_name,
        s.done_time::timestamp AS done_time,
        s.update_time::timestamp AS update_time
    FROM 
        deposit.stg_td_rollover s
    WHERE NOT EXISTS (
        SELECT 1 
        FROM deposit.new_td_rollover n 
        WHERE n.trade_number = s.trade_number
    );
    
    RAISE NOTICE 'INCREMENTAL LOAD SUCCESSFULLY COMPLETED';
END;
$$;
```
```
INSERT INTO deposit.new_td_rollover
    (trade_number, investment_type, start_date, maturity_date, tenor, currency, interest_rate,
     maturity_status, reference_number, old_reference_number, time_deposit_amount,
     time_deposit_account_number, settlement_account_number, frequency, interest_accrued_till_date,
     interest_at_maturity, branch, trade_type, funding_source,
     obs_code, sun_id, client_name, account_official_name, done_time, update_time)
SELECT 
    trade_number,
    investment_type,
    start_date::date,
    maturity_date::date,
    NULLIF(tenor, '')::integer,
    currency,
    interest_rate::decimal(20,2),
    maturity_status::varchar,
    NULLIF(REGEXP_REPLACE(reference_number, '[^0-9]', '', 'g'), '')::integer,
    old_reference_number,
    NULLIF(time_deposit_amount, '')::integer,
    time_deposit_account_number,
    settlement_account_number,
    frequency,
    interest_accrued_till_date::numeric,
    NULLIF(interest_at_maturity, '')::integer,
    branch,
    trade_type,
    funding_source,
    obs_code,
    sun_id,
    client_name,
    account_official_name,
    done_time::timestamp,
    update_time::timestamp
FROM 
    deposit.stg_td_rollover
ON CONFLICT (reference_number)
DO UPDATE SET
    trade_number = EXCLUDED.trade_number,
    investment_type = EXCLUDED.investment_type,
    start_date = EXCLUDED.start_date,
    maturity_date = EXCLUDED.maturity_date,
    tenor = EXCLUDED.tenor,
    currency = EXCLUDED.currency,
    interest_rate = EXCLUDED.interest_rate,
    maturity_status = EXCLUDED.maturity_status,
    old_reference_number = EXCLUDED.old_reference_number,
    time_deposit_amount = EXCLUDED.time_deposit_amount,
    time_deposit_account_number = EXCLUDED.time_deposit_account_number,
    settlement_account_number = EXCLUDED.settlement_account_number,
    frequency = EXCLUDED.frequency,
    interest_accrued_till_date = EXCLUDED.interest_accrued_till_date,
    interest_at_maturity = EXCLUDED.interest_at_maturity,
    branch = EXCLUDED.branch,
    trade_type = EXCLUDED.trade_type,
    funding_source = EXCLUDED.funding_source,
    obs_code = EXCLUDED.obs_code,
    sun_id = EXCLUDED.sun_id,
    client_name = EXCLUDED.client_name,
    account_official_name = EXCLUDED.account_official_name,
    done_time = EXCLUDED.done_time,
    update_time = EXCLUDED.update_time;
```
