# Gold Layer - Dimensional Model (Star Schema)

## Overview

The Gold layer uses a Star Schema design, consolidating six Silver tables into three optimized tables.

**Purpose:** Support BI analytics and improve query performance.

## Fact Table

### fact_sales

The main fact table that stores each sales transaction.

**Primary Key:** `sls_ord_num`

**Columns:**

* `sls_ord_num` (string) – Order number, primary key
* `sls_cust_id` (integer) – Customer ID, foreign key → `dim_customers`
* `sls_prd_key` (string) – Product key, foreign key → `dim_products`
* `sls_order_dt` (date) – Order date
* `sls_ship_dt` (date) – Shipping date
* `sls_due_dt` (date) – Due date
* `sls_quantity` (integer) – Quantity (measure)
* `sls_price` (integer) – Unit price (measure)
* `sls_sales` (integer) – Sales amount (measure)

**Row Count:** 27,659

---

## Dimension Tables

### dim_customers

Customer dimension table that combines CRM, ERP, and location data.

**Primary Key:** `cst_id`

**Columns:**

* `cst_id` (integer) – Customer ID, primary key
* `cst_firstname` (string) – First name
* `cst_lastname` (string) – Last name
* `cst_marital_status` (string) – Marital status
* `cst_gndr` (string) – Gender
* `cst_create_date` (date) – Customer creation date
* `BDATE` (date) – Birth date (from `erp_customers`)
* `CNTRY` (string) – Country (from `locations`)

**Data Sources:**

* Base data: `silver.customers`
* Additional data: `silver.erp_customers` (`BDATE`)
* Additional data: `silver.locations` (`CNTRY`)

**Row Count:** 18,484

---

### dim_products

Product dimension table that combines product and category information.

**Primary Key:** `prd_key`

**Columns:**

* `prd_key` (string) – Product key, primary key
* `prd_id` (integer) – Product ID
* `prd_nm` (string) – Product name
* `prd_cost` (integer) – Product cost
* `prd_line` (string) – Product line
* `prd_start_dt` (date) – Start date
* `prd_end_dt` (date) – End date
* `CAT` (string) – Product category (from `categories`)
* `SUBCAT` (string) – Product subcategory (from `categories`)
* `MAINTENANCE` (string) – Maintenance flag (from `categories`)

**Data Sources:**

* Base data: `silver.products`
* Additional data: `silver.categories` (`CAT`, `SUBCAT`, `MAINTENANCE`)

**Row Count:** 397

---

## Design Decisions

### 1. Denormalization

* `dim_customers` contains data consolidated from three Silver tables.
* `dim_products` contains data consolidated from two Silver tables.
* Purpose: Reduce the number of joins required during query execution.

### 2. Join Strategy

* `fact_sales` → `dim_customers`: **Inner Join** (every sales record must have a valid customer).
* `fact_sales` → `dim_products`: **Inner Join** (every sales record must have a valid product).
* Silver tables → Dimension tables: **Left Join** (to preserve all base records).

### 3. Primary Keys

* `fact_sales`: `sls_ord_num`
* `dim_customers`: `cst_id`
* `dim_products`: `prd_key`

### 4. Performance Optimization

* Reduced the model from **6 Silver tables** to **3 Gold tables**.
* Reduced query-time joins from **6 joins** to **2 joins**.
* Expected query performance improvement: **5–10× faster**, depending on workload and query complexity.

