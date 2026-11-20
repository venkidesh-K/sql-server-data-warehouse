# Run guide

This guide covers the files included in this repository and the setup they still require. The scripts have not been executed against SQL Server as part of preparing this documentation.

## Before you start

Use a disposable development database and a SQL Server client that understands `GO` batches, such as SSMS. Select the intended database before executing any SQL.

The repository does not include database initialization, schema creation, Bronze table DDL, Silver table DDL or Silver transformation procedures. Prepare those objects before running the supplied scripts. Do not use a production database or invent table types from CSV headers without reviewing the data.

## 1. Prepare the required relations

Create the `bronze`, `silver` and `gold` schemas in the development database. The Bronze loader expects:

```text
bronze.crm_cust_info       bronze.erp_cust_az12
bronze.crm_prd_info        bronze.erp_loc_a101
bronze.crm_sales_details   bronze.erp_px_cat_g1v2
```

The Gold views expect:

```text
silver.crm_cust_info       silver.erp_cust_az12
silver.crm_prd_info        silver.erp_loc_a101
silver.crm_sales_details   silver.erp_px_cat_g1v2
```

Review [ddl_gold.sql](../scripts/gold/ddl_gold.sql) for the required Silver columns and join keys. For example, Silver product data must expose `cat_id`, while the raw CRM product CSV does not have a separate column by that name.

## 2. Make the source files accessible

Open [proc_load_bronze.sql](../scripts/bronze/proc_load_bronze.sql) in a working copy. Replace all six hard-coded Windows `FROM` paths with absolute paths to the correct files on your SQL Server host. The procedure takes no path parameter.

| Loader target | Repository file |
| :--- | :--- |
| `bronze.crm_cust_info` | `datasets/source_crm/cust_info.csv` |
| `bronze.crm_prd_info` | `datasets/source_crm/prd_info.csv` |
| `bronze.crm_sales_details` | `datasets/source_crm/sales_details.csv` |
| `bronze.erp_cust_az12` | `datasets/source_erp/cust_az12.csv` |
| `bronze.erp_loc_a101` | `datasets/source_erp/loc_a101.csv` |
| `bronze.erp_px_cat_g1v2` | `datasets/source_erp/px_cat_g1v2.csv` |

The loader's original ERP filenames are uppercase. This repository uses lowercase names. Match the actual filename and casing, especially on case-sensitive filesystems.

`BULK INSERT` reads from the SQL Server machine, not automatically from the computer running SSMS. Grant the SQL Server service account read access to the files and review the database permissions needed for bulk loading.

## 3. Separate the Bronze definition and invocation

> The loader truncates each target table before loading it. It does not wrap the entire refresh in a transaction. A later failure can leave a partial refresh.

The supplied file ends with an `EXEC bronze.load_bronze;` invocation immediately after the procedure definition, without a `GO` separator. Do not run the whole file unchanged as a single batch, because that invocation can be included in the procedure body.

In a working copy, add a batch separator after the procedure definition's final `END`, before the trailing invocation. Alternatively, select and execute only the `CREATE OR ALTER PROCEDURE` definition first, excluding the final invocation, then run the invocation separately:

```sql
-- Run separately, after creating the procedure:
EXEC bronze.load_bronze;
GO
```

The repository preserves the supplied SQL logic rather than silently changing it.

## 4. Check the Bronze load

The procedure prints load durations and uses `TRY/CATCH` to print errors. It does not rethrow caught errors, so a client reporting that the batch finished is not proof that all files loaded.

Read all messages and compare each Bronze table's row count with the [raw source inventory](data-catalog.md#source-inventory). Investigate mismatches before moving on.

## 5. Prepare Silver

Supply and run the missing Silver DDL and transformations. The Gold script assumes those relations already contain prepared data with matching keys and suitable column types. The raw CRM and ERP key formats are not identical.

No Silver procedure, cleansing test suite or transformation result is supplied in this snapshot. Do not skip Silver by simply renaming raw Bronze tables.

## 6. Create Gold

Once Silver is ready, execute [ddl_gold.sql](../scripts/gold/ddl_gold.sql) in the intended database. It drops and recreates the three views using separate `GO` batches. Review dependencies before replacing any existing views.

The outputs are:

```text
gold.dim_customers
gold.dim_products
gold.fact_sales
```

## 7. Inspect the results

These are starter checks to run in your prepared environment, not recorded results:

```sql
SELECT TOP (10) * FROM gold.dim_customers;
SELECT TOP (10) * FROM gold.dim_products;
SELECT TOP (10) * FROM gold.fact_sales;

-- Sales rows that did not match a dimension.
SELECT COUNT(*) AS unmatched_sales_rows
FROM gold.fact_sales
WHERE customer_key IS NULL OR product_key IS NULL;

-- Duplicate customer surrogate keys.
SELECT customer_key, COUNT(*) AS occurrences
FROM gold.dim_customers
GROUP BY customer_key
HAVING COUNT(*) > 1;

-- Duplicate product surrogate keys.
SELECT product_key, COUNT(*) AS occurrences
FROM gold.dim_products
GROUP BY product_key
HAVING COUNT(*) > 1;
```

Also review natural-key uniqueness, null identifiers, whitespace, unexpected categorical values and date ordering. No automated test suite is included here.

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| Object or schema not found | Create the missing setup objects; check the active database and the required names above |
| CSV file cannot be opened | Check the absolute server-side path, filename casing and SQL Server service-account permissions |
| Procedure behaves unexpectedly when called | Ensure the final invocation was separated from the definition with `GO` |
| Gold views cannot be created | Confirm that Silver relations expose every column referenced by the Gold SQL |
| Sales rows have null dimension keys | Review CRM/ERP key cleanup, product filtering and source-key matches in Silver |
| Some tables loaded but others did not | Inspect all printed errors; the full refresh is not protected by one transaction |

[Back to README](../README.md)
