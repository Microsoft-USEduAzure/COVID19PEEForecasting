# Build copy pipeline using Azure Data Factory

## Pre-requisite task: [Create Azure Data Factory V2](provision-azure-data-factory-v2.md)

## Task: Create Linked Services Connections

In this section you will create a linked service to the SQL database and to your Azure DataBricks Notebook to process data and calculate forecasts daily.

1. Open the **Azure Data Factory** portal and click the **Author option *(pencil icon)*** on the left-hand side panel. Under **Connections** tab, click **Linked Services** and then click **+ New** to create a new linked service connection.

    ![Create new pipeline](media/5.png)

1. On the **New Linked Service** blade, type “Azure Sql” in the search box to find the **Azure SQL** linked service. Click **Continue**.
    
    ![Create new pipeline](media/Pipeline/2.png)

1.	On the New Linked Service (Azure SQL Database) blade, enter the following details:
    <br>- **Name**: COVID19SQL
    <br>- **Connect via integration runtime**: AutoResolveIntegrationRuntime
    <br>- **Account selection method**: From Azure subscription
    <br>- **Azure subscription**: *your subscription*
    <br>- **Server Name**: *your server name*
    <br>- **Database Name**: COVID19Forecasts
    <br>- **Authentication** Type: SQL Authentication
    <br>- **User** Name: *your user name*
    <br>- **Password**: *your password*

    ![](media/Pipeline/3.png)

1.	Click **Test connection** to make sure you entered the correct connection details and then click **Finish**

1. On the **New Linked Service** blade, Select the Compute Option, find the **Azure DataBricks** linked service. Click **Continue**.

    ![Create new pipeline](media/Pipeline/4.png)

1. On the New Linked Service (Azure DataBricks) blade, enter the following details:
    <br>- **Name**: COVID19Databricks_ADFLink
    <br>- **Connect via integration runtime**: AutoResolveIntegrationRuntime
    <br>- **Account selection method**: From Azure subscription
    <br>- **Azure subscription**: *your subscription*
    <br>- **DataBricks Workspace**: *COVID19DataBricks*
    <br>- **Access Token**: *Access Token Generated on the Notebook Copied to Notepad*
    <br>- **Select Cluster** Existing Interactive Cluster 
    <br>- **Choose from existing clusters**: *COVID19-clusters*    

 ![Create new pipeline](media/Pipeline/5.png)

 1.	Click **Test connection** to make sure you entered the correct connection details and then click **Finish**

1. You should now see 2 linked services connections that will be used in our new pipeline

    ![Create new pipeline](media/Pipeline/6.png)
  

## Task Create the initial Pipeline

In this section you create a data factory pipeline to copy data in the following sequence:

    * Execute a stored procedure in the SQL database to delete the COVID19Staging Table
    * Execute the DataBricks Notebook that calculates Forecasting
    * Execute a stored procedure in the SQL database to copy records to the COVID19 table and calculate new daily cases.

1. On the New Pipeline tab, enter the following details:

    <br>- **General > Name**: Process Daily Data
    
1. Leave remaining fields with default values.

    ![Create new pipeline](media/Pipeline/7.png)
    ![Create new pipeline](media/Pipeline/8.png)

1.	From the Activities panel, type “Stored Procedure” in the search box. Drag the *Stored Procedure* on to the design surface. This stored procedure will delete the COVIDStating Table in the SQL Database

  ![Create new pipeline](media/Pipeline/9.png)

1. Select the Stored Procedure activity and enter the following details:
    <br>- **General > Name**: Clear Staging Tables
    <br>- **SQL Account > Linked Service**: COVID19SQL
    <br>- **Stored Procedure > Stored procedure name**: DeleteStagingTable
   
1.	Leave remaining fields with default values.

    ![Create new pipeline](media/Pipeline/10.png)
    ![Create new pipeline](media/Pipeline/11.png)
    ![Create new pipeline](media/Pipeline/12.png)

1.	From the Activities panel, type “Stored Procedure” in the search box. Drag the *Stored Procedure* on to the design surface. This stored procedure will delete the COVIDStating Table in the SQL Database

1.	From the Activities panel, type “Notebook” in the search box. Drag the Notebook activity on to the design surface.
    ![Create new pipeline](media/Pipeline/13.png)

1.	Select the Notebook activity and enter the following details:
    <br>- **General > Name**: Forecasting Models
    <br>- **Azure Databricks > Databricks linked service**: COVID19Databricks_ADFLink
    <br>- **Settings > Notebook path**: Click on the Browse button and follow the instructions to select the **build forecast** notebook
    
1.	Leave remaining fields with default values.
    
    ![Create new pipeline](media/Pipeline/14.png)
    ![Create new pipeline](media/Pipeline/15.png)
    ![Create new pipeline](media/Pipeline/16.png)

    1.	From the Activities panel, type “Stored Procedure” in the search box. Drag the *Stored Procedure* on to the design surface. This stored procedure will delete the COVIDStating Table in the SQL Database

  ![Create new pipeline](media/Pipeline/17.png)

1. Select the Stored Procedure activity and enter the following details:
    <br>- **General > Name**: Process New Records
    <br>- **SQL Account > Linked Service**: COVID19SQL
    <br>- **Stored Procedure > Stored procedure name**: ProcessNewRecords

1.	Leave remaining fields with default values.

    ![Create new pipeline](media/Pipeline/18.png)
     ![Create new pipeline](media/Pipeline/11.png)
    ![Create new pipeline](media/Pipeline/19.png)

1.	Create a **Success *(green)*** precedence constraint between Clear Stating Table to  Forecasting Models to Process New records. You can do it by dragging the green connector from Clear Staging Table and landing the arrow onto Forecasting Models.    

    ![Create new pipeline](media/Pipeline/20.png)

## Task Triggers & Publish Pipeline

We are going to create a trigger that will run every 12 hours at 5:00 AM and 5:00 pm to refresh data.

1. Navigate to the top menu blade on the pipeline and **click** on **Add Trigger** then select the **New/Edit** Option

    ![Create Trigger](media/Pipeline/21.png)

1. At the New Trigger blade enter the following parameters:
    <br>- **Name**: Update Daily Data
    <br>- **Type**: Schedule
    <br>- **Start Date**: Select the closest 5:00 AM or PM closest to you
    <br>- **Recurrence**: Every 12 Hours

Click the **Ok** button

  ![Create Trigger](media/Pipeline/22.png)

2.	Publish your pipeline changes by clicking the **Publish** button and follow the instructions

    ![Create Trigger](media/Pipeline/30.png)

3. Once publishing is completed, run the trigger manually to populate the database. Nagivate to the **Trigger** menu option on the top blade of the pipeline, select trigger now, click the OK button

    ![Create Trigger](media/Pipeline/32.png)
    ![Create Trigger](media/Pipeline/33.png)

4. Navigate to the monitor option to check execution of the pipeline
    ![Create Trigger](media/Pipeline/34.png)

5. You will see the results of the inital execution

    ![Create Trigger](media/Pipeline/35.png)

## Next task: [Provision Azure Function](../azure-function/create-azure-function.md)
