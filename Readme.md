# COVID19 PEE Forecasting Calculator | Azure Serverless Architecture


This project was born from [Rush Medical Center COVID19 Calculator](https://webalyticos.home.blog/2020/03/24/covd19forecast/) development of the same project using a Junyper Notebook & hosting it on a virtual machine using Voila.

Rush's Medical Center calculates forecasting based on 3 models: 
* Exponential
* Logistics
* Polynomial

### [Deploy to Azure](deploy/deploy.md)

## Architecture

![](media/architecture.png)

## Component Description

* Azure Data Factory
* Serverless Single SQL Database
* Azure Databricks
* Web App
* Azure Functions


### Azure Data Factory
It was implemented to orchestrate the ingestion and processing of daily COVID-19 confirmend cases data collected and provided by [John Hopkins University](https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports/)

The pipeline clears the staging table that holds data to be processed, calls an Azure Databricks that reads data from John Hopkins, parses daily information per estate and then generates the forecasting calculations, saves them to SQL server, then a stored proceduce is called to calculate new cases and save the fresh data into the COVID19Forecast Table.

The pipeline is scheduled to run every day at 5:00 am

![](media/pipeline.png)

### Azure SQL Single Database - Serverless pricing schema

Serverless SQL databases charges you for the storage space used by your database and the compute time only, making it a very affordable option. The database contains to tables: COVID19 and CoVID19Staging. There are 2 stored procedures: DeleteStagingTable (self explanatory)  and ProcessNewRecords, this stored procedure truncates the COVID19 table, inserts the new records and calculates new cases.

Separation between the two tables is due to indexing on COVID19 for faster searches.

![](media/databaseschema.png)

#### Database Schema

~~~~sql

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
~~~~

### Azure Databricks

Azure data bricks has 2 notebooks:

get_dataframe_dailyreports: this notebook aggreates data from [John Hopkins University](https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports/) and creates a temporary file

build forecast reads the temporary file and calculates forcasting values for every estate for 3 models:

* Exponential
* Logistic
* Polynomial

### Web App

This is a front end application for end users. This is a C# - ASP .NET application.

![](media/frontend.png)

### Azure Functions

This is a phython based function that runs statisticals calculations to calculate PPE forcasting for the selected state based on the selected parameters

![](media/functions.png)


