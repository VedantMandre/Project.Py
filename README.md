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
    -- Insert new records from staging table to target table
    INSERT INTO deposit.new_td_rollover
    (trade_number, investment_type, start_date, maturity_date, tenor, currency, interest_rate,
     maturity_status, reference_number, old_reference_number, time_deposit_amount,
     time_deposit_account_number, settlement_account_number, frequency, interest_accrued_till_date,
     interest_at_maturity, branch, trade_type, funding_source,
     obs_code, sun_id, client_name, account_official_name, done_time, update_time)
    
    SELECT 
        src.trade_number,
        src.investment_type,
        CAST(src.start_date AS date) AS start_date,
        CAST(src.maturity_date AS date) AS maturity_date,
        CAST(src.tenor AS integer) AS tenor,
        src.currency,
        CAST(src.interest_rate AS decimal(20,2)) AS interest_rate,
        CAST(src.maturity_status AS varchar) AS maturity_status,
        CAST(src.reference_number AS integer) AS reference_number,
        src.old_reference_number,
        CAST(src.time_deposit_amount AS integer) AS time_deposit_number,
        src.time_deposit_account_number,
        src.settlement_account_number,
        src.frequency,
        CAST(src.interest_accrued_till_date AS numeric) AS interest_accrued_till_date,
        CAST(src.interest_at_maturity AS integer) AS interest_at_maturity,
        src.branch,
        src.trade_type,
        src.funding_source,
        src.obs_code,
        src.sun_id,
        src.client_name,
        src.account_official_name,
        CAST(src.done_time AS timestamp) AS done_time,
        CAST(src.update_time AS timestamp) AS update_time
    FROM 
        deposit.stg_td_rollover src
    LEFT JOIN 
        deposit.new_td_rollover tgt ON tgt.trade_number = src.trade_number
    WHERE 
        tgt.trade_number IS NULL;
    
    -- Raise notice on successful completion
    RAISE NOTICE 'INCREMENTAL LOAD SUCCESSFULLY COMPLETED';
    
EXCEPTION
    WHEN OTHERS THEN
        -- Log the error and rethrow
        RAISE NOTICE 'Error occurred during delta load: %', SQLERRM;
        RAISE;
END;
$$;
```

```
CREATE OR REPLACE PROCEDURE usp_delta_load_rollover()
LANGUAGE plpgsql
AS
$$
BEGIN
    RAISE NOTICE 'Starting procedure execution';
    -- Add your main logic here
    RAISE NOTICE 'Procedure completed successfully';
EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'Error in procedure: %', SQLERRM;
        RAISE;
END;
$$;
```
