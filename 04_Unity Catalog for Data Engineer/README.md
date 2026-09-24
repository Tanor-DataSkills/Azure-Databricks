
Azure End-to-End Data Engineering Project: Medallion Architecture with Databricks [Part 2]
Rihab Feki
Rihab Feki

Using Azure SQL DB, Azure Data Factory, Databricks, Delta Lake, Power BI

Press enter or click to view image in full size

In the first part of this project tutorial, the incremental data pipeline was implemented using Microsoft Data Factory and we reached saving the raw data in the bronze layer.

In this part of the project, we will focus on using Databricks to implement the Medallion Architecture which supports data quality by refining data incrementally at each layer. Bronze layer captures raw data, Silver layer cleans and transforms it and Gold layer aggregates and enriches it for business use.

Step 1: Create a Unity Metastore
We will be following these steps:

Press enter or click to view image in full size

To create Compute, we must attach the Databricks workspace to the Unity Catalog. But to be able to create a Unity Metastore, we need to do that from the admin console.

All you need to do is navigate to Azure > Microsoft Intra ID > users, copy the User principal name, and log in to the console https://accounts.azuredatabricks.net/ (by resetting the password).

Press enter or click to view image in full size

The Databricks admin console
Then all you need to do is assign the admin role to your email address which you used in your Azure account.

Press enter or click to view image in full size

click in Account admin
Then go back to the Databricks workspace & refresh the page and you should see the ‘Manage account’ button.

Press enter or click to view image in full size

Notes to keep in mind:

It is only possible to create one Metastore per region.
Databricks creates default Metastores (to be deleted)
Press enter or click to view image in full size

delete the default metastore
Now, in the Databricks admin console in the Catalog tab, click on Create metastore.

Press enter or click to view image in full size

Add a name, select the region and provide the ADLS Gen 2 path (Azure Datalake Storage) following this convention:

<container_name>@<storage_account_name>.dfs.core.windows.net/<path>

Example: unity-metastore@datalakecarsale.dfs.core.windows.net/

This storage account will be used to store the default data e.g. metadata. To create this ADLS storage, navigate to the Azure portal > our project resource group > account storage > containers

Press enter or click to view image in full size

About the Access Connector ID which is required to create the metastore, we need to create a Databricks Access Connector which will connect the Databricks workspace and the ADLS Gen 2 storage (more detail in the below graph)

Press enter or click to view image in full size

Then in the resource group, add the access connector for Databricks

Press enter or click to view image in full size

Then, we need to assign to the access connector the role of “storage blob contributor” to be able to contribute to the datalake (storage account). To do that, click on the access connector > Access Control (IAM) > Add role.

After configuring the role, move to assign the managed identity members as shown in the screenshot below:

Press enter or click to view image in full size

After creating the needed resources, finish filling the form to creating the metastore and Finally assign the Workspace to the metastore.

Press enter or click to view image in full size

After completing this step, we successfully created a metastore and attached it to the Databricks workspace. Now we get back to the Databricks workspace and continue to create Compute.

Step 2: Create Compute
Press enter or click to view image in full size

Step 3: Create External locations
At the current state, we have the raw data on Azure datalake in the bronze container, now, we need to create 3 External Locations (bronze, silver, and gold) because we need to read & write data between these containers, so we should have an external location for it.

To create an external location, we should have “storage credentials”.

Press enter or click to view image in full size

to create an External Location, you need to start by creating credentials. So, navigate to Databricks Workspace and click on Catalog > External Data > Credentials.

Press enter or click to view image in full size

start by creating the credential, then the external storage
After creating the credentials, click again on Catalog > External location And provide a URL following this structure: abfss://<container_name>@<storage_account_name>.dfs.core.windows.net/<path>

Press enter or click to view image in full size

When clicking on create, you will have the following error message:

User does not have CREATE EXTERNAL LOCATION on Metastore ‘cars_project’.

Get Rihab Feki’s stories in your inbox
Join Medium for free to get updates from this writer.

Enter your email
Subscribe

Remember me for faster sign in

