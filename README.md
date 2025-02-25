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
	cat_id VARCHAR(50),
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
Check unwanted spaces in string values
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

--In this data warehouse, I aim to store clear and meaningful values rather than using abbreviated terms and
--use the default value 'n/a' for missing values.
--Use CASE statement to make the data more meaningful
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
	END  cst_marital_status,
	CASE WHEN UPPER(TRIM(cst_gndr)) = 'F' THEN 'Female'
	     WHEN UPPER(TRIM(cst_gndr)) = 'M' THEN 'Male'
	     ELSE 'n/a'
	END cst_gndr,
	cst_create_date
	cst_create_date
FROM(
	SELECT *,
		ROW_NUMBER() OVER(PARTITION BY cst_id ORDER BY cst_create_date DESC) as flag_last
	FROM bronze.crm_cust_info
	WHERE cst_id IS NOT NULL
)WHERE flag_last = 1; 
```












