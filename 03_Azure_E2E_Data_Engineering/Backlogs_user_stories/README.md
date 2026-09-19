
🗺️ Project Phases & Guide
🏗️ Phase1 - Project Initialization
Goal: Preparation des étapes en définissant les Backlogs, users stories, sprints avant de commencer le développement du projet. On peut utiliser jira ou notion.

 Design the architecture
 Read Databricks reference for the project → LINK
 Draw the data lakehouse architecture using draw.io or similar → LINK
 Create GitHub repository → LINK
 **Connect GitHub to Databricks using URL (**Workspace → Create → Git Folder)
 Create Lakehouse schemas (Unity Catalog) usingUI or SQL**:** bronze silver gold
 Create a volume inside bronze schema raw_sources
 Upload the 6 CSV files from engineering folder into the Bronze volume → LINK
Result: Project is ready to start building Bronze, Silver, and Gold layers.

🥉 Phase2 - Building Bronze Layer
Goal: Build the Bronze layer by ingesting all raw CSV files into Delta tables without any kind of transformations.

 Create a folder in the repository called bronze to store all scripts inside it
 Create Bronze notebook
 Initial ingestion (manual) For each of the 6 CSV files:
 Read the CSV into a DataFrame
 Write the DataFrame to a table in the Bronze schema using overwrite mode, and use a source-system prefix in the table name (for example erp_ or crm_) to clearly identify where the data comes from.
 Run the script and query the bronze table to verify it is loaded correctly
 Run the whole notebook to see if everything works successfully.
 Commit & Push your changes to the GitHub repository
🔥
Bonus Advanced Task

Code review & Identify repeated logic

Create a dictionary to store file paths and table names
Loop through the dictionary to ingest all files
Result: All 6 raw source files are ingested into dedicated Bronze tables with no transformations applied.

🥈 Phase3 - Building Silver Layer
Goal: It is time to clean and transform our bronze data and load the clean results into silver layer. This is usually the most time consuming phase of the project and the fun part!

 Create repository structure
 Create a folder called silver
 Create two subfolders: crm erp
 For each Bronze table (6 tables)
 Create a Silver notebook silver_<source>_<table_name> e.g. silver_crm_cust_info
 Analyze data quality using SQL and List all identified issues
 Find duplicates
 Validate string values: Check extra spaces, Identify abbreviations to normalize
 Validate dates values: Check Data Type, check the format, handle missing values
 Validate numeric values
 Standardize business key IDs to ensure tables can be joined correctly.
 Check the name of columns and table and make a plan how to rename them to something friendly.
 Section 1: Read data Bronze Table and Load it into a DataFrame
 Section 2: Transform data
Fix issues one by one
Keep transformations small and clear
Avoid one large transformation block
Use Spark SQL or PySpark (Python)
Before going to next transformation always check the result “df.display()”
 Sanity checks the final DataFrame before writing
 Section 3: Write the DataFrame to a new Silver Table and use a friendly name for the new table
 Sanity checks of silver table after writing
 Finalize notebook
 Run the full notebook end to end
 Review structure and readability
 Add comments and documentation
 Clone the notebook as a template for the next table
 Commit & Push your changes to the GitHub repository
🔥
Bonus Advanced Task

Review all 6 Silver notebooks and identify repeated code
Reduce repetition by:
Using a config file with loops, or
Creating reusable Python functions
Result: Cleaner, scalable code and a strong step toward senior data engineering.

Result: All Bronze tables are transformed into analytics-ready Silver tables with validated data quality and standardized structure.

🥇 Phase4 - Building Gold Layer
Goal:

Break the data model away from the source systems and introduce a new data model that is suitable for business intelligence and analytics.
Use dimensional modeling to transform the Silver tables into a star schema with fact and dimension tables.
 Data Modeling Phase
 Understand the content of each Silver table
 Map each table to a business object such as customers, products, or sales
 Use draw.io to design the target data model. Example: fact_sales, dim_customers, dim_products
 Build Gold tables - For each table in the new model:
 Write an SQL query
 Join all relevant Silver tables for the dimension or fact
 Ensure no duplicates after joins
 Validate the query output
 Load the result into a DataFrame
 Write the DataFrame to a new Gold table using a clear naming prefix such as dim_ for dimension tables or fact_ for fact tables.
 Sanity checks of gold table after writing
 Commit & Push your changes to the GitHub repository
🔥
Bonus Task - Data Product Ownership

At this point, the data is ready for analytics and your tables represents a data product

You are now responsible for making it reliable, clear, and easy to use

Enhance metadata in Unity Catalog

Add meaningful descriptions to Gold tables
Add clear descriptions to all important columns
Ensure column names and meanings are easy to understand for analysts
Define data relationships

Define primary keys for dimension tables
Define foreign keys between fact and dimension tables
Result: All Silver tables are transformed into business-ready Gold tables designed for analytics and reporting.

Phase5 - Building the Pipeline
Goal: Automate the end-to-end Lakehouse flow so data is processed reliably from Bronze to Silver to Gold.

Current Setup
1 Bronze notebook
6 Silver transformation notebooks
3 Gold notebooks (dimensions and facts)
To run each layer cleanly, we introduce orchestration notebooks that act as single entry points.

 Create Orchestration Notebooks
 Silver orchestration: Create one Silver orchestration notebook that triggers all 6 Silver notebooks in sequence. Use dbutils.notebook.run to run notebookes.
 Silver orchestration: Create one Gold orchestration notebook that triggers all 6 Silver notebooks in sequence. Use dbutils.notebook.run to run notebookes.
 Create a Databricks Job
 Go to Databricks → Jobs & Pipelines then create Create a new Job

 Create a new Job and Give it a clear name, for example: loading_bike_data_lakehouse

 Add three Tasks:

!image.png

 Bronze layer: bronze notebook
 Silver layer: silver_orchestration that triggers all other silver notebookes
 Gold layer: gold_orchestration that triggers all other gold notebookes
 Run and Validate
 Click Run All,
 Monitor the job execution
 Ensure all tasks complete successfully
 Verify Bronze, Silver, and Gold tables are created correctly
 Schedule the Pipeline
 Add a trigger to run the job on a schedule (for example daily)
 For the first few days: (Mointor the runes and check logs)
 After three days, pause or adjust the trigger as needed
🎉 Congratulations
You’ve just built a complete Data Lakehouse.

This is Lakehouse 1.0 and it represents the core foundation of real data engineering work.

🎓 Portfolio & Career Tip
You can confidently use this project as a portfolio project. You have my permission to do so.

If you share it on GitHub or LinkedIn, I would appreciate it if you give credit to the original source.

If you are preparing for job interviews, make sure you practice explaining this project:

Why you designed the Lakehouse this way
How data flows from Bronze to Silver to Gold
How you ensured data quality and scalability
How you automated everything with pipelines
Being able to clearly explain this project can be a strong differentiator and may be one of the reasons a company decides to hire you.

This is real, practical data engineering work.

🚀 Next Steps!
From here, you can take your Lakehouse to the next level by adding more advanced capabilities, such as:

Data quality checks
Row counts, null checks, duplicates, and business rules
Reusable code and functions
Shared transformation logic, configs, and utilities
New data sources
APIs, Kafka, streaming data, and operational databases
CI/CD pipelines
Automated testing, deployment, and environment promotion
Security and governance
Access control, row-level security, and data masking
Monitoring and observability
Pipeline health, alerts, and performance tracking
Incremental and streaming pipelines
CDC, MERGE patterns, and real-time data processing
This is exactly how real-world data platforms evolve.

Strong foundations first, then continuous improvement.
