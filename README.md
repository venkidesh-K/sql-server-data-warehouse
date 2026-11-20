# SQL Server Data Warehouse Development

A complete data warehouse built in Microsoft SQL Server using Medallion Architecture (Bronze, Silver, Gold) to integrate ERP and CRM data sources. The Gold layer is modelled as a Star Schema of fact and dimension tables, ready for analytics.

## Tools & Tech
- Microsoft SQL Server
- T-SQL (stored procedures, JOINs, CTEs, subqueries, CASE, window functions)

## Key Features
- Medallion Architecture: raw ingestion in Bronze, cleansed and standardized data in Silver, analytics-ready Star Schema in Gold
- ETL pipelines and stored procedures that ingest, cleanse, and transform ERP and CRM source data
- Star Schema Gold layer: fact tables with conformed dimensions for customer, product, and sales analysis
- Query tuning applied across analytical workloads
- Data validation, deduplication, and consistency checks at every layer
- Full documentation of database structures and transformation logic

## How to Run
1. Run the scripts in order: `/scripts/bronze` then `/scripts/silver` then `/scripts/gold`
2. Execute the stored procedures in `/scripts/etl` to load each layer
3. Run the validation queries in `/scripts/validation` to confirm row counts and data quality across layers
4. Sample analytical queries for customer, product, and sales analysis are in `/scripts/analysis`

## Screenshots
> **[Placeholder: schema diagram of the Gold Star Schema, and/or a screenshot of a sample analytical query result]**

## Credits
This was a team project from our training, built together with [Thouseef](https://github.com/thousifva). This repo contains my own write-up of it.
