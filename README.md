# Adventure Works
Azure Data Engineering Project.



## Project Overview
This project demonstrates how to build a modern data pipeline using **Azure Data Factory (ADF)** and **Azure Data Lake Storage (ADLS)**.  
The dataset comes from Kaggle’s [Adventure Works dataset](https://www.kaggle.com/datasets/ukveteran/adventure-works?resource=download), and is ingested into a **medallion architecture** (Bronze, Silver, Gold) for future transformations and analytics.

---

### Tech Stack
- **Azure Data Factory (ADF)** - Orchestration & data movement
- **Azure Data Lake Storage Gen2 (ADLS)** - Storage (Bronze, Silver, Gold zones)
- **Azure Databricks** - Data transformation & processing (PySpark)
- **Azure Synapse Analytics** - Data modeling, SQL queries, external tables, and abstraction layer
- **Microsoft Entra ID (App Registrations & Managed Identity)** - Secure authentication and access control
- **Power BI** - Data visualization and reporting
- **GitHub** - Source control and dataset hosting
- **Kaggle Dataset** - Raw dataset source

---

### **Steps Completed So Far**:

### Adventure Works Data Ingestion Pipeline (Azure Data Factory & ADLS Bronze Layer)

### 1. Dataset Preparation
- Created a GitHub repository and cloned it locally.  
- Added the Kaggle dataset (`Adventure Works`) into the repo folder.  
- Pushed updates to the remote GitHub repository.

### 2. Azure Resources Setup
- Created a **Resource Group** in Azure.  
- Created a **Storage Account**.  
- Created an **Azure Data Factory (ADF)** instance.  
- Created two **Linked Services**:
  - `HttpLinkedService` - Connects to GitHub repo (dataset source).
  - `StorageDataLake` - Connects to ADLS (storage destination).

### 3. Data Ingestion to Bronze Layer
- Built an **ADF Copy Data pipeline** (`CopyRawData`).  
- Source: GitHub (via `HttpLinkedService`).  
- Sink: ADLS `bronze` container (via `StorageDataLake`).  
- Verified pipeline run succeeded ✅.  
- File `Products.csv` is now stored in: **bronze/products/Products.csv**

### 4. Dynamic Parameterized Ingestion
- Created a **JSON configuration file** (`git.json`) containing metadata for all datasets:
  - `p_csv_relative_url` - GitHub dataset path  
  - `p_sink_folder` - Destination folder in Bronze layer  
  - `p_file_name` - Final CSV name  

- Uploaded the JSON file into a separate ADLS container called **parameters**.  
- Built a new pipeline using:
  - **Lookup Activity** - Reads the JSON file.  
  - **ForEach Activity** - Iterates through each dataset entry.  
  - **Copy Data Activity** - Dynamically copies each dataset into its respective Bronze folder.  

- Successfully ingested all Adventure Works datasets into the **bronze** container, each with its own dedicated folder (e.g., `AW_Products`, `AW_Customers`, `AW_Sales`, etc.).

## Adventure Works Data Transformation & Analysis (Azure Databricks & ADLS Silver Layer)

### 5. Azure Databricks Setup for Silver Layer
- Created an **Azure Databricks resource** and set up a compute cluster.
- Created an **App Registration** in Microsoft Entra ID, generated a **Client Secret**, and stored 4 key values (`app_id`, `object_id`, `secret`, `secret_id`) securely in a local file.
- In Azure Storage Account:
  - Created **Access Control (IAM)** role assignment.
  - Added **Storage Blob Contributor** role to the registered application.
- Back in Databricks:
  - Created a new **workspace** and **notebook**.
  - Connected Databricks to ADLS using OAuth credentials (based on [Microsoft Docs guide](https://learn.microsoft.com/en-us/azure/databricks/connect/storage/azure-storage)).
  - Configured `spark.conf` with OAuth settings to allow secure access.

### 6. Silver Layer Transformations
- Built a **Databricks Notebook (Silver Layer)** using PySpark.
- **Read Bronze Data** (e.g., Calendar, Customers, Products, etc.) into DataFrames.
- **Transformations performed:**
  - Example: Customer Data - Created new **Full Name** column by concatenating Prefix, FirstName and LastName.
  - Some datasets (e.g., Returns, Categories) required no transformations and were directly moved to Silver.
- **Saved transformed data** in **Parquet format** into the **Silver container** in ADLS.

### 7. Silver Layer Analysis (Databricks Visualizations)
- Read and transformed the **Sales Data**, then pushed it into the **Silver container** in ADLS.
- Created visualizations inside Databricks to explore business insights:
  - **Orders per Day** - Aggregated sales data to count total orders customers made each day.
  - **Category Performance** - Analyzed which product category performed best.

These visualizations help validate the transformed data in the Silver layer and provide initial analytical insights before moving to the Gold layer or BI tools.

## Adventure Works Analytics with Synapse (Serverless SQL & Gold Layer)

### 8. Synapse Analytics Setup

- Created an Azure Synapse Analytics resource and explored its interface (pipelines, transformations, and SQL pool features).

- Configured Managed Identity IAM in ADLS so Synapse could securely access transformed data in the Silver layer.

### 9. Querying Silver Layer with Serverless SQL

- Created a serverless database (aw_database) inside Synapse.

- Granted myself access to ADLS by assigning the Storage Blob Data Contributor role.

- Queried data from ADLS Silver layer using the OPENROWSET function:
  - SELECT *
    FROM OPENROWSET(
    BULK 'https://adworksstorageacc.dfs.core.windows.net/silver/AW_Calendar/',
    FORMAT = 'PARQUET'
    ) as Calendar;
- This allowed me to explore the Silver data directly in Synapse SQL.

### 10. Creating Views (Gold Schema)

- Created a gold schema in the serverless database.

- Defined views for each Silver dataset (calendar, customers, products, sales, etc.).

- Views act as an abstraction layer, hiding storage paths and exposing clean logical tables.

### 11. Creating External Tables (Gold Layer)

- Configured external resources in Synapse:

- Master Key

- Database Scoped Credential (Managed Identity)

- External Data Sources (Silver and Gold ADLS containers)

- External File Format (Parquet with Snappy compression)
- Created an external table in the Gold layer to persist transformed Sales data:
  - CREATE EXTERNAL TABLE gold.ext_sales
    WITH (
    LOCATION = 'ext_sales',
    DATA_SOURCE = source_gold,
    FILE_FORMAT = format_parquet
    )
    AS
    SELECT *
    FROM gold.sales;
- Successfully queried the new Gold external table.

## Adventure Works Visualization with Power BI

### 12. Connecting Synapse to Power BI

- Copied the Serverless SQL Endpoint from Synapse to connect with Azure data source in Power BI

- Connected Power BI to Synapse via the Azure Synapse Analytics SQL option.

- Loaded the gold.ext_sales external table into Power BI.

- Built quick visuals to validate the connection and explore the sales data.

---

## Images (Project Timeline)

1. **ADF Pipeline run success (CopyRawData)**

![ADF Pipeline Run Success](<Images/1 - ADF - data ingestion process, from github to ADLS bronze storage.png>)

2. **File successfully landed in ADLS Bronze container**

![Bronze Storage File](<Images/2 - raw data i initially ingested in the bronze container ADLS.png>)

3. **Dynamic pipeline using Lookup + ForEach (parameterized ingestion)**

![ADF pipeline with Lookup and ForEach activities](<Images/3 - ADF Dynamic parameterized data ingestion.png>)

4. **JSON parameter file uploaded into **parameters** container**

![ADLS parameters container with git.json file](<Images/4 - json file in the parameter container on ADLS.png>)

5. **Bronze container now contains all Adventure Works datasets in separate folders**

![ADLS bronze container with multiple dataset folders](<Images/5 - bronze container containing the folders for each of the files for Adventure Works.png>)

6. **Example: `Products.csv` stored inside `AW_Products/`**

![ADLS AW_Products folder with Products.csv](<Images/6 - products.csv file inside the aw_products folder in the bronze container on ADLS.png>)

7. **Databricks App Access Setup**

![Databricks notebook configuring spark.conf with OAuth credentials for ADLS access](<Images/7 - Snippet of databricks notebook for reading ne of my project data.png>)

8. **Reading Calendar Data**

![Databricks notebook reading Calendar CSV data from Bronze container using PySpark](<Images/8 - Snippet of databricks notebook for reading ne of my project data.png>)

9. **Reading Customer Data**

![Databricks notebook reading Customer CSV data from Bronze container into DataFrame](<Images/9 - Snippet of databricks notebook for reading ne of my project data.png>)

10. **Transform Customer Data**

![Databricks PySpark code transforming customer dataset by creating Full Name column](<Images/10 - Transform customer data in Azure Databricks.png>)

11. **Transformed Customer Data**

![Databricks table preview showing transformed customer data with new Full Name column](<Images/11 - Transformed customer data in Azure Databricks - Full Name column transformed.png>)

12. **Save to Silver Layer**

![PySpark code saving transformed customer data into Silver container in ADLS](<Images/12 - Transformed customer data in Azure Databricks - Saved to Silver layer in ADLS.png>)

13. **Silver Container in ADLS**

![ADLS Silver container showing multiple folders with transformed datasets](<Images/13 - All folders containing data transformation from bronze to silver zone in ADLS.png>)

14. **Sales Orders Per Day**

![Databricks line chart showing daily total orders from sales data](<Images/14 - Sales analysis - count of total orders in each day.png>)

15. **Category Performance**

![Databricks donut chart showing product category performance distribution](<Images/15 - Sales analysis - which category is performing best.png>)

16. **Querying Silver Data with OPENROWSET**

![Querying Silver Data with OPENROWSET](<Images/16 - Azure synapse analytics - Running SQL Script to query the Calendar data in ADLS.png>)

17. **Creating External Table from Sales View in Synapse**

![Creating External Table from Sales View in Synapse](<Images/17 - Azure synapse analytics - Created external table for sales data from the view, stored in gold later.png>)

18. **Sales Data Stored in Gold Container (Parquet format)**

![Sales Data Stored in Gold Container (Parquet format)](<Images/18 - Azure synapse analytics & ADLS gold layer - Stored external table of sales data into gold layer as parquet format after transformation.png>)

19. **Established Connection Between Synapse and Power BI**

![Established Connection Between Synapse and Power BI](<Images/19 - Established connection between Azure Synapse and Power BI.png>)

20. **Visualizing External Sales Table in Power BI**

![Visualizing External Sales Table in Power BI](<Images/20 - Visualized the external sales table in Power BI.png>)

## THE END

- **Project Completed**: Data successfully ingested (ADF to ADLS), transformed (Databricks to Silver zone), modeled (Synapse to Gold zone), and visualized (Power BI).