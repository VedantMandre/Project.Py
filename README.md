# Proof of Concept: On-Premises Oracle to Azure Blob Storage via ADF

## Objective

This Proof of Concept (PoC) aims to establish a structured data storage system in Azure Blob Storage and an automated pipeline using Azure Data Factory (ADF) to extract data from an on-premises Oracle database, process it, and prepare it for ingestion into an Azure PostgreSQL database. The solution ensures efficient data handling, incremental loading, and scalability within the constraints of available resources.

## Scope

- **Storage Account Access**: Utilize Azure Blob Storage for data storage.
- **Azure Data Factory (ADF) Access**: Leverage ADF for data extraction, transformation, and movement.
- **No Additional Services**: Limit the solution to ADF and Blob Storage, avoiding Data Lakes or other Azure services.
- **Oracle Connectivity**: Assume the cloud team will configure the Self-Hosted Integration Runtime (SHIR) for on-premises Oracle access.

## Objective 1: Structuring Data Storage & Access Setup

### 1. Storage Account Structure

A well-defined storage structure ensures organized data management and easy access for multiple purposes (e.g., sales, FX rates). The proposed structure is as follows:

#### Containers & Purpose
- **`landing/`**: Stores raw data extracted from Oracle.
- **`processed/`**: Stores transformed data ready for PostgreSQL ingestion.
- **`control/`**: Stores metadata files for tracking incremental loads (e.g., timestamps).

#### Folder Structure
```
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

#### File Naming Convention
- **Data Files**: `[table_name]_[YYYYMMDDHHMM].csv` (e.g., `sales_data_202502201230.csv`).
- **Watermark File**: `sales_data_watermark.json` (e.g., a JSON file with table name and last load timestamp).

This structure aligns with the existing CMS application architecture and supports scalability for additional data types or sources.

### 2. Access Setup

- **Azure Role-Based Access Control (RBAC)**:
  - Database Team: Full access to `landing`, `processed`, and `control` containers.
  - ADF Service: Write access to `landing` and `control` containers; read access to `control` for watermark tracking.
- **Collaboration**: Coordinate with the cloud team to configure RBAC and ensure ADF connectivity to Blob Storage via access keys or SAS tokens.
- **Tools**: Teams can access data via Azure Storage Explorer or integrate with Power BI for validation.

## Objective 2: Creating an ADF Pipeline to Process Data

### Pipeline Overview

The ADF pipeline will:
1. Extract selective columns from an on-premises Oracle table.
2. Save raw data as CSV files in the `landing/` container.
3. Apply a simple transformation (e.g., filter based on business rules).
4. Store transformed data in the `processed/` container.
5. Prepare data for PostgreSQL ingestion (to be handled separately).
6. Update the watermark file in the `control/` container for incremental loading.

### Steps to Build the ADF Pipeline

1. **Create Linked Services in ADF**:
   - **OracleLinkedService**: Connects to the on-premises Oracle database via SHIR (managed by the cloud team).
   - **BlobStorageLinkedService**: Connects to Azure Blob Storage using access keys or SAS tokens.
   - **PostgreSQLLinkedService**: Placeholder for future PostgreSQL integration (to be configured later).

2. **Create Datasets**:
   - **OracleSourceDataset**: Represents the Oracle table with selective columns and incremental load logic.
   - **BlobSinkDataset**: Defines the CSV output in the `landing/` container.
   - **ProcessedBlobDataset**: Defines the transformed CSV output in the `processed/` container.
   - **ControlDataset**: Manages the watermark metadata in the `control/` container.

3. **Define Pipeline Logic**:
   - **LookupWatermark**: Reads the last processed timestamp from the watermark file.
   - **CopyOracleToBlob**: Extracts new or updated data from Oracle and writes it to the `landing/` container.
   - **TransformData**: Applies a basic transformation (e.g., filtering records based on a threshold).
   - **CopyToProcessed**: Saves transformed data to the `processed/` container.
   - **UpdateWatermark**: Updates the watermark file with the latest processed timestamp.

### Incremental Loading
- Use a watermark-based approach (e.g., a timestamp column) to identify new or updated records.
- Store the last processed timestamp in the `control/` container for each table.

## Testing & Validation

### 1. Test Data Generation
- Insert sample data into the Oracle table to simulate real-world scenarios (to be coordinated with the Oracle DBA team).

### 2. Pipeline Execution & Verification
- Run the ADF pipeline manually.
- Validate raw data in the `landing/` container.
- Confirm transformed data in the `processed/` container meets business rules.
- Check the watermark file in the `control/` container for an updated timestamp.

## Acceptance Criteria

- ✅ **Storage Structure Created**: `landing/`, `processed/`, and `control/` containers exist in Azure Blob Storage with the defined folder structure.
- ✅ **Access Granted**: ADF and the database team have appropriate permissions to read/write to Blob Storage.
- ✅ **Pipeline Success**: ADF pipeline extracts, processes, and stores data as expected.
- ✅ **Incremental Loading**: Pipeline processes only new or updated records based on the watermark.
- ✅ **Documentation**: Comprehensive setup details provided (this README).

## Next Steps

- **Cloud Team Collaboration**: Finalize SHIR setup for Oracle connectivity.
- **PostgreSQL Integration**: Extend the pipeline to load data from `processed/` into PostgreSQL (requires PostgreSQL access).
- **Monitoring**: Add error handling and ADF monitoring (e.g., alerts for pipeline failures).
- **Scaling**: Test with additional tables or data types (e.g., FX rates).

## Documentation

This README serves as the primary documentation, covering:
- Storage setup and naming conventions.
- Access control requirements.
- ADF pipeline design and objectives.
- Testing and validation steps.

For detailed ADF pipeline configuration, refer to the ADF portal or coordinate with the cloud team for SHIR-specific settings.
