# Provision Front End App

## Pre-requisite task: [Deploy Azure Function](..\azure-function\deploy-function-app.md)

## Task: Provision an App Service

In this section will provision a Windows Based Web App - This is the front end of the application

1. In Azure Portal, click **+Create a resource** link at top left of the page

1. In the Azure Marketplace search bar, type **Function App** and click on **Function APP** that appears in the drop down list

    ![provision web app](media/1.png)

1. On the New Linked Service (Azure SQL Database) blade, enter the following details:
    <br>- **Azure subscription**: *your subscription*
    <br>- **Resource Group**: COVID19FrontEnd
    <br>- **Function App Name**: COVID19FrontEnd+suffix
    <br>- **Run Time Stack**: ASP.NET V4.7
    <br>- **App Service Plan**: (click new), enter a name
   
    Leave default the rest of the fields as default.
    
    ![Create Web App](media/2.png)

     At the bottom of the screen, Click **Create**.

1. Once the deployment is completed click on **Go to resource**

    ![website](media/3.png)

    You should be able to see your new Web App

    ![Create new pipeline](media/4.png)

## Next task: [Deploy Website](deploy-website.md)
