Quality Check
```sql
SELECT DISTINCT gender
FROM gold.dim_customers  --Clean Data
```
#
SELECT * FROM gold.fact_sales;
```
Foreign Key Integrity(Dimensions)
Check if all dimension tables can successfully join to the fact table
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
```