To fix it, go to the Databricks admin console (https://accounts.azuredatabricks.net/) and edit the Metastore admin to make it your user account (not the intra ID email)

Press enter or click to view image in full size

Press enter or click to view image in full size

after creating the external location for the bronze layer, do the same work for the silver and gold layers.

Press enter or click to view image in full size

Now we are all set to pull the data from bronze, we need to apply transformation and we need to store that data in the silver container.

Step 4: Create a Workspace
Click on Workspace in the sidebar, then in the Workspace folder, create a new project folder, and inside it create a Notebook to define the catalogs and schemas.

Press enter or click to view image in full size

Within the notebook, we create one catalog and two schemas.

Press enter or click to view image in full size

To better understand the concept of Unity Catalog hierarchical architecture, check the following graph:

Press enter or click to view image in full size

Source: https://docs.databricks.com/en/data-governance/unity-catalog/best-practices.html
Catalog: Catalogs are the highest level in the data hierarchy (catalog > schema > table/view/volume) managed by the Unity Catalog metastore. They are intended as the primary unit of data isolation in a typical Databricks data governance model.
Schema: Schemas, or databases, are logical groupings of tabular data (tables and views), non-tabular data (volumes), functions, and machine learning models.
Tables: They contain rows of data.
Unity Catalog allows the creation of managed and external tables.

Managed tables are fully managed by Unity Catalog, including their lifecycle and storage
External tables rely on cloud providers for data management, making them suitable for integrating existing datasets and allowing write access from outside Databricks.
Now after creating the first Notebook in which we created the catalog and the schemas, we create a second Notebook to read the data & transform it.

Step 5: Silver layer — data transformation (one big table)
We will use PySpark API to read the data and one thing to note here is the ‘inferSchema’ option which helps to derive the schema from the raw data in parquet file format.

Press enter or click to view image in full size

Then, after reading the dataset, we will do some column transformation, to split the Model_ID and make the part before the ‘-’ as model_category.

Press enter or click to view image in full size

Then we created an additional column to calculate the revenue per unit this can be useful for the analytics.

Press enter or click to view image in full size

withColumn will create a new column if the name does not exists, if if does it will modify the column
Press enter or click to view image in full size

You can also create visualizations by clicking on the + button near Table.

Press enter or click to view image in full size

Then we write the transformed data to the silver storage container

Press enter or click to view image in full size

Then check that the parquet files were saved on Azure.

Press enter or click to view image in full size

To query the data, we can use SQL:

SELECT * FROM 'abfss://<container>@<storageaccount>.dfs.core.windows.net/<path>/<file>'
Press enter or click to view image in full size

Step 6: Gold layer (Dimension Model)
The main goal of transitioning data from the silver to the gold layer is to prepare data for high-level business intelligence and reporting. This involves modeling the data e.g. following the start schema, to ensure it is ready for consumption by end-users, analysts, and decision-makers.

Silver layer: doesn’t maintain historical changes — it’s more about reflecting the current, cleaned state of incoming data.

Gold layer — Moving to the Gold layer with a focus on dimensional modeling and implementing SCD, the strategy needs to capture and store historical changes for analysis.

Slowly Changing Dimension — Type 1
In this part of the tutorial, we will dive into the steps to implement the incremental data update of the dim_model table to create the dimension in the gold layer.

A detailed step-by-step guide is in this Databricks Notebook (PySpark)

link: https://github.com/RihabFekii/azure-databricks-end-to-end-project/blob/main/databricks_notebooks/gold_dim_model.py

One of the most important functions is the following:

# Incremental RUN 
if spark.catalog.tableExists('cars_catalog.gold.dim_model'):
    delta_table = DeltaTable.forPath(spark, "abfss://gold@datalakecarsale.dfs.core.windows.net/dim_model")
    # update when the value exists
    # insert when new value 
    delta_table.alias("target").merge(df_final.alias("source"), "target.dim_model_key = source.dim_model_key")\
        .whenMatchedUpdateAll()\
        .whenNotMatchedInsertAll()\
        .execute()

# Initial RUN 
else: # no table exists
    df_final.write.format("delta")\
        .mode("overwrite")\
        .option("path", "abfss://gold@datalakecarsale.dfs.core.windows.net/dim_model")\
        .saveAsTable("cars_catalog.gold.dim_model")
The final result should look like this in the dimension table:

Press enter or click to view image in full size

Then to create the rest of the dimensions, you can simply clone the same notebook and just rename it with the new dimension name, and make the necessary changes, like the relative columns and table name.

Press enter or click to view image in full size

The process repeats for all the dimensions which are the dim_branch and dim_dealer.


All the notebooks to re-produce these dimension tables creation with incremental load are found here.

Step 7: Create the Fact Table
In data warehousing, a fact table consists of a business process's measurements, metrics, or facts. It is located at the center of a star schema or a snowflake schema surrounded by dimension tables.

Press enter or click to view image in full size

The Fact Table is created after the Dim tables are made. So the first step is reading the Revenue, Units_Sold, and RevPerUnit from the silver layer and then joining the Dim tables with the fact table. Then, we add the keys to the created dimensions.

The following is the code snippet to make the left join of the fact table with the dimension tables and also to bring the rest of the columns from the silver table and the surrogate keys we created for the dimensions.

df_fact = df_silver.join(df_branch, df_silver.Branch_ID==df_branch.Branch_ID, how='left') \
    .join(df_dealer, df_silver.Dealer_ID==df_dealer.Dealer_ID, how='left') \
    .join(df_model, df_silver.Model_ID==df_model.Model_ID, how='left') \
    .join(df_date, df_silver.Date_ID==df_date.Date_ID, how='left')\
    .select(df_silver.Revenue, df_silver.Units_Sold, df_branch.dim_branch_key, 
    df_dealer.dim_dealer_key, df_model.dim_model_key, df_date.dim_date_key)
and then we need to write the resultant fact sales table in the gold layer, using this code snippet:

if spark.catalog.tableExists('factsales'): 
    deltatable = DeltaTable.forName(spark, 'cars_catalog.gold.factsales')

    deltatable.alias('trg').merge(df_fact.alias('src'), 'trg.dim_branch_key = src.dim_branch_key and trg.dim_dealer_key = src.dim_dealer_key and trg.dim_model_key = src.dim_model_key and trg.dim_date_key = src.dim_date_key')\
        .whenMatchedUpdateAll()\
        .whenNotMatchedInsertAll()\
        .execute()

else: 
    df_fact.write.format('delta')\
            .mode('Overwrite')\
            .option("path", "abfss://gold@datalakecarsale.dfs.core.windows.net/factsales")\
            .saveAsTable('cars_catalog.gold.factsales')
To have all the details of the code, check this Databricks Notebook.

Step 8: Databricks Workflows (End-to-end Pipeline)
We can automate this whole pipeline with Azure Data Factory, but we will opt for using Databricks.

To do that, navigate to Workflows on Databricks workspace and click on ‘create job’ and then fill in the needed info as shown below attach the silver_notebook and the cluster, and finally click on create task.

Press enter or click to view image in full size

Add more tasks in this manner:

Press enter or click to view image in full size

For the dimension model, make sure to configure a parameter of the incremental_flag at the stage of creating the task, as shown below:

Press enter or click to view image in full size

after adding all the dimensions tasks and the fact table task, you will end up having a sequential pipeline like the following:

Press enter or click to view image in full size

But to enhance the performance, we need to make all the DIM_tables tasks depend on the silver table and then make the fact_table depend on all the dimension tables by editing the Depends on option in the form.

Press enter or click to view image in full size

Now that we have all the tasks organized, click on ‘Run now’ to test the pipeline. Some steps of the pipeline could throw an error, in that case, click on the task highlight the error fix it in the Notebooks in the workspace, and re-run the workflow until it all succeeds.

Press enter or click to view image in full size

After creating the Fact tables and the dimensions in the Gold layer, the Data Analyst can now use this data to make SQL queries via the SQL Editor

Press enter or click to view image in full size

Make sure to turn off the compute once you are done with it.

Press enter or click to view image in full size

To test the functioning of the whole pipeline, navigate to the data factory, choose the incremental pipeline, run it again, and verify the count of the rows to verify the results (via the query editor in databricks)

At this stage we finished the whole end to end pipeline using Azure and Databricks.

Thank you for reading :)

Databricks

Azure

Data Engineering

Pipeline

Data Science

381


9

1


Rihab Feki
Written by Rihab Feki
1.8K followers
·
28 following
Data engineer


Follow

Help

Status

About

Careers

Press

Blog

Store

Privacy

Rules

Terms

Text to speech
