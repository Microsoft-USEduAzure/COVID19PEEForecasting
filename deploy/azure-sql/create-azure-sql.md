
# Provision Serverless SQL Database

## Pre-requisite task: [Create Azure Resource Group](../azure-resource-group/create-resource-group.md)

## Task: Provision Azure Serverless Database & SQL Server
### This database will hold COVID18 Forecast records
1. In the [Azure Portal](https://portal.azure.com), click **+Create a resource** link at top left of the page.

1. In the Azure Marketplace search bar, type **azure SQL** and click on **Azure SQL)** that appears in the drop down list

    ![New](media/1.png)

1. Click the **Create** button.

1. Select single database option

    ![New](media/1a.png)

1. Enter the following paramenters:
    <br> - **Name**: COVID19ForeCasts
    <br> - **Subscription**: *Select your subscription*
    <br> - **Resource Group**: COVID19Backend
    <br> - **Select source**: *Select Blank database*
    <br> - **Server**:  *Create a new server*
    <br> - **Server Name**: covid19+*suffix*
    <br> - **Server admin login**: mainuser
    <br> - **Password**: #covid2019#
   
   > **NOTE: Make sure the Location of your SQL Server is the same as the location of your resource group**

![New](media/2.png)

1. Click the **Configure Database** and select the following configuration:
    
    * Serverless
    * Max VCores: 16
    * Min VCores: 5
    * Pause after: 5 hours
    
    ![New](media/4.png)
    
1. Check the **Notifications** icon in the upper right and wait until you see **Deployment succeeded** then click the **Go to resource** button.

    ![Notifications](media/5.png)

## Task: Configure Server Firewall, Create Database Tables & Stored Procedures

1. Navigate to the **Query editor** blade and login using your server credentials, you will get an error message, select the IP Address on the error message and copy it, then click on the link **set server firewall**

    ![Login](media/6.png)

1. At the firewall configuration settings follow these steps:
	1. Allow Azure services and resources to access this server to Yes
	1. Click in Add Client IP button to allow your PC to access the database
	1. Click the save button
	1. Click the Query Editor breadcrumb at the top of the screen to go back to the Query Editor
	
    ![firewall](media/7.png)

1. In the query editor, enter the following command and click the **Run** button

        CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'P@$$word123';
        
    ![Query editor](media/provision/5.png)

1.	On the new query window, create a new database schema named [NYC]. Use this SQL Command:

```sql
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[CovId19](
	[fdates] [date] NOT NULL,
	[forecast_vals] [float] NOT NULL,
	[CountryRegion] [varchar](100) NOT NULL,
	[ProvinceState] [varchar](100) NOT NULL,
	[model] [varchar](20) NOT NULL,
	[obs_pred_r2_G] [float] NULL,
	[newcases] [int] NULL,
 CONSTRAINT [PK_CovId19] PRIMARY KEY CLUSTERED 
(
	[fdates] ASC,
	[forecast_vals] ASC,
	[CountryRegion] ASC,
	[ProvinceState] ASC,
	[model] ASC
)WITH (STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
GO

SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[CovId19Staging](
	[fdates] [nvarchar](max) NULL,
	[forecast_vals] [float] NULL,
	[CountryRegion] [nvarchar](max) NULL,
	[ProvinceState] [nvarchar](max) NULL,
	[model] [nvarchar](max) NULL,
	[obs_pred_r2_G] [float] NULL
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO

SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE PROCEDURE [dbo].[DeleteStagingTable]

AS
BEGIN
IF OBJECT_ID('dbo.CovId19Staging', 'U') IS NOT NULL 
  DROP TABLE dbo.CovId19Staging; 
   
END
GO

SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE PROCEDURE [dbo].[ProcessNewRecords]

AS
BEGIN
    truncate table [dbo].[CovId19]


INSERT INTO [dbo].[CovId19]
           ([fdates]
           ,[forecast_vals]
           ,[CountryRegion]
           ,[ProvinceState]
           ,[model]
           ,[obs_pred_r2_G])
    select [fdates]
           ,[forecast_vals]
           ,[CountryRegion]
           ,[ProvinceState]
           ,[model]
           ,[obs_pred_r2_G] from [dbo].[CovId19Staging]


Update CovID19 
set newcases =  forecast_vals - isnull((Select forecast_vals from CovId19 c where CovId19.ProvinceState = c.[ProvinceState] and CovId19.CountryRegion = c.countryregion and c.fdates = DATEADD(DAY, -1, CovId19.fdates) and CovId19.model = c.model),0) 


Declare @deleteDate  date
select @deleteDate = min(fdates) from covid19 
select @deleteDate

delete from CovID19 where fdates = @deleteDate

END
GO

```

