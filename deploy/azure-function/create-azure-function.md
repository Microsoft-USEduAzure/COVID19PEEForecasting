# Provision Azure Function

## Pre-requisite task: [Create Azure Data Factory V2](..\azure-data-factory-v2\provision-azure-data-factory-v2.md)

## Task: Provision an Azure Linux Based Azure Function

In this section will provision a Linux Based Azure function - this will hold the python code that performs statistical calculations for PPE forecasting

1. In Azure Portal, click **+Create a resource** link at top left of the page

1. In the Azure Marketplace search bar, type **Function App** and click on **Function APP** that appears in the drop down list

    ![provision new azure ](media/1.png)

1. On the New Linked Service (Azure SQL Database) blade, enter the following details:
    <br>- **Azure subscription**: *your subscription*
    <br>- **Resource Group**: COVID19Backend
    <br>- **Function App Name**: COVID19Forecasting+suffix
    <br>- **Run Time Stack**: Python
    <br>- **Version**: *3.7*
    <br>- **Region**: Same as your resource group
    <br>- **Authentication** Type: SQL Authentication
    <br>- **User** Name: *your user name*
    <br>- **Password**: *your password*
    
    ![Create new pipeline](media/2.png)

1. At the bottom of the screen, Click **Next -> Hosting**. Enter the following parameters:

    <br>- **Plan Type**: *Premium*
    <br>- **Sku & Size**: Elastic Premium

     At the bottom of the screen, Click **Create**.
    
    ![](media/3.png)

1. Once the deployment is completed click on **Go to resource**

    ![Create new pipeline](media/4.png)

## Next task: [Deploy Function to App](deploy-function-app.md)