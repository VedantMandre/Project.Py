# Objective

The goal of this Proof of Concept (PoC) is to set up a structured data storage system and an automated pipeline to extract data from an on-premises Oracle database, store it in Azure Blob Storage, process it, and insert it into a PostgreSQL database.

# Scope

Storage Account Access: The solution will be implemented using Azure Blob Storage.

Azure Data Factory (ADF) Access: The pipeline will be built using ADF for data movement and transformation.

No Data Lakes or Additional Services: The solution will work strictly within the limits of available resources.

# Objective 1: Structuring Data Storage & Access Setup
```
1. Storage Account Structure

A well-defined storage structure ensures easy access and organization. The following naming conventions and folder structures will be used:

Containers & Purpose:

landing/ → Stores raw data extracted from Oracle.

processed/ → Stores transformed data ready for PostgreSQL ingestion.

control/ → Stores metadata files, including tracking timestamps for incremental loads.

Folder Structure Example:

landing/
  oracle/
    sales/
      [YYYY]/[MM]/[DD]/
        sales_data_[YYYYMMDDHHMM].csv

processed/
  oracle/
    sales/
      [YYYY]/[MM]/[DD]/
        sales_data_[YYYYMMDDHHMM]_processed.csv

control/
  oracle/
    sales_data_watermark.json
```
# File Naming Convention:

Data files: sales_data_[YYYYMMDDHHMM].csv (e.g., sales_data_202502201230.csv)

Watermark file: sales_data_watermark.json (e.g., {"table_name": "sales_data", "last_load_time": "2025-02-20 12:30:00"})

2. Access Setup

Azure Role-Based Access Control (RBAC) will be used to define who can read, write, and manage files.

ADF Access: ADF will be granted permission to write to the landing and control containers.

Storage Access: Teams can access data via Azure Storage Explorer or Power BI.

Objective 2: Creating an ADF Pipeline to Process Data

# Pipeline Overview

Extract selective columns from Oracle (sale_id, sale_amount, last_updated_date)

Save the data as a CSV file in Blob Storage (landing/oracle/sales/YYYY/MM/DD/)

Perform a simple transformation (e.g., filter out sale_amount < 500)

Store the processed file in the processed/ container

Insert transformed data into PostgreSQL

Update the control file with the latest processed timestamp

# Steps to Build the ADF Pipeline

1. Create Linked Services in ADF

OracleLinkedService: Connects to the on-premises Oracle database using a Self-Hosted Integration Runtime (SHIR).

BlobStorageLinkedService: Connects to the Azure Blob Storage using access keys or SAS tokens.

PostgreSQLLinkedService: Connects to the PostgreSQL database.

2. Create Datasets

OracleSourceDataset (Table: sales_data with query for incremental loading)

BlobSinkDataset (Writes CSV files to Blob Storage)

ProcessedBlobDataset (For transformed files)

ControlDataset (Stores watermark metadata)

3. Define Pipeline Logic

LookupWatermark: Reads control/sales_data_watermark.json to get the last processed timestamp.

CopyOracleToBlob: Extracts data from Oracle based on last_updated_date and writes it to Blob Storage.

TransformData: Filters records where sale_amount > 500 (using ADF Data Flow or Stored Procedure in PostgreSQL).

CopyToPostgreSQL: Loads transformed data into PostgreSQL.

UpdateWatermark: Updates sales_data_watermark.json with the latest timestamp.
```
Example SQL Query for Incremental Loading

SELECT sale_id, sale_amount, last_updated_date
FROM sales_data
WHERE last_updated_date > TO_TIMESTAMP('@{pipeline().parameters.LastLoadTime}', 'YYYY-MM-DD HH24:MI:SS')
```
Testing & Validation

1. Insert Test Data into Oracle
```
INSERT INTO sales_data
SELECT
  LEVEL as sale_id,
  ROUND(DBMS_RANDOM.VALUE(100, 1000), 2) as sale_amount,
  SYSDATE as last_updated_date
FROM DUAL
CONNECT BY LEVEL <= 100;
```
2. Run Pipeline & Verify Output

Check the landing container for sales_data_YYYYMMDDHHMM.csv.

Ensure only filtered data appears in processed container.

Verify PostgreSQL table contains the transformed data.

Confirm sales_data_watermark.json has updated timestamp.

# Acceptance Criteria

✅ Storage Structure Created: landing/, processed/, and control/ containers exist in Azure Blob Storage.✅ Access Granted: ADF and relevant teams have proper permissions.✅ Pipeline Successfully Extracts and Processes Data: Oracle to Blob, simple transformation, and PostgreSQL insertion work as expected.✅ Incremental Loading Works: Only new/updated records are processed.✅ Documentation Completed: Includes storage setup, access controls, pipeline design, and testing steps.
