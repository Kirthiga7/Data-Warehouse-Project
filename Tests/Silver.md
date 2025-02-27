# Quality Check
**Script Purpose:**
  
    This script performs various quality checks for data consistency, accuracy, 
    and standardization across the 'silver' layer. It includes checks for:
    - Null or duplicate primary keys.
    - Unwanted spaces in string fields.
    - Data standardization and consistency.
    - Invalid date ranges and orders.
    - Data consistency between related fields
# crm_cust_info Table
**Check for NULLs or Duplicates in Primary Key**
```sql
SELECT 
	cst_id,
	COUNT(*)
FROM silver.crm_cust_info
GROUP BY cst_id
HAVING COUNT(*) > 1 OR cst_id IS NULL; --No Result
```
**Check for Unwanted Spaces**
```
SELECT cst_firstname
FROM silver.crm_cust_info
WHERE cst_firstname != TRIM(cst_firstname); --No Result
```
**Data Standardization & Consistency**
```sql
SELECT 
	DISTINCT cst_gndr
FROM silver.crm_cust_info; --Clean Values
```
# crm_prd_info Table
**Check for NULLs or Duplicates in Primary Key**
```sql
SELECT 
    prd_id,
    COUNT(*) 
FROM silver.crm_prd_info
GROUP BY prd_id
HAVING COUNT(*) > 1 OR prd_id IS NULL; --No Result
```
**Check for NULLs or Negative Values in Cost**
```sql
SELECT prd_cost
FROM silver.crm_prd_info
WHERE prd_cost < 0 or prd_cost IS NULL; --No Result
```
**Data Standardization & Consistency**
```sql
SELECT DISTINCT prd_line
FROM silver.crm_prd_info; --Clean Data
```
**Check for Invalid Date Orders (Start Date > End Date)**
```sql
SELECT *
FROM silver.crm_prd_info
WHERE prd_end_dt < prd_start_dt; --No Result
```
# crm_sales_details Table
**Check Data Consistency: Sales = Quantity * Price**
```sql
SELECT
	sls_sales,
	sls_quantity,
	sls_price
FROM silver.crm_sales_details
WHERE sls_sales != sls_quantity * sls_price
OR sls_sales <=0 OR sls_quantity <=0 OR sls_price <= 0
OR sls_sales IS NULL OR sls_quantity IS NULL OR sls_price IS NULL
ORDER BY sls_sales, sls_quantity, sls_price;   --No Result
```
**Check for Invalid Date Orders (Order Date > Shipping/Due Dates)**
```sql
SELECT 
    * 
FROM silver.crm_sales_details
WHERE sls_order_dt > sls_ship_dt 
   OR sls_order_dt > sls_due_dt;
```
# erp_cust_az12 Table
**Identify Out-of-Range Dates**
```sql
SELECT DISTINCT bdate
FROM silver.erp_cust_az12
WHERE bdate > NOW(); --No Result
```
**Check Data Consistency**
```sql
SELECT DISTINCT gen
FROM silver.erp_cust_az12; --Clean Values

SELECT DISTINCT cid
FROM silver.erp_cust_az12
WHERE cid NOT IN (SELECT cst_key FROM silver.crm_cust_info ) --No Result
```
# silver.erp_loc_a101 Table
**Check Data Consistency**
```sql
SELECT DISTINCT cntry 
FROM silver.erp_loc_a101; --Clean values
	
SELECT DISTINCT cid
FROM silver.erp_loc_a101
WHERE cid NOT IN (SELECT cst_key FROM silver.crm_cust_info ); --No Result
```
# erp_px_cat_g1v2 Table
