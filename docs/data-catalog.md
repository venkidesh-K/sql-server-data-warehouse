# Data catalog

This catalog describes the six supplied raw CSVs and the three views defined in `scripts/gold/ddl_gold.sql`. It does not infer Silver transformations or SQL column types that were not supplied.

## Source inventory

Counts were measured from the supplied CSV records, excluding the header. They are raw input counts, not validated warehouse output counts.

| System | File | Raw records | Main fields |
| :--- | :--- | ---: | :--- |
| CRM | [cust_info.csv](../datasets/source_crm/cust_info.csv) | 18,494 | Customer IDs/keys, names, marital status, gender, creation date |
| CRM | [prd_info.csv](../datasets/source_crm/prd_info.csv) | 397 | Product ID/key, name, cost, line, start/end dates |
| CRM | [sales_details.csv](../datasets/source_crm/sales_details.csv) | 60,398 | Order number, product/customer references, dates, sales, quantity, price |
| ERP | [cust_az12.csv](../datasets/source_erp/cust_az12.csv) | 18,484 | Customer reference, birthdate, gender |
| ERP | [loc_a101.csv](../datasets/source_erp/loc_a101.csv) | 18,484 | Customer reference, country |
| ERP | [px_cat_g1v2.csv](../datasets/source_erp/px_cat_g1v2.csv) | 37 | Category ID, category, subcategory, maintenance flag |

Raw input records include values that need preparation. For example, customer names contain whitespace and customer references differ in format between source systems. No cleaned output count is claimed.

## Gold customer dimension

**Object:** `gold.dim_customers`, a view.

**Sources:** `silver.crm_cust_info`, joined to `silver.erp_cust_az12` and `silver.erp_loc_a101` on the prepared customer key.

| Output | Definition |
| :--- | :--- |
| `customer_key` | `ROW_NUMBER()` ordered by CRM `cst_id` |
| `customer_id` | CRM `cst_id` |
| `customer_number` | CRM `cst_key` |
| `first_name`, `last_name` | CRM `cst_firstname`, `cst_lastname` |
| `country` | ERP location `cntry` |
| `marital_status` | CRM `cst_marital_status` |
| `gender` | CRM `cst_gndr` when not `n/a`; otherwise ERP `gen`, defaulting to `n/a` |
| `birthdate` | ERP customer `bdate` |
| `create_date` | CRM `cst_create_date` |

The CRM gender rule is reproduced from the supplied SQL. It does not independently clean blank strings or all other unexpected values.

## Gold product dimension

**Object:** `gold.dim_products`, a view.

**Sources:** `silver.crm_prd_info`, left joined to `silver.erp_px_cat_g1v2` on `cat_id = id`.

**Filter:** current product records only, `prd_end_dt IS NULL`.

| Output | Definition |
| :--- | :--- |
| `product_key` | `ROW_NUMBER()` ordered by `prd_start_dt`, then `prd_key` |
| `product_id`, `product_number` | CRM `prd_id`, `prd_key` |
| `product_name` | CRM `prd_nm` |
| `category_id` | Prepared CRM product `cat_id` |
| `category`, `subcategory`, `maintenance` | ERP category `cat`, `subcat`, `maintenance` |
| `cost`, `product_line`, `start_date` | CRM `prd_cost`, `prd_line`, `prd_start_dt` |

## Gold sales fact

**Object:** `gold.fact_sales`, a view, despite the script's fact-table comment.

**Source:** `silver.crm_sales_details`.

| Output | Definition |
| :--- | :--- |
| `order_number` | `sls_ord_num` |
| `product_key` | Product-dimension match on `sls_prd_key = product_number` |
| `customer_key` | Customer-dimension match on `sls_cust_id = customer_id` |
| `order_date`, `shipping_date`, `due_date` | `sls_order_dt`, `sls_ship_dt`, `sls_due_dt` |
| `sales_amount`, `quantity`, `price` | `sls_sales`, `sls_quantity`, `sls_price` |

Each output row is derived from a Silver sales record, subject to join matches. The supplied SQL does not aggregate sales and does not enforce uniqueness of the dimension join columns. Duplicate matches can multiply rows. Do not assume `order_number` alone is a unique row key.

The Gold view exposes `sls_sales`; it does not calculate sales as quantity multiplied by price. Any such derivation or repair would belong to the missing Silver implementation.

## Modeling notes

- All three Gold objects are views, not stored fact/dimension tables.
- Join lines in the diagram are logical relationships, not enforced foreign-key constraints.
- Surrogate keys are calculated with `ROW_NUMBER()` at query time and can shift when underlying records change.
- `LEFT JOIN` retains unmatched sales records with null dimension keys.
- No end-to-end execution, output row count or quality-check result is claimed.

[Back to README](../README.md) · [Run guide](run-guide.md)