1.	Create a new table called Staging.NYCTaxiLocationLookup to host NYC taxi location lookup data. Use the script below:

```sql
create table [Staging].[NYCTaxiLocationLookup]
(
    [LocationID] [int] NULL,
    [Borough] [varchar](200) NULL,
    [Zone] [varchar](200) NULL,
    [service_zone] [varchar](200) NULL
)
with
(
    distribution = round_robin,
    clustered columnstore index
)
```

1.	Create a stored procedure that will transform and aggregate data and save the result in a new table called NYC.TaxiDataSummary . Use the script below:

```sql
create procedure Staging.spNYCLoadTaxiDataSummary
as
    --Drop and re-create Staging.idx_NYCTaxiData index
    if (exists(select top 1 1 from sys.indexes where name = 'idx_NYCTaxiData' and object_id = object_id('Staging.NYCTaxiData')))
        drop index idx_NYCTaxiData on Staging.NYCTaxiData 
    
    create index idx_NYCTaxiData on Staging.NYCTaxiData(tpep_pickup_datetime, PULocationID, payment_type, passenger_count, trip_distance, tip_amount, fare_amount, total_amount)

    --Drop and re-create NYC.TaxiDataSummary table
    if (exists(select top 1 1 from sys.objects where name = 'TaxiDataSummary' and schema_id = schema_id('NYC') and type = 'U'))
        drop table NYC.TaxiDataSummary 

    create table NYC.TaxiDataSummary 
    with (
        distribution = round_robin
    )
    as
    select 
        cast(tpep_pickup_datetime as date) as PickUpDate
        , PickUp.Borough as PickUpBorough
        , PickUp.Zone as PickUpZone
        , case payment_type
            when 1 then 'Credit card'
            when 2 then 'Cash'
            when 3 then 'No charge'
            when 4 then 'Dispute'
            when 5 then 'Unknown'
            when 6 then 'Voided trip'
        end as PaymentType
        , count(*) as TotalTripCount
        , sum(passenger_count) as TotalPassengerCount
        , sum(trip_distance) as TotalDistanceTravelled
        , sum(tip_amount) as TotalTipAmount
        , sum(fare_amount) as TotalFareAmount
        , sum(total_amount) as TotalTripAmount
    from Staging.NYCTaxiData
        inner join Staging.NYCTaxiLocationLookup as PickUp
            on NYCTaxiData.PULocationID = PickUp.LocationID
    group by cast(tpep_pickup_datetime as date) 
        , PickUp.Borough
        , PickUp.Zone
        , payment_type

    --drop index idx_NYCTaxiData so it does not impact future loads
    drop index idx_NYCTaxiData on Staging.NYCTaxiData
go
```

## Task: Pause Data Warehouse For Now
1. Make sure to click on the **Pause** button to ensure you are not being billed while the not actively working with the Data Warehouse. We'll turn this on when ready.

    ![Pause billing](media/provision/6.png)

## Next task: [Create Azure Data Factory V2](../azure-data-factory-v2/provision-azure-data-factory-v2.md)
