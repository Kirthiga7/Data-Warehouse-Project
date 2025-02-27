# GOLD LAYER
![image](https://github.com/user-attachments/assets/1332d0ff-d0fc-4f07-903a-225bd897d10b)

**Data Modeling**
A data model defines how data is structured, stored, and related within a database. It provides a blueprint for organizing data efficiently.
Type: Physical Data Model
**Dimensional Modeling**
Dimensional modeling is a technique used in data warehousing to structure data for fast retrieval and easy reporting. 

It organizes data into facts and dimensions, making it optimized for OLAP (Online Analytical Processing).
Type: Star Schema

Dimensions - Descriptive information that give context to the data. It answers Who? What? Where? Ex: Product Details

Facts- Quantitative information that represents events. It answers How much? How many? . It have
- Multiple IDs 
- Date information
- Measures & Numbers

**Create Customer Dimension**
For Customer Dimension, keep customer information table (crm_cust_info) as master table and join extra data about customer from 
extra customer information(erp_cust_az12) table and location of customer(erp_loc_a101) table.
```sql
SELECT
	ci.cst_id,
	ci.cst_key,
	ci.cst_firstname,
	ci.cst_lastname,
	ci.cst_marital_status,
	ci.cst_gndr,
	ci.cst_create_date,
	ca.bdate,
	ca.gen,
	la.cntry
FROM silver.crm_cust_info ci
LEFT JOIN silver.erp_cust_az12 ca
ON ci.cst_key = ca.cid
LEFT JOIN silver.erp_loc_a101 la
ON ci.cst_key = la.cid;
```
After joining the tables, I'm checking if any duplicates were introduced by the join logic.
```sql
SELECT cst_id, COUNT(*) FROM
(	SELECT *
 	FROM silver.crm_cust_info ci
	LEFT JOIN silver.erp_cust_az12 ca
	ON ci.cst_key = ca.cid
	LEFT JOIN silver.erp_loc_a101 la
	ON ci.cst_key = la.cid
) T
GROUP BY cst_id
HAVING COUNT(*) >1; --No Result (No Duplicates)
```
DATA INTEGRATION
Integrating Gender details from crm table and erp table.
```sql
SELECT DISTINCT
	ci.cst_gndr,
	ca.gen,
	CASE WHEN ci.cst_gndr != 'n/a' THEN ci.cst_gndr --CRM is master for gender info
	ELSE COALESCE(ca.gen,'n/a')
	END AS new_gen
FROM silver.crm_cust_info ci
LEFT JOIN silver.erp_cust_az12 ca
ON ci.cst_key = ca.cid
ORDER BY 1,2; 
```
Rename columns to friendly, meaningful names.

Sort the columns into logical groups to improve readability.

Create View as dimension customer and generate Surrogate Key 
```sql
CREATE VIEW gold.dim_customers AS
SELECT 
	ROW_NUMBER() OVER(ORDER BY cst_id) AS customer_key,
	ci.cst_id AS customer_id,
	ci.cst_key AS customer_name,
	ci.cst_firstname AS first_name,
	ci.cst_lastname AS last_name,
	la.cntry AS country,
	ci.cst_marital_status AS marital_status,
	CASE WHEN ci.cst_gndr != 'n/a' THEN ci.cst_gndr --CRM is master for gender info
	ELSE COALESCE(ca.gen,'n/a')
	END AS gender,
	ca.bdate AS birth_date,
	ci.cst_create_date AS create_date
FROM silver.crm_cust_info ci
LEFT JOIN silver.erp_cust_az12 ca
ON ci.cst_key = ca.cid
LEFT JOIN silver.erp_loc_a101 la
ON ci.cst_key = la.cid;

SELECT * FROM gold.dim_customers;
```
Quality Check
```sql
SELECT DISTINCT gender
FROM gold.dim_customers  --Clean Data
```
**Create Product Dimension**
For Product Dimension, keep crm_prd_info as master table and join erp_px_cat_g1v2 table
```sql
SELECT
	pn.prd_id,
 	pn.cat_id,
  	pn.prd_key,
   	pn.prd_nm,
    	pn.prd_cost,
     	pn.prd_line,
        pn.prd_start_date,
	pc.cat,
  	pc.subcat,
   	pc.maintenance
FROM silver.crm_prd_info pn
LEFT JOIN silver.erp_px_cat_g1v2 pc
ON pn.cat_id = pc.id
where prd_end_dt IS NULL; --FILTER out all historical data (If end_date is NULL then it is current information of the product)
```
Check Duplicates
```SQL
SELECT prd_key, COUNT(*)
FROM(
	SELECT *
 	FROM silver.crm_prd_info pn
	LEFT JOIN silver.erp_px_cat_g1v2 pc
	ON pn.cat_id = pc.id
	where prd_end_dt IS NULL ) T
 GROUP BY prd_key
 HAVING COUNT(*) > 1; --No Result (No Duplicates)
```
Rename columns to friendly, meaningful names.

Sort the columns into logical groups to improve readability.

Create View as dimension product and generate Surrogate Key 
```sql
CREATE VIEW gold.dim_products AS
SELECT
	ROW_NUMBER() OVER(ORDER BY pn.prd_start_dt, pn.prd_key) AS product_key, 
	pn.prd_id AS product_id,
	pn.prd_key AS product_number,
	pn.prd_nm AS product_name,
	pn.cat_id AS category_id,
	pc.cat AS category,
	pc.subcat AS subcategory,
	pc.maintenance,
	pn.prd_cost AS product_cost,
	pn.prd_line AS product_line,
	pn.prd_start_dt AS start_date
	FROM silver.crm_prd_info pn
LEFT JOIN silver.erp_px_cat_g1v2 pc
ON pn.cat_id = pc.id
where prd_end_dt IS NULL;
```
**Create Sales Fact**
Create Sales_Detail as Fact . Because it has multiple KEYS, DATES and MEASURES.
Use the dimension's surrogate keys instead of IDs to connect facts with dimension easily.
Give friendly name to columns and sort the columns into logical groups to improve readability.
```sql
CREATE VIEW gold.fact_sales AS
SELECT
	sd.sls_ord_num AS order_number,
	pr.product_key,     --Dimension Key
	cu.customer_key,    --Dimension Key
	sd.sls_order_dt AS order_date,
	sd.sls_ship_dt AS ship_date,
	sd.sls_due_dt AS due_date,
	sd.sls_sales AS sales_amount,
	sd.sls_quantity AS quantity,
	sd.sls_price AS price
FROM silver.crm_sales_details sd
LEFT JOIN gold.dim_products pr
ON sd.sls_prd_key = pr.product_number
LEFT JOIN gold.dim_customers cu
ON sd.sls_cust_id = cu.customer_id;

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
