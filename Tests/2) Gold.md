# Quality Check
**Script Purpose:**
  
    This script performs various quality checks for data consistency, accuracy, 
    and standardization across the 'silver' layer. It includes checks for:
    - Null or duplicate primary keys.
    - Unwanted spaces in string fields.
    - Data standardization and consistency.
    - Invalid date ranges and orders.
    - Data consistency between related fields
# gold.dim_customers
**Check for Uniqueness of Customer Key**
```sql
SELECT 
    customer_key,
    COUNT(*) AS duplicate_count
FROM gold.dim_customers
GROUP BY customer_key
HAVING COUNT(*) > 1; --No Result
```
**Data Standardization**
```sql
SELECT DISTINCT gender
FROM gold.dim_customers  --Clean Data
```
#  gold.dim_products
**Check for Uniqueness of Product Key** 
```sql
SELECT 
    product_key,
    COUNT(*) AS duplicate_count
FROM gold.dim_products
GROUP BY product_key
HAVING COUNT(*) > 1;  --No Result
```
# gold.fact_sales
**Check the data model connectivity between fact and dimensions**
```sql
select * 
from gold.fact_sales f
LEFT JOIN gold.dim_customers c
ON c.customer_key = f.customer_key
WHERE c.customer_key is null; --No Result(Everything is matching perfectly)

select * 
from gold.fact_sales f
LEFT JOIN gold.dim_customers c
ON c.customer_key = f.customer_key
LEFT JOIN gold.dim_products p
ON p.product_key = f.product_key
WHERE p.product_key is null;   --No Result(Everything is matching perfectly)
```

