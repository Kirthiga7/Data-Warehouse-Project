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
![image](https://github.com/user-attachments/assets/ff53aaaf-3e29-416b-9544-9dfcdb0a71bf)

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







