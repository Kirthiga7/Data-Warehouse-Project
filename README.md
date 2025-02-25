# Data-Warehouse-Project
**Data Warehouse**
A subject-oriented, integrated, time-variant, and non-volatile collection of data in support of management's decision making process.
It includes
-ETL Processing
-Data Architecture
-Data Integration
-Data Cleansing
-Data Load
-Data Modeling

#ETL Processing
**Extract (E)** – Data is retrieved from various sources like databases
-Method: Pull Extraction
-Type: Full Extraction 
-Technique: File Parsing
**Transform (T)** – Data is cleansed, formatted, and structured before loading
It includes Data enrichment, Data integration, Derived column, Data normalization& /standardization, Business rules& logic and Data aggregation.
Data Cleansing include remove duplicates, data filtering, data type casting, handling missing data, invalid values and unwanted spaces.
**Load (L)** – The transformed data is loaded into a data warehouse
-Processing type: Batch processing
-Load method: Full load - Truncate and Insert
-Slowly Changing Simensions(SCD) - SCD1-overwrite(updating the content of data warehouse)

Between Source and Target We have multiple layers depend on building the data warehouse
![image](https://github.com/user-attachments/assets/d5ed9044-4efa-471f-a73f-c02bc133ef2b)



![image](https://github.com/user-attachments/assets/3a0c5afe-e83a-4b94-a9db-54fb5311ea48)
