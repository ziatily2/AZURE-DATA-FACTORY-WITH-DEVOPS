# Azure Data Factory End-to-End Project

Complete data engineering pipeline demonstrating data ingestion, transformation, and deployment using Azure Data Factory with CI/CD integration.

## Project Overview

This project implements a data pipeline that:
- Ingests data from on-premises SQL Server and APIs
- Transforms data using mapping data flows
- Stores processed data in Azure Data Lake Storage Gen2
- Follows medallion architecture (Bronze → Silver → Gold)
- Deploys using Azure DevOps with Git integration

## Architecture

```
On-Premises SQL Server → Self-Hosted Integration Runtime → ADLS Gen2 (Bronze - Raw Data)
                                                                 ↓
                                                        Data Transformations
                                                                 ↓
                                                      ADLS Gen2 (Silver - Cleaned Data)
                                                                 ↓
                                                          Aggregations
                                                                 ↓
                                                      ADLS Gen2 (Gold - Business Data)
```

## Prerequisites

- Azure Subscription
- Azure Data Factory instance
- Azure Data Lake Storage Gen2 account
- Self-Hosted Integration Runtime (for on-premises connectivity)
- Azure DevOps account
- SQL Server database

## Data Sources

The project uses five data sources representing an airline booking system:

| Source | Format | Description |
|--------|--------|-------------|
| DimAirline | CSV | Airline master data (ID, name, country) |
| DimAirport | JSON | Airport master data (ID, name, city) |
| DimFlight | CSV | Flight information |
| DimPassenger | CSV | Passenger details (ID, name, gender, age, country) |
| FactBookings | SQL | Booking transactions |

## Project Structure

**Pipelines:**
- `nprem_ingestion` - Ingests data from on-premises sources
- `API_Ingestion` - Ingests data from REST APIs
- `Parent Pipeline` - Orchestrates multiple pipelines

**Data Flows:**
- `SilverDataFlow` - Transforms raw data into cleaned format
- `GoldDataFlow` - Creates aggregated business views

**Storage Containers:**
- `silver` - Stores dimension tables (DimAirline, DimAirport, DimFlight, DimPassenger, FactBookings)
- `gold` - Stores analytics tables (businessView)

## Data Flow Details

### SilverDataFlow (Bronze to Silver)

Transforms raw data into standardized format:

1. **derivedColumnC** - Creates airline dimension columns (airline_id, airline_name, country)
2. **selectCols** - Selects and renames flight columns to flight_id
3. **derivedGenderFlag** - Generates gender flag from passenger data
4. **derivedGenderFe** - Creates full_name and gender_flag columns
5. **filterGreater25** - Filters passengers where age > 25 and country is not null
6. **derivedName** - Derives passenger full names

**Output:** Cleaned dimension and fact tables in Silver container

### GoldDataFlow (Silver to Gold)

Creates business-ready analytics:

1. **AirlineJoin** - Left outer join between FactBookings and DimAirline on booking_id
2. **aggregateAirline** - Groups by airline_name and aggregates total bookings
3. **Ranking** - Ranks airlines based on booking volume
4. **filterTop5** - Filters top 5 performing airlines

**Output:** Aggregated business views in Gold container

## Setup Instructions

### 1. Azure Resources

Create Storage Account:
```
- Enable hierarchical namespace (Data Lake Gen2)
- Create containers: silver, gold
```

Create Data Factory:
```
- Enable Git integration
- Connect to Azure DevOps repository
```

### 2. Self-Hosted Integration Runtime

Install on your on-premises machine:
```
1. Download Self-Hosted IR from Azure portal
2. Install and register with authentication key
3. Verify connectivity in ADF portal
```

### 3. File Share Configuration

For on-premises data access:
```powershell
New-SmbShare -Name "IlyasZiatFiles" -Path "C:\IlyasZiatFiles" -FullAccess "Everyone"
Grant-SmbShareAccess -Name "IlyasZiatFiles" -AccountName "NT SERVICE\DIAHostService" -AccessRight Full
```

### 4. Linked Services

Configure connections:
- `SQLToDataLake` - On-premises SQL Server connection
- `ds_silver_src` - ADLS Gen2 silver container
- `ds_sqlsource` - SQL Server source

Update with your connection strings and credentials.

### 5. Upload Sample Data

Upload these files to your source location:
- DimAirline.csv
- DimAirport.json
- DimFlight.csv
- DimPassenger.csv
- fact_bookings_full.sql

### 6. Publish Pipelines

In ADF Studio:
```
1. Validate all pipelines
2. Enable Data flow debug (optional)
3. Click "Publish all"
4. Confirm changes
```

## Azure DevOps Integration

**Repository Configuration:**
- Organization: safouanziat
- Project: ADF_Project
- Repository: ilyasziatadtproject
- Collaboration Branch: main
- Publish Branch: adf_publish

**Deployment Process:**
1. Develop in ADF Studio (saves to main branch)
2. Create pull request for review
3. Merge approved changes
4. Click "Publish" to generate ARM templates
5. ARM templates saved to adf_publish branch
6. Deploy templates to target environments

## Pipeline Execution

Monitor pipeline runs in the Monitor tab:

| Pipeline | Status | Duration | Runtime |
|----------|--------|----------|---------|
| SilverDataFlow | Succeeded | 5m 49s | AutoResolveIntegrationRuntime (East US) |
| GoldDataFlow | Succeeded | 1m 24s | AutoResolveIntegrationRuntime (East US) |

## Testing

Enable Data flow debug mode for:
- Interactive data preview
- Transformation testing
- Expression validation
- Schema inspection

Debug sessions remain active for 60 minutes by default.

## Monitoring

View detailed metrics:
- Pipeline run history
- Activity-level logs
- Data flow execution times
- Integration runtime status
- Error messages and troubleshooting

Access via Monitor tab or Azure Monitor integration.

## Key Features

- No-code/low-code data transformations using mapping data flows
- Medallion architecture for data quality layers
- Parameterized pipelines for reusability
- Self-hosted IR for hybrid connectivity
- CI/CD with Azure DevOps Git integration
- ARM template-based deployments
- Managed Spark execution for data flows

## Troubleshooting

**Integration Runtime Issues:**
- Verify Self-Hosted IR service is running
- Check network connectivity and firewall rules

**Pipeline Failures:**
- Check activity run details in Monitor tab
- Verify linked service connections
- Review data flow transformation logic

**Authentication Errors:**
- Confirm Managed Identity permissions on storage
- Verify service principal credentials

## Notes

- Data flows execute on Azure-managed Spark clusters
- Storage account uses hierarchical namespace for Data Lake Gen2
- Authentication uses Managed Identity where possible
- All transformations are serverless and auto-scaling
