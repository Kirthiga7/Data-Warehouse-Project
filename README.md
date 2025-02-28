# Data Warehouse Project
This project demonstrates a comprehensive data warehousing and analytics solution, from building a data warehouse to generating actionable insights. 

**Project Overview**

This project involves:

- Data Architecture: Designing a Modern Data Warehouse Using Medallion Architecture Bronze, Silver, and Gold layers.
- ETL Pipelines: Extracting, transforming, and loading data from source systems into the warehouse.
- Data Modeling: Developing fact and dimension tables optimized for analytical queries.

**Data Warehouse**

A subject-oriented, integrated, time-variant, and non-volatile collection of data in support of management's decision making process.


![image](https://github.com/user-attachments/assets/31cd31a2-102a-4386-96a5-7a05ea1cf96c)


**ETL Processing**

**Extract (E)** - Data is retrieved from various sources like databases
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

**Layer Design**

![image](https://github.com/user-attachments/assets/9f1c6eca-3685-4853-92e6-8b27d01d01c8) 

1) Bronze Layer: Stores raw data as-is from the source systems. Data is ingested from CSV Files into SQL Server Database.
2) Silver Layer: This layer includes data cleansing, standardization, and normalization processes to prepare data for analysis.
3) Gold Layer: Houses business-ready data modeled into a star schema required for reporting and analytics.

**Building the Data Warehouse** 

Objective:

Develop a modern data warehouse using SQL Server to consolidate sales data, enabling analytical reporting and informed decision-making.

Specifications

- Data Sources: Import data from two source systems (ERP and CRM) provided as CSV files.
- Data Quality: Cleanse and resolve data quality issues prior to analysis.
- Integration: Combine both sources into a single, user-friendly data model designed for analytical queries.
- Scope: Focus on the latest dataset only; historization of data is not required.
- Documentation: Provide clear documentation of the data model to support both business stakeholders and analytics teams.
