# Data-Warehouse-Project
**Data Warehouse**

A subject-oriented, integrated, time-variant, and non-volatile collection of data in support of management's decision making process.
It includes
- ETL Processing
- Data Architecture
- Data Integration
- Data Cleansing
- Data Load
- Data Modeling

# ETL Processing
![image](https://github.com/user-attachments/assets/d5ed9044-4efa-471f-a73f-c02bc133ef2b)

**Extract (E)** : Data is retrieved from various sources like databases
- Method: Pull Extraction
- Type: Full Extraction 
- Technique: File Parsing

**Transform (T)** – Data is cleansed, formatted, and structured before loading

It includes Data enrichment, Data integration, Derived column, Data normalization & standardization, Business rules& logic and Data aggregation.

Data Cleansing include remove duplicates, data filtering, data type casting, handling missing data, invalid values and unwanted spaces.

**Load (L)** – The transformed data is loaded into a data warehouse
- Processing type: Batch processing
- Load method: Full load - Truncate and Insert
- Slowly Changing Simensions(SCD) - SCD1-overwrite(updating the content of data warehouse)

Between Source and Target We have multiple layers depend on building the data warehouse

# Data Architecture

Data Architecture is the blueprint that defines how data is collected, stored, processed, and used within an organization.

It ensures data is organized, secure, and accessible for analytics, decision-making, and operational processes.

Type: Data Warehouse

Approach: Medallion Architecture

**Design the layers**

![image](https://github.com/user-attachments/assets/9f1c6eca-3685-4853-92e6-8b27d01d01c8) 

**Diagram of the Architecture**

![image](https://github.com/user-attachments/assets/0bbc85bd-7b5a-4f1e-8cdc-e89d82f29073)

**Project Initialization**

**Naming Conventions**
 Using snake_case, with lowercase letters and underscores (_) to separate words.

**Table Naming Conventions**
 
All names must start with the source system name, and table names must match their original names without renaming.

(sourcesystem)_(entity)

(sourcesystem): Name of the source system (e.g., crm, erp).

(entity): Exact table name from the source system.

Example: crm_customer_info → Customer information from the CRM system.

# Creating Database and Schemas

```sql
--Create Database
DROP DATABASE IF EXISTS DataWarehouse
CREATE DATABASE DataWarehouse

--Create Schema
CREATE SCHEMA BRONZE;
CREATE SCHEMA SILVER;
CREATE SCHEMA GOLD;
```
# Bronze Layer
![image](https://github.com/user-attachments/assets/bf4e2bd7-6c2d-42f4-bd3a-21cc51d0133c)


**Create DDL for Tables**

Data Definition Language defines the structure of database tables

```sql
DROP TABLE IF EXISTS bronze.crm_cust_info;
CREATE TABLE bronze.crm_cust_info(
	cst_id INT,
	cst_key	VARCHAR(20),
	cst_firstname VARCHAR(20),
	cst_lastname VARCHAR(30),
	cst_marital_status VARCHAR(20),
	cst_gndr VARCHAR(20),
	cst_create_date DATE
);

DROP TABLE IF EXISTS bronze.crm_prd_info;
CREATE TABLE bronze.crm_prd_info(
	prd_id 	INT,
	prd_key	VARCHAR(50),
	prd_nm	VARCHAR(50),
	prd_cost INT,
	prd_line VARCHAR(20),
	prd_start_dt DATE,
	prd_end_dt DATE
);

DROP TABLE IF EXISTS bronze.crm_sales_details;
CREATE TABLE bronze.crm_sales_details(
	sls_ord_num	VARCHAR(50),
	sls_prd_key VARCHAR(50),
	sls_cust_id	INT,
	sls_order_dt DATE,
	sls_ship_dt	DATE,
	sls_due_dt DATE,
	sls_sales INT,
	sls_quantity INT,
	sls_price INT
);

DROP TABLE IF EXISTS bronze.erp_cust_az12;
CREATE TABLE bronze.erp_cust_az12(
	CID VARCHAR(50),
	BDATE DATE,
	GEN VARCHAR(10)
);

DROP TABLE IF EXISTS bronze.erp_px_cat_g1v2;
CREATE TABLE bronze.erp_px_cat_g1v2(
	id VARCHAR(10),
	cat VARCHAR(20),
	subcat VARCHAR(30),
	maintenance VARCHAR(5)
);

DROP TABLE IF EXISTS bronze.erp_loc_a101;
CREATE TABLE bronze.erp_loc_a101(
	cid	VARCHAR(20),
	cntry VARCHAR(20)
);

```


**Import data from CSV files to respective Table**

**Data Flow**

![image](https://github.com/user-attachments/assets/9bb6a3f9-1dbb-44db-a5e7-d2a28a631384)

# Silver Layer
![image](https://github.com/user-attachments/assets/756c1d2a-a1f4-4c10-a210-e42eae65f5a4)

**Explore and Understand Data**

Exploring all columns in every table and relationship between these tables

**INTEGRATION MODEL:**
![image](https://github.com/user-attachments/assets/48132423-c0b4-4269-be05-af7aba3c090e)

**Metadata Columns:**
Extra Columns added by data engineers that do not orginate from the source data.

Here I add create_date to track the record's load timestamp

**Create DDL for Tables**

```SQL
DROP TABLE IF EXISTS silver.crm_cust_info;
CREATE TABLE silver.crm_cust_info(
	cst_id INT,
	cst_key	VARCHAR(20),
	cst_firstname VARCHAR(20),
	cst_lastname VARCHAR(30),
	cst_marital_status VARCHAR(20),
	cst_gndr VARCHAR(20),
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
	sls_ord_num	VARCHAR(50),
	sls_prd_key VARCHAR(50),
	sls_cust_id	INT,
	sls_order_dt DATE,
	sls_ship_dt	DATE,
	sls_due_dt DATE,
	sls_sales INT,
	sls_quantity INT,
	sls_price INT,
	dwh_create_date TIMESTAMP DEFAULT NOW()
);

DROP TABLE IF EXISTS silver.erp_cust_az12;
CREATE TABLE silver.erp_cust_az12(
	CID VARCHAR(50),
	BDATE DATE,
	GEN VARCHAR(10),
	dwh_create_date TIMESTAMP DEFAULT NOW()
);
DROP TABLE IF EXISTS silver.erp_px_cat_g1v2;
CREATE TABLE silver.erp_px_cat_g1v2(
	id VARCHAR(10),
	cat VARCHAR(20),
	subcat VARCHAR(30),
	maintenance VARCHAR(5),
	dwh_create_date TIMESTAMP DEFAULT NOW()
);
DROP TABLE IF EXISTS silver.erp_loc_a101;
CREATE TABLE silver.erp_loc_a101(
	cid	VARCHAR(20),
	cntry VARCHAR(20),
	dwh_create_date TIMESTAMP DEFAULT NOW()
);

```
**Clean crm_cust_info**

A Primary Key must be unique and not NULL
Remove Duplicates: Ensure omly one record per entity by identifying and retaining the most relevant row
```sql
--Check for NULLs or Duplicates in Primary Key
--Expectations: No Result

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
Check unwanted spaces in string values.

Remove unnecessary spaces to ensure data consistency, and uniformity across all records.
```sql
--Check for unwanted spaces
--Expectations: No Result

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
Check the consistency of values in low cardinality columns
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
Load the updated data
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
Quality Check of silver table

Re-run the quality  check queries from bronze layer to verify the quality of data in the silver laye.
```sql
SELECT 
	cst_id,
	COUNT(*)
FROM silver.crm_cust_info
GROUP BY cst_id
HAVING COUNT(*) > 1 OR cst_id IS NULL; --No Result

SELECT cst_firstname
FROM silver.crm_cust_info
WHERE cst_firstname != TRIM(cst_firstname); --No Result

SELECT 
	DISTINCT cst_gndr
FROM silver.crm_cust_info; --No Result

SELECT * FROM silver.crm_cust_info
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
```
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
