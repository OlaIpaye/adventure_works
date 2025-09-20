# Adventure Works
Azure Data Engineering Project.



## Project Overview
This project demonstrates how to build a modern data pipeline using **Azure Data Factory (ADF)** and **Azure Data Lake Storage (ADLS)**.  
The dataset comes from Kaggle’s [Adventure Works dataset](https://www.kaggle.com/datasets/ukveteran/adventure-works?resource=download), and is ingested into a **medallion architecture** (Bronze, Silver, Gold) for future transformations and analytics.

---

### Tech Stack
- **Azure Data Factory (ADF)** - Orchestration & data movement
- **Azure Data Lake Storage Gen2 (ADLS)** - Storage (Bronze, Silver, Gold zones)
- **GitHub** - Source control and dataset hosting
- **Kaggle Dataset** - Raw dataset source

---

### **Steps Completed So Far**:

### Adventure Works Data Ingestion Pipeline (Azure Data Factory + ADLS)

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
- File `Products.csv` is now stored in: **bronze**/products/Products.csv


## Images (Project Timeline)

1. **ADF Pipeline run success (CopyRawData)**

![ADF Pipeline Run Success](<Images/1 - ADF - data ingestion process, from github to ADLS bronze storage.png>)

2. **File successfully landed in ADLS Bronze container**

![Bronze Storage File](<Images/2 - raw data i initially ingested in the bronze container ADLS.png>)


