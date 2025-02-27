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
FROM silver.crm_cust_info; --Clean Data
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
