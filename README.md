End-to-End Data Engineering Pipeline for Olist E-commerce Analytics
This project demonstrates a complete, end-to-end data engineering pipeline built on the Microsoft Azure platform. The goal is to ingest raw, transactional e-commerce data, perform complex transformations, and load the data into a clean, optimized table ready for consumption by BI tools like Power BI. 
 
Architecture Diagram: Olist End-to-End Data Engineering Pipeline
This pipeline answers key business questions by linking marketing leads to final sales, seller demographics, and monetary performance. 

Technologies Used

•	Orchestration: Azure Data Factory (ADF)

•	Storage (Data Lake): Azure Data Lake Storage (ADLS) Gen2

•	Compute (Transformation): Azure Databricks

•	Transformation Language: PySpark

•	Data Sources: Azure SQL Database

•	Data Warehouse (BI Destination): Azure SQL Database

•	BI Tool (Consumer): Power BI

Pipeline Workflow

This project is broken down into three main stages, following a modern data architecture pattern.

1. Ingestion: SQL Database -> Data Lake (Bronze Layer)
   
The first step is to extract raw data from the source (an Azure SQL Database) and land it in our data lake.

•	Tool: Azure Data Factory

•	Pattern: A dynamic, metadata-driven pipeline was built.

•	Process:

 1.	A Lookup activity queries a metadata table in the SQL DB to get a list of all tables to be copied.
 2.	A ForEach loop iterates over this list of table names.
 3.	Inside the loop, a Copy activity dynamically parameterizes the source (table name) and sink (file name) to copy each table from the SQL DB into our ADLS Gen2 (Bronze) container as raw files (as .csv).


2. Transformation: Bronze -> Silver -> Gold (Databricks)
   
This is the core of the project, where all business logic and transformations are applied using PySpark in an Azure Databricks notebook.

•	Tool: Azure Databricks & PySpark

•	Process:
1.	Read Raw Data: Data is read from the Bronze layer using spark.read.csv() ensuring header and inferSchema options are set.
   
2	Join Data: Four key datasets were joined to create a single, wide table:

	dbo.olist_marketing_qualified_leads (All potential leads)

	dbo.olist_closed_deals (Leads that converted)

	dbo.olist_sellers (Demographic info for sellers)

	dbo.olist_order_items (Monetary/sales data for each transaction)

3.	Apply Business Logic:
   
	A Left Join strategy was used to connect leads -> deals -> sellers -> sales. This ensures we keep all leads, even those not yet closed, for a complete funnel analysis.

	Created a new column lead_status ("Open" vs. "Closed-Won") based on whether a match was found in the closed_deals table.

	Created a new feature time_to_close_days by calculating the datediff between won_date and first_contact_date.

4.	Final Table: The resulting DataFrame was cleaned, and key columns were selected to create the final PBI_Lead_Performance table.

3. Load: Gold Layer -> Data Warehouse
   
The final transformed data is loaded into two destinations, making it robust and available for different use cases.

•	Tool: PySpark (JDBC & Parquet Writers)

•	Destination 1 (Data Warehouse):

o	The final PBI_Lead_Performance DataFrame was written directly to our Azure SQL Database using a JDBC connection.

o	This provides a high-performance, relational endpoint for Power BI to connect to in DirectQuery or Import mode.

•	Destination 2 (Data Lake - Gold Layer):

o	(Best Practice) A copy of the final, clean table is also saved back to the ADLS Gen2 (Gold) container in Parquet format. This serves as the permanent, queryable "source of truth" for the data lakehouse.

Final Output for BI

The project's output is a single, clean table in Azure SQL Database named PBI_Lead_Performance. This table is optimized for BI analysis and can be directly connected to Power BI to build reports that answer key questions like:

•	What is our overall lead-to-sale conversion rate?

•	Which marketing source (origin) generates the most revenue?

•	What is the average time_to_close_days for a lead?

•	Which seller cities (seller_city) are our most valuable?

•	What is the relationship between business_segment and total_revenue?

