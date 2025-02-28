### This document outlines the process for transforming and loading data from Azure Blob Storage to Azure Cosmos DB for PostgreSQL. The workflow involves data transformation using Azure Data Factory (ADF) and loading the transformed data into a PostgreSQL table for further processing.

### Blob Storage Structure
The input file is stored in Azure Blob Storage in the following format:

File Name: timedeposit-->2025-02-21_timedeposit.csv
Storage Location: Azure Blob Storage (Container: incoming-data/Timedeposit)
### Transformation Process
### Source File:

The raw data is in the CSV format located in Azure Blob Storage.
### Data Flow:

A Derived Row transformation is applied to the data to clean and process it as needed.
The transformed data is saved in Parquet format for efficient storage and further processing.
Output File: cleaned_td_rollover.parquet
### Pipeline:

A pipeline named Transform_pipeline was created to automate the transformation process.
The pipeline reads the input CSV, applies the transformations, and stores the cleaned data in Parquet format in Blob Storage.
Loading Data to Cosmos DB for PostgreSQL
### Destination Table:
A PostgreSQL table named deposit.test_td_rollover was created to load the transformed data.
### Pipeline:
Another pipeline was built to load the data from the cleaned_td_rollover.parquet file (stored in Blob Storage) to the PostgreSQL table.
This pipeline uses the Azure Cosmos DB for PostgreSQL as the destination.
Successful Execution:
The data from the Parquet file was successfully loaded into the deposit.test_td_rollover table in the PostgreSQL database.
### Outcome
The data transformation and loading process is fully automated with two pipelines:
Transform_pipeline: Transforms and stores data as a Parquet file.
Load_to_DB_pipeline: Loads the transformed data into the PostgreSQL database.

```
UPDATE your_table
SET total_exact_days = 
    (EXTRACT(YEAR FROM AGE(due_date::DATE, as_of_date)) * 365) + 
    (EXTRACT(MONTH FROM AGE(due_date::DATE, as_of_date)) * 30) + 
    (EXTRACT(DAY FROM AGE(due_date::DATE, as_of_date)))
WHERE as_of_date IS NOT NULL AND due_date IS NOT NULL;
```
