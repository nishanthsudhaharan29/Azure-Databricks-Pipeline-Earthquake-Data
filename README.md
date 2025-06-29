# Azure Data Engineering Pipeline

## Overview and Architecture

Designed and implemented an end-to-end Azure Data Engineering pipeline to automate the ingestion, processing, and storage of earthquake data for real-time analytics. Leveraged Azure Data Factory for orchestrating daily ingestion from the USGS Earthquake API, Databricks for data processing across bronze, silver, and gold layers, and Azure Data Lake Storage (ADLS) for scalable data storage. Employed reverse geocoding and data cleansing techniques to enrich and normalize data for analytics-ready outputs. This modular pipeline supports automated workflows using Databricks Workflows and delivers clean, structured data to stakeholders like government agencies and insurance companies, enabling faster decision-making and better risk assessment. The project showcases strong use of Azure-native tools, medallion architecture, and best practices in data engineering and pipeline automation.
### Business Case

Earthquake data is incredibly valuable for understanding seismic events and mitigating risks. Government agencies, research institutions, and insurance companies rely on up-to-date information to plan emergency responses and assess risks. With this automated pipeline, we ensure these stakeholders get the latest data in a way that’s easy to understand and ready to use, saving time and improving decision-making.

### Architecture Overview

This pipeline follows a modular architecture, integrating Azure’s powerful data engineering tools to ensure scalability, reliability, and efficiency. The architecture includes:

1. **Data Ingestion**: Azure Databricks orchestrates the daily ingestion of earthquake data from the USGS Earthquake API.
2. **Data Processing**: Databricks processes raw data into structured formats (bronze, silver, gold tiers).
3. **Data Storage**: Azure Data Lake Storage serves as the backbone for storing and managing data at different stages.

![Logo](screenshots/arcc.png)


### Data Modeling

We implement a **medallion architecture** to structure and organize data effectively:

1. **Bronze Layer**: Raw data ingested directly from the API, stored in Parquet format for future reprocessing if needed.
2. **Silver Layer**: Cleaned and normalized data, removing duplicates and handling missing values, ensuring it’s ready for analytics.
3. **Gold Layer**: Aggregated and enriched data tailored to specific business needs, such as adding in country codes.

![Logo](screenshots/2.png)

### Understanding the API

- The earthquake API provides detailed seismic event data for a specified start and end date.
- **Start Date**: Defines the range of data. This is dynamically set via Azure Data Factory for daily ingestion.
- **API URL**: `https://earthquake.usgs.gov/fdsnws/event/1/`

### Key Benefits

- **Automation**: Eliminates manual data fetching and processing, reducing operational overhead.
- **Scalability**: Handles large volumes of data seamlessly using Azure services.
- **Actionable Insights**: Provides stakeholders with ready-to-use data for informed decision-making.

---

## Step 1: Create an Azure Account
1. Sign up for an Azure account if you do not already have one.

---

## Step 2: Create a Databricks Resource
1. Create a Databricks resource in Azure.
2. Select the **Standard LTS (Long Term Support)** tier. Avoid using ML or other specialized tiers.

![Logo](screenshots/3.png)

---

## Step 3: Set Up a Storage Account
1. Create a Storage Account and enable **hierarchical namespaces** in the advanced settings.
2. Navigate to the Storage Account resource:
   - Go to **Data Storage > Containers > + Containers**.
   - Create three containers: `bronze`, `silver`, and `gold`.
3. Configure access:
   - Go to **IAM > Add role assignment > Storage Blob Data Contributor**.
   - Click **Next > Managed Identity > Select Members**.
   - Select **Access Connector for Azure Databricks** as the managed identity.
   - Click **Review + Assign**.
![Logo](screenshots/4.png)
---

## Step 4: Configure Databricks
1. Open the Databricks resource and click **Launch Workspace**.
2. Start a compute instance (this may take a few minutes).
3. Set up external data access:
   - Go to **Catalog > External Data > Credentials > Create Credential**.
   - For the **Access Connector ID**, use the Resource ID of the Access Connector:
     - Search for **Access Connector**, copy the Resource ID, and paste it here.
   - Use this section to grant permissions or delete credentials as needed.
4. Define external locations:
   - Navigate to **External Data > External Locations**.
   - Assign a name, select the storage credential, and specify the URL (use the container name and storage account name for `bronze`, `silver`, and `gold`).

![Logo](screenshots/5.png)
---

## Step 6: Install Required Libraries
1. Before running the `gold` notebook, install the `reverse_geocoder` library.
   - Navigate to **Compute > Cluster > Libraries > + Install New Library**.
   - Select **Source: PyPI** and enter **reverse_geocoder**.
   - Wait a few minutes for the installation to complete.
2. Use cluster-level libraries for consistency and shared environments across notebooks.
---

## Step 5: Create and Execute Notebooks
1. In the Databricks workspace, create a notebook for each layer (`bronze`, `silver`, `gold`).
   - Add the relevant code for `bronze` from GitHub.
   - Execute the notebook and refresh the Storage Account containers to verify updates.
   - Repeat the process for `silver` and `gold` notebooks, adding the corresponding code.

   ![Logo](screenshots/6.png)
  
## Data in Bronze Notebook
![Logo](screenshots/bn.png)
## Data in Silver Notebook
![Logo](screenshots/sn1.png)

![Logo](screenshots/sn2.png)

![Logo](screenshots/sn3.png)
## Data in Gold Notebook
![Logo](screenshots/gn.png)
---
## Create a Workflow
1. In Databricks -> Workflows, create a new job
2. Create three tasks in the order, bronze -> silver -> gold
3. In bronze,
   task name = bronze
   type = notebooth
   path = select the notebook path
   cluster = select the compute used in the notebook
4. In silver,
   task name = silver
   type = notebooth
   path = select the notebook path
   cluster = select the compute used in the notebook
   parameters = select json, and paste
   {
  "bronze_output": "{{tasks.bronze.values.bronze_output }}"
   }
5. In gold,
   task name = gold
   type = notebooth
   path = select the notebook path
   cluster = select the compute used in the notebook
   parameters = select json, and paste
   {
  "silver_output": "{{tasks.silver.values.silver_output}}",
  "bronze_output": "{{tasks.bronze.values.bronze_output}}"
   }

6. In schedules and triggers, add a trigger to run every day at any particular time. eg. evryday at 6 a.m.
7. Check if the workflow is working by running it

![Logo](screenshots/7.png)

![Logo](screenshots/8.png)
---

## Data Written to ADLS
## Bronze Container
![Logo](screenshots/bc.png)
## Silver Container
![Logo](screenshots/sc.png)
## Gold Container
![Logo](screenshots/gc.png)
## Key Considerations
- **Linked Services**: Ensure reusable and secure connections between Azure services.
- **Scalability**: Use Synapse for querying large datasets efficiently.
- **Data Engineering Focus**: Maintain an emphasis on structured pipelines and optimized workflows.

This guide provides a comprehensive approach to setting up a professional-grade Azure Databricks and Synapse workflow for data engineering.
