# SQL Server Data Warehouse

A small project that takes sales data from two systems (CRM and ERP) and organizes it step by step, so it is ready for reports.

## How it works

![Bronze, Silver and Gold layers](docs/images/medallion-architecture.svg)

- **Bronze**: the raw data, loaded as it is.
- **Silver**: the data after cleaning.
- **Gold**: the final tables for reports (customers, products, sales).

## What is inside

- `datasets/`: the CSV files (CRM and ERP)
- `scripts/bronze/`: loads the raw data
- `scripts/gold/`: creates the final views
- `docs/`: diagrams and notes

The database setup scripts and the Silver scripts are not included here.

## Tools

SQL Server and T-SQL.

---

Based on the SQL Data Warehouse project by [Data with Baraa](https://github.com/DataWithBaraa/sql-data-warehouse-project). MIT License, see [LICENSE](LICENSE).
