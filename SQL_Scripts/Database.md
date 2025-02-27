# Create Database and Schemas

Script Purpose:
   
    This script creates a new database named 'DataWarehouse' after checking if it already exists. 
    If the database exists, it is dropped and recreated. Additionally, the script sets up three schemas 
    within the database: 'bronze', 'silver', and 'gold'.


**Create Database**
```sql
DROP DATABASE IF EXISTS DataWarehouse;
CREATE DATABASE DataWarehouse;
```
**Create Schema**
```sql
CREATE SCHEMA BRONZE;
CREATE SCHEMA SILVER;
CREATE SCHEMA GOLD;
```
    

**Naming Conventions**
 
 Using snake_case, with lowercase letters and underscores (_) to separate words.

**Table Naming Conventions**
 
All names must start with the source system name, and table names must match their original names without renaming.

(sourcesystem)_(entity)

(sourcesystem): Name of the source system (e.g., crm, erp).

(entity): Exact table name from the source system.

Example: crm_customer_info → Customer information from the CRM system.

