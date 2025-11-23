# End-to-End Fabrics Data Engineering Pipeline of Banking Loan Transactions

## Navigation/Quick Access
Quickly move to the section you are interested in by clicking on the appropriate link:

- [Overview](#overview)
- [Project Goal](#project-goal)
- [Project Architecture (Medallion)](#architecture-medallion)
- [Source Dataset](#source-dataset)
- [Gold Layer – Final Tables](#gold-layer-final-tables)
- [How to Reproduce (long section)](#how-to-reproduce)
- [Pipeline Performance](#pipeline-performance)
- [Semantic model](#semantic-model)
- [Power BI Dashboard](#dashboard)

---
## Overview
The **PKDD’99 Financial Dataset** (aka **Berka dataset**) contains real anonymized transactional data from a Czech bank (1993–1999). 
The data was ingested from Kaggle via think [this link](https://www.kaggle.com/datasets/marceloventura/the-berka-dataset). The dataset is originally from [here](https://webpages.charlotte.edu/mirsad/itcs6265/group1/index.html). More extensive information about the dataset there.

Business Question:
The bank wants to improve their services. For instance, the bank managers have only vague idea, who is a good client (whom to offer some additional services) and who is a bad client (whom to watch carefully to minimize the bank loses). Fortunately, the bank stores data about their clients, the accounts (transactions within several months), the loans already granted, the credit cards issued The bank managers hope to improve their understanding of customers and seek specific actions to improve services.

This project transforms all the raw data into a **production-grade analytics lakehouse** entirely inside **Microsoft Fabric** using the classic **Bronze → Silver → Gold** medallion architecture. It does this within seconds. Then it creates Visualization to assist answer the relevant business question.

## Project Goal
- Ingest all 8 original CSV files into Bronze Layer
- Build a documented Silver layer
- Deliver a Gold layer optimized to answer business questions completely
- Fully document a reproducible approach to implementing the project
- Create a Power BI report that answers key questions for the Czech bank

---
## Architecture (Medallion)

![Architectural diagram of project](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/Fabrics_berka_diagram.png)

---
## Source Dataset
The 8 original CSV (primarily .asc because it uses ; for value separation) files can be accessed [here](https://www.kaggle.com/datasets/marceloventura/the-berka-dataset).

The table below is a quick overview of each file. More details also [here](https://webpages.charlotte.edu/mirsad/itcs6265/group1/domain.html)

| File               | Rows       | Description                          |
|--------------------|------------|--------------------------------------|
| account.csv        | 4,500      | Bank accounts                        |
| client.csv         | 5,369      | Clients + district                   |
| disposition.csv    | 5,369      | Owner/disponent relationships        |
| card.csv           | 892        | Credit cards (classic/junior/gold)   |
| district.csv       | 77         | Demographics & unemployment          |
| loan.csv           | 682        | Loans + status (A/D = good/bad)      |
| order.csv          | 6,471      | Permanent orders                     |
| trans.csv          | 1,056,320  | All transactions                     |

## Gold Layer Final Tables  
### 🥇 Gold Tables Overview

The data output of this project are the **5 production-ready gold tables** that power all downstream analytics. The table below is an overview of the 5 gold tables.

| Table | Rows | Description |
|-------|-------|-------------|
| **z_gold_dim_date** | **2,922** | Full calendar for 1993–2000 (`date_sk` as PK). |
| **z_gold_dim_client** | **5,369** | Enriched client dimension (age in 1999, demographics, card/loan flags). |
| **z_gold_fact_transaction** | **1,262,625** | Fully enriched transaction fact table with `client_id`, `card_type`, `is_card_transaction`, etc. |
| **z_gold_monthly_card_spending** | **5597** | Monthly credit-card KPIs by client + card type. |
| **z_gold_client_360** | **5,369** | One row per client with lifetime spend, latest card, defaulter flag, high-value flags, dormant cards, etc. |

#### Gold Tables Connection Diagram
The robustness of the gold tables meant joins of several cleaned silver tables. The diagram below shows the connection between silver and gold tables created.

Database diagram connection of silver and gold tables:
![Database diagram showing gold and silver connections](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/db_diagram_gold_tables.png)

---

## How to Reproduce

This project uses **Microsoft Fabric Data Factory** to ingest 8 CSV (primarily .asc) financial datasets into a Fabric Lakehouse, where PySpark notebooks process the data through the **Bronze → Silver → Gold** layers following the Medallion Architecture.

Follow the steps below to recreate the environment.

### 1. Create a Microsoft Fabric Account
- Go to the Microsoft Fabric portal: https://app.fabric.microsoft.com
- Sign in with your Microsoft account
- Enable **Fabric Trial** (if not already active)

You’ll get access to Data Factory, Lakehouse, Notebooks, and Power BI.

### 2. Create a Workspace
- Workspaces → **New workspace**
- Name: `berka-medallion-project`
- Turn ON the following features:
  - Data Engineering
  - Data Factory
  - Power BI

### 3. Create a Fabric Lakehouse
- In the workspace → **New** → **Lakehouse**
- Name: `berka_financial_lakehouse`
- Inside the Lakehouse → **Files** section → create folder:


### 5. Set Up Microsoft Fabric Data Factory
- In the workspace → **New** → **Data pipeline**
- Name the pipeline: `berka-pipeline`

### 6. Create the Bronze Notebook
- In the Lakehouse, Create a new notebook and name it `Bronze Notebook` or download from repository [here](https://github.com/EdidiongEsu/berka_fabrics/blob/main/code/Fabrics%20Notebooks/Bronze%20Notebook.ipynb)
-  Always ensure that when you create a notebook, you connect to the existing Lakehouse you created by adding data sources (click on add data items on the left pane like in the picture below and select the lakehouse
  you created.

  ![Add lakehouse](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/add_data_bronze.png)

You will then see the lakehouse on the left and full code like in this picture here:

  ![Bronze Notebook](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/bronze_notebook_pic.png)



### 7. Create the Silver Notebook
- In the Lakehouse, Create a new notebook and name it `Silver Notebook` or download from repository [here](https://github.com/EdidiongEsu/berka_fabrics/blob/main/code/Fabrics%20Notebooks/Silver%20Notebook.ipynb)
  
  ![Silver Notebook](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/silver_notebook_pic.png)

  
### 8. Create the Gold Notebook
- In the Lakehouse, Create a new notebook and name it `Gold Notebook` or download from repository [here](https://github.com/EdidiongEsu/berka_fabrics/blob/main/code/Fabrics%20Notebooks/Gold%20Notebook.ipynb)
  
  ![Gold Notebook](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/gold_notebook_pic.png)


### 9. Connect Data Factory to PySpark Notebooks
- Open the Data factory created earlier and connect the notebooks created.
   ![Data Factory](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/data_factory_pipeline.png)

### 10. Run the Pipeline
- Trigger a run after connecting all notebooks, and wait for all three notebooks to complete successfully like in the picture below. Then verify that all tables have been created.
    
    ![pipeline run](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/pipeline_run.png)
    
Well Done! You now have a fully working Medallion Architecture pipeline in Microsoft Fabric. 

## Pipeline Performance
The pipeline successfully ingested 8 source files containing more than 1.07 million raw records. It transformed the data into the Bronze, Silver, and Gold layers and produced 1.28 million records across the final Gold tables.
The entire process, including ingestion, PySpark transformations, joins, aggregations, and Gold table creation, completed in under 5 minutes (indicated in the pipeline run picture above) on a relatively slow 35 Mbps network at the time.

This shows **strong pipeline performance**, efficient Spark execution, and an **optimized Medallion architecture** even under limited network conditions.

---
## Semantic Model
Go to the created Lakehouse and create a semantic model. This will be used to create the power BI report.

![connect lakehouse with semantic model](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/connect_lakehouse_semantic.png)

Then use semantic mode to begin creation of the Power BI report

![Created semantic model](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/semantic_model.png)


## Dashboard
### Visualizing in Power BI

The data in the Fabric Lakehouse is visualized using **Power BI** to deliver insights on banking transactions of the Czech bank. The gold tables created are very rich and as a result, the dashboard has many insights. 

![Combined Power BI Visualization](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/dashboard_all.png)

Dashboards include:
- Total Count of transactions done by the Czech bank
- Geographic profile of the bank's loan clients
- Defaulter profile of Clients
- Daily Transactions of Clients and lots more!
  
#### Looking Closer:
The Visualization link cannot be added here because of the semantic model link as Power BI restricts public sharing then. Looking closer:

![Page 1 Power BI Visualization](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/dashboard_page1.png)

![Page 2 Power BI Visualization](https://github.com/EdidiongEsu/berka_fabrics/blob/main/pics/dashboard_page2.png)
