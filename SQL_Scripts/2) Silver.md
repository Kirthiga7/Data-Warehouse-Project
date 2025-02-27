# Silver Layer
![image](https://github.com/user-attachments/assets/756c1d2a-a1f4-4c10-a210-e42eae65f5a4)

**Explore and Understand Data**

Exploring all columns in every table and relationship between these tables

**Metadata Columns:**
Extra Columns added by data engineers that do not orginate from the source data.

Here I add create_date to track the record's load timestamp

**Create DDL for Tables**

```SQL
DROP TABLE IF EXISTS silver.crm_cust_info;
CREATE TABLE silver.crm_cust_info(
	cst_id INT,
	cst_key	VARCHAR(50),
	cst_firstname VARCHAR(50),
	cst_lastname VARCHAR(50),
	cst_marital_status VARCHAR(50),
	cst_gndr VARCHAR(50),
	cst_create_date DATE,
	dwh_create_date TIMESTAMP DEFAULT NOW()
);

DROP TABLE IF EXISTS silver.crm_prd_info;
CREATE TABLE silver.crm_prd_info(
	prd_id 	INT,
	cat_id VARCHAR(50),
	prd_key	VARCHAR(50),
	prd_nm	VARCHAR(50),
	prd_cost INT,
	prd_line VARCHAR(20),
	prd_start_dt DATE,
	prd_end_dt DATE,
	dwh_create_date TIMESTAMP DEFAULT NOW()
);

DROP TABLE IF EXISTS silver.crm_sales_details;
CREATE TABLE silver.crm_sales_details(
	sls_ord_num VARCHAR(50),
	sls_prd_key VARCHAR(50),
	sls_cust_id INT,
	sls_order_dt DATE,
	sls_ship_dt DATE,
	sls_due_dt DATE,
	sls_sales INT,
	sls_quantity INT,
	sls_price INT,
	dwh_create_date TIMESTAMP DEFAULT NOW()
);

DROP TABLE IF EXISTS silver.erp_cust_az12;
CREATE TABLE silver.erp_cust_az12(
	cid VARCHAR(50),
	bdate DATE,
	gen VARCHAR(50),
	dwh_create_date TIMESTAMP DEFAULT NOW()
);
DROP TABLE IF EXISTS silver.erp_px_cat_g1v2;
CREATE TABLE silver.erp_px_cat_g1v2(
	id VARCHAR(50),
	cat VARCHAR(50),
	subcat VARCHAR(50),
	maintenance VARCHAR(50),
	dwh_create_date TIMESTAMP DEFAULT NOW()
);
DROP TABLE IF EXISTS silver.erp_loc_a101;
CREATE TABLE silver.erp_loc_a101(
	cid VARCHAR(50),
	cntry VARCHAR(50),
	dwh_create_date TIMESTAMP DEFAULT NOW()
);

```


# Clean crm_cust_info

**A Primary Key must be unique and not NULL**

Remove Duplicates: Ensure only one record per entity by identifying and retaining the most relevant row
```sql
SELECT 
	cst_id,
	COUNT(*)
FROM bronze.crm_cust_info
GROUP BY cst_id
HAVING COUNT(*) > 1 OR cst_id IS NULL;

--Create row_number to remove duplicates
SELECT *
FROM(
SELECT *,
	ROW_NUMBER() OVER(PARTITION BY cst_id ORDER BY cst_create_date DESC) as flag_last
	FROM bronze.crm_cust_info
)f
WHERE flag_last != 1;
```
**Check unwanted spaces in string values.**

Remove unnecessary spaces to ensure data consistency, and uniformity across all records.
```sql
SELECT cst_firstname
FROM bronze.crm_cust_info
WHERE cst_firstname != TRIM(cst_firstname);

SELECT cst_lastname
FROM bronze.crm_cust_info
WHERE cst_lastname != TRIM(cst_lastname);

SELECT cst_gndr
FROM bronze.crm_cust_info
WHERE cst_gndr != TRIM(cst_gndr); --Quality of gender is good as it does not have any unwanted spaces

--Trim the column
TRIM(cst_firstname) AS cst_firstname;
TRIM(cst_lastname) AS cst_lastname;
```
**Check the consistency of values in low cardinality columns**
```sql
--Data Consistency & Standardization
SELECT 
	DISTINCT cst_gndr
FROM bronze.crm_cust_info;

SELECT 
	DISTINCT cst_marital_status
FROM bronze.crm_cust_info;

/* 	In this data warehouse, I aim to store clear and meaningful values rather than using abbreviated terms and
	use the default value 'n/a' for missing values.
	Using CASE statement to make the data more meaningful	 */

CASE WHEN UPPER(TRIM(cst_marital_status)) = 'S' THEN 'Single'
     WHEN UPPER(TRIM(cst_marital_status)) = 'M' THEN 'Married'
     ELSE 'n/a'
END  cst_marital_status;

CASE WHEN UPPER(TRIM(cst_gndr)) = 'F' THEN 'Female'
     WHEN UPPER(TRIM(cst_gndr)) = 'M' THEN 'Male'
     ELSE 'n/a'
END cst_gndr;
```
**Load the updated data**
```sql
INSERT INTO silver.crm_cust_info
(cst_id,cst_key,cst_firstname,cst_lastname,cst_marital_status,cst_gndr,cst_create_date)
SELECT
	cst_id,
	cst_key,
        TRIM(cst_firstname) AS cst_firstname,
	TRIM(cst_lastname) AS cst_lastname,
	CASE WHEN UPPER(TRIM(cst_marital_status)) = 'S' THEN 'Single'
	     WHEN UPPER(TRIM(cst_marital_status)) = 'M' THEN 'Married'
	     ELSE 'n/a'
	END  cst_marital_status, --Normalize marital status values to readable format
	CASE WHEN UPPER(TRIM(cst_gndr)) = 'F' THEN 'Female'
	     WHEN UPPER(TRIM(cst_gndr)) = 'M' THEN 'Male'
	     ELSE 'n/a' --Normalize gender values to readable format
	END cst_gndr,
	cst_create_date
	cst_create_date
FROM(
	SELECT *,
		ROW_NUMBER() OVER(PARTITION BY cst_id ORDER BY cst_create_date DESC) as flag_last
	FROM bronze.crm_cust_info
	WHERE cst_id IS NOT NULL
)WHERE flag_last = 1; --Select the most recent record per customer
```


**Clean & Load crm_prd_info**

--Check primary key is don't have duplicate and NULL values
```sql
SELECT 
	prd_id,
	COUNT(*)
FROM bronze.crm_prd_info
GROUP BY prd_id
HAVING COUNT(*) > 1 OR prd_id IS NULL;  --No Result
```
prd_key → First five characters represent cat_id in erp_px_cat_g1v2

ERP Table → Uses an underscore (_) between parts.

CRM Table → Uses a hyphen (-) between parts.

prd_key - after five characters represent prd_key in crm_sales_details table
```sql
REPLACE(SUBSTRING(prd_key,1,5),'-','_') AS cat_id;
SUBSTRING(prd_key,7,LENGTH(prd_key)) AS prd_key;
```
CHECK FOR UNWANTED SPACES
```sql
SELECT prd_nm
FROM bronze.crm_prd_info
WHERE prd_nm != TRIM(prd_nm); -=-No Result
```
Check for nulls or negative numbers
```sql
SELECT prd_cost
FROM bronze.crm_prd_info
WHERE prd_cost < 0 or prd_cost IS NULL;

--Replace null with zero
COALESCE(prd_cost,0) AS prd_cost;
```
Data Standardization and consistency
```sql
SELECT DISTINCT prd_line
FROM bronze.crm_prd_info;

--Using CASE statement to make the data more meaningful

CASE UPPER(TRIM(prd_line))
	WHEN 'M' THEN 'Mountain'
	WHEN 'R' THEN 'Road'
	WHEN 'S' THEN 'Other Sales'
	WHEN 'T' THEN 'Touring'
	ELSE 'n/a'
END AS prd_line;
```
Check for Invalid Date Orders
```sql
--End date must not be earlier than the start date
SELECT *
FROM bronze.crm_prd_info
WHERE prd_end_dt < prd_start_dt;

/*  	Start date and end date are overlapping
	I derive the end date of current record from the start date of next record
	End Date = Start Date of the NEXT Record - 1      	*/

LEAD(prd_start_dt) OVER (PARTITION BY prd_key ORDER BY prd_start_dt)-1 AS prd_end_dt;
```
Load the updated data
```sql
INSERT INTO silver.crm_prd_info
(prd_id, cat_id, prd_key, prd_nm, prd_cost, prd_line, prd_start_dt, prd_end_dt)
SELECT
        prd_id,
	REPLACE(SUBSTRING(prd_key,1,5),'-','_') AS cat_id,
	SUBSTRING(prd_key,7,LENGTH(prd_key)) AS prd_key,
	prd_nm,
	COALESCE(prd_cost,0) AS prd_cost,
	CASE UPPER(TRIM(prd_line))
		 WHEN 'M' THEN 'Mountain'
	         WHEN 'R' THEN 'Road'
		 WHEN 'S' THEN 'Other Sales'
		 WHEN 'T' THEN 'Touring'
	         ELSE 'n/a'	 
	END AS prd_line,
	prd_start_dt,
	LEAD(prd_start_dt) OVER (PARTITION BY prd_key ORDER BY prd_start_dt)-1 AS prd_end_dt
FROM bronze.crm_prd_info;
```
Quality check
```sql
SELECT prd_cost
FROM silver.crm_prd_info
WHERE prd_cost < 0 or prd_cost IS NULL; --No Result

SELECT DISTINCT prd_line
FROM silver.crm_prd_info; --No Result

SELECT *
FROM silver.crm_prd_info
WHERE prd_end_dt < prd_start_dt; --No Result

SELECT * FROM silver.crm_prd_info;
```
**Clean & Load crm_ales_details**
Check for unwanted spaces
```sql
SELECT *
FROM bronze.crm_sales_details
WHERE sls_ord_num != TRIM(sls_ord_num);
```
Checking foreignkey is in another table
SELECT *
FROM bronze.crm_sales_details
WHERE sls_cust_id NOT IN(SELECT cst_id FROM silver.crm_cust_info); --No Result

SELECT *
FROM bronze.crm_sales_details
WHERE sls_prd_key NOT IN(SELECT prd_key FROM silver.crm_prd_info); --No Result
--So we can connect sales with customer and product table
```
Check for invalid date. Negative numbers or zeros can't be cast to a date
```sql
SELECT *
FROM bronze.crm_sales_details
WHERE sls_order_dt <= 0;

/*	Replace 0 with NULL
	The length of date must be 8. If not, change it to NULL
 	CAST the type to DATE     		*/

CASE WHEN sls_order_dt = 0 OR LENGTH(sls_order_dt::TEXT) != 8 THEN NULL
     ELSE CAST(CAST(sls_order_dt AS VARCHAR) AS DATE)
END AS sls_order_dt;

CASE WHEN sls_ship_dt = 0 OR LENGTH(sls_ship_dt::TEXT) != 8 THEN NULL
     ELSE CAST(CAST(sls_ship_dt AS VARCHAR) AS DATE)
END AS sls_ship_dt;

CASE WHEN sls_due_dt = 0 OR LENGTH(sls_due_dt::TEXT) != 8 THEN NULL
     ELSE CAST(CAST(sls_due_dt AS VARCHAR) AS DATE)
END AS sls_due_dt;

Order date must always be earlier than shipping date or due date
```sql
SELECT *
FROM bronze.crm_sales_details
WHERE sls_order_dt > sls_ship_dt	
OR  sls_order_dt > sls_due_dt; --No Result
```
Business Rules:

Sales = Quamtity * Price . Sales can't be negative, zeros or NULLs
```sql
SELECT
	sls_sales,
	sls_quantity,
	sls_price
FROM bronze.crm_sales_details
WHERE sls_sales != sls_quantity * sls_price
OR sls_sales <=0 OR sls_quantity <=0 OR sls_price <= 0
OR sls_sales IS NULL OR sls_quantity IS NULL OR sls_price IS NULL
ORDER BY sls_sales, sls_quantity, sls_price;

/*	If Sales is negative, zero, or null derive it using Quantity and Price
	If Price is zero or negative, calculate it using Sales and Quantity
	If Price is negative, convert it to positive value		 */

SELECT
	sls_sales AS old_sls_sales,
	sls_quantity ,
	sls_price AS old_sls_price,
        CASE WHEN sls_sales IS NULL OR sls_sales <= 0 OR sls_sales != sls_quantity * ABS(sls_price)
	     		THEN sls_quantity * ABS(sls_price)
             ELSE sls_sales
	END AS sls_sales,
        sls_quantity,
	CASE WHEN sls_price IS NULL OR sls_price <= 0
		 THEN sls_sales / NULLIF(sls_quantity,0)
	ELSE sls_price
	END AS sls_price
FROM bronze.crm_sales_details
WHERE sls_sales != sls_quantity * sls_price
OR sls_sales <=0 OR sls_quantity <=0 OR sls_price <= 0
OR sls_sales IS NULL OR sls_quantity IS NULL OR sls_price IS NULL
ORDER BY sls_sales, sls_quantity, sls_price;
```

Load the updated data
```sql
INSERT INTO silver.crm_sales_details
(sls_ord_num, sls_prd_key, sls_cust_id,sls_order_dt, sls_ship_dt, sls_due_dt, sls_sales, sls_quantity, sls_price)
SELECT
	sls_ord_num	,
	sls_prd_key ,
	sls_cust_id	,
	CASE WHEN sls_order_dt = 0 OR LENGTH(sls_order_dt::TEXT) != 8 THEN NULL
	     ELSE CAST(CAST(sls_order_dt AS VARCHAR) AS DATE)
	END AS sls_order_dt,
	CASE WHEN sls_ship_dt = 0 OR LENGTH(sls_ship_dt::TEXT) != 8 THEN NULL
	     ELSE CAST(CAST(sls_ship_dt AS VARCHAR) AS DATE)
	END AS sls_ship_dt,
	CASE WHEN sls_due_dt = 0 OR LENGTH(sls_due_dt::TEXT) != 8 THEN NULL
	     ELSE CAST(CAST(sls_due_dt AS VARCHAR) AS DATE)
	END AS sls_due_dt,
	CASE WHEN sls_sales IS NULL OR sls_sales <= 0 OR sls_sales != sls_quantity * ABS(sls_price)
	     THEN sls_quantity * ABS(sls_price)
	ELSE sls_sales
	END AS sls_sales,
	sls_quantity,
	CASE WHEN sls_price IS NULL OR sls_price <= 0
		 THEN sls_sales / NULLIF(sls_quantity,0)
	ELSE sls_price
	END AS sls_price
FROM bronze.crm_sales_details;
```
Quality Check
```sql
SELECT
	sls_sales,
	sls_quantity,
	sls_price
FROM silver.crm_sales_details
WHERE sls_sales != sls_quantity * sls_price
OR sls_sales <=0 OR sls_quantity <=0 OR sls_price <= 0
OR sls_sales IS NULL OR sls_quantity IS NULL OR sls_price IS NULL
ORDER BY sls_sales, sls_quantity, sls_price;   --No Result!

SELECT * FROM silver.crm_sales_details
```
**Clean & Load erp_cust_az12**
```sql
SELECT *
FROM bronze.erp_cust_az12;
```
cid has character 'NAS' in starting, which is not in cst_key of customer tabl e(ctm_cust_info) . So remove the characters 
```sql
CASE WHEN cid LIKE 'NAS%' THEN SUBSTRING(cid,4,LENGTH(cid))
     ELSE cid
END AS cid;
```
--Identify out of range date
```sql
SELECT DISTINCT bdate
FROM bronze.erp_cust_az12
WHERE bdate < '1924-01-01' OR bdate > NOW();

--Remove the birthdays in the future
CASE WHEN bdate > NOW() THEN NULL
     ELSE bdate
END AS bdate;
```
--Data standardization & consistency
SELECT 
	DISTINCT gen,
	CASE WHEN UPPER(TRIM(gen)) IN ('F','FEMALE') THEN 'Female'
	     WHEN UPPER(TRIM(gen)) IN ('M','MALE') THEN 'Male'
	     ELSE 'n/a'
	END AS gen
FROM bronze.erp_cust_az12;
```
Load the updated data
```sql
INSERT INTO silver.erp_cust_az12(cid, bdate, gen)
SELECT 
	CASE WHEN cid LIKE 'NAS%' THEN SUBSTRING(cid,4,LENGTH(cid))
	     ELSE cid
	END AS cid,
	CASE WHEN bdate > NOW() THEN NULL
		 ELSE bdate
	END AS bdate,
	CASE WHEN UPPER(TRIM(gen)) IN ('F','FEMALE') THEN 'Female'
		 WHEN UPPER(TRIM(gen)) IN ('M','MALE') THEN 'Male'
	ELSE 'n/a'
	END AS gen
FROM bronze.erp_cust_az12;
```
Quality Check
```sql
SELECT DISTINCT bdate
FROM silver.erp_cust_az12
WHERE bdate > NOW(); --No Result

SELECT DISTINCT gen
FROM silver.erp_cust_az12; --Clean Values

SELECT DISTINCT cid
FROM silver.erp_cust_az12
WHERE cid NOT IN (SELECT cst_key FROM silver.crm_cust_info ) --No Result

SELECT * FROM silver.erp_cust_az12;
```

**Clean & Load erp_loc_a101**
```sql
SELECT * FROM  bronze.erp_loc_a101;
```
cid → Uses a hyphen (-) between parts.

But cst_key in customer table doesn't have anything in between. So remove the hyphen
```sql
REPLACE(cid,'-','')	cid;

--Data standardization & consistency
SELECT DISTINCT cntry FROM silver.erp_cust_az12;

--To make it more standardized
SELECT
	DISTINCT cntry AS old_cnty,
	CASE WHEN TRIM(cntry) = 'DE' THEN 'Germany'
		 WHEN TRIM(cntry) IN ('US','USA') THEN 'United States'
		 WHEN TRIM(cntry) = '' OR cntry IS NULL THEN 'n/a'
	ELSE TRIM(cntry)
	END AS cntry
FROM bronze.erp_loc_a101;
```
Load the updated date
```sql
INSERT INTO silver.erp_loc_a101(cid, cntry)
SELECT
	REPLACE(cid,'-','')	cid,
	CASE WHEN TRIM(cntry) = 'DE' THEN 'Germany'
		 WHEN TRIM(cntry) IN ('US','USA') THEN 'United States'
		 WHEN TRIM(cntry) = '' OR cntry IS NULL THEN 'n/a'
	ELSE TRIM(cntry)
	END AS cntry
FROM bronze.erp_loc_a101;
```
QUALITY CHECK
```sql
SELECT DISTINCT cntry 
FROM silver.erp_loc_a101; --Clean values
	
SELECT DISTINCT cid
FROM silver.erp_loc_a101
WHERE cid NOT IN (SELECT cst_key FROM silver.crm_cust_info ); --No Result
```
**Clean & Load erp_px_cat_g1v2**
Check for NULL values
```sql
SELECT 
	id,
	COUNT(*)
FROM bronze.erp_px_cat_g1v2
GROUP BY id
HAVING COUNT(*) > 1;  --No Result
```
Check for unwanted space
```sql
SELECT * FROM bronze.erp_px_cat_g1v2
WHERE cat != TRIM(cat) OR subcat != TRIM(subcat) OR maintenance != TRIM(maintenance);  --No Result
```
Data standardization
```sql
SELECT DISTINCT cat
FROM bronze.erp_px_cat_g1v2; --Good Data

SELECT DISTINCT subcat
FROM bronze.erp_px_cat_g1v2; --Good Data

SELECT DISTINCT maintenance
FROM bronze.erp_px_cat_g1v2;  --Good Data
```
This table has very good data. So there is no need for transformation.

Load the data
```sql
INSERT INTO silver.erp_px_cat_g1v2(id,cat,subcat,maintenance)
SELECT
	id,
	cat,
	subcat,
	maintenance 
FROM bronze.erp_px_cat_g1v2;
