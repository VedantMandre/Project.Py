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
MERGE INTO deposit.new_td_rollover tgt
USING deposit.ste_td_rollever src
ON tgt.trade_number = src.trade_number
WHEN MATCHED THEN
    UPDATE SET 
        investment_type = src.investment_type,
        start_date = CAST(src.start_date AS DATE),
        maturity_date = CAST(src.maturity_date AS DATE),
        tenor = CAST(src.tenor AS INTEGER),
        currency = src.currency,
        interest_rate = CAST(src.interest_rate AS DECIMAL(20,2)),
        maturity_status = CAST(src.maturity_status AS VARCHAR),
        reference_number = CAST(src.reference_number AS INTEGER),
        old_reference_number = src.old_reference_number,
        time_deposit_amount = CAST(REPLACE(src.time_deposit_amount, ',', '') AS DECIMAL(20,2)),
        time_deposit_account_number = src.time_deposit_account_number,
        settlement_account_number = src.settlement_account_number,
        frequency = src.frequency,
        interest_accrued_till_date = CAST(src.interest_accrued_till_date AS DECIMAL(20,2)),
        interest_at_maturity = CAST(REPLACE(src.interest_at_maturity, '''', '') AS DECIMAL(20,2)),
        branch = src.branch,
        trade_type = src.trade_type,
        funding_source = src.funding_source,
        obs_code = src.obs_code,
        sun_id = src.sun_id,
        client_name = src.client_name,
        account_official_name = src.account_official_name,
        done_time = CAST(src.done_time AS TIMESTAMP),
        update_time = CAST(src.update_time AS TIMESTAMP)
WHEN NOT MATCHED THEN
    INSERT (
        trade_number, investment_type, start_date, maturity_date, tenor, currency,
        interest_rate, maturity_status, reference_number, old_reference_number,
        time_deposit_amount, time_deposit_account_number, settlement_account_number, frequency,
        interest_accrued_till_date, interest_at_maturity, branch, trade_type, funding_source,
        obs_code, sun_id, client_name, account_official_name, done_time, update_time
    )
    VALUES (
        src.trade_number, src.investment_type,
        CAST(src.start_date AS DATE), CAST(src.maturity_date AS DATE), CAST(src.tenor AS INTEGER), src.currency,
        CAST(src.interest_rate AS DECIMAL(20,2)), CAST(src.maturity_status AS VARCHAR), 
        CAST(src.reference_number AS INTEGER), src.old_reference_number,
        CAST(REPLACE(src.time_deposit_amount, ',', '') AS DECIMAL(20,2)), src.time_deposit_account_number, 
        src.settlement_account_number, src.frequency,
        CAST(src.interest_accrued_till_date AS DECIMAL(20,2)), 
        CAST(REPLACE(src.interest_at_maturity, '''', '') AS DECIMAL(20,2)), 
        src.branch, src.trade_type, src.funding_source, 
        src.obs_code, src.sun_id, src.client_name, src.account_official_name,
        CAST(src.done_time AS TIMESTAMP), CAST(src.update_time AS TIMESTAMP)
    );
```
